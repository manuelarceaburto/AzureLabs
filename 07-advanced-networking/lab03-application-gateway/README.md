# Lab 03: Application Gateway with Web Application Firewall

## Lab Overview

In this lab, you'll deploy Azure Application Gateway with Web Application Firewall (WAF) to provide layer 7 load balancing and web application security. You'll configure backend pools, routing rules, health probes, and SSL termination.

## Objectives

After completing this lab, you will be able to:
- Deploy Application Gateway with WAF
- Configure backend pools and health probes
- Implement path-based and multi-site routing
- Enable SSL termination and end-to-end SSL
- Configure WAF rules to protect web applications
- Monitor Application Gateway metrics and logs

## Prerequisites

- Azure subscription with Contributor access
- VS Code with Azure extensions
- Basic understanding of HTTP/HTTPS and load balancing

## Architecture

```
Internet
    ↓
Application Gateway (WAF)
    ↓
Backend Pool
    ├── Web Server 1
    └── Web Server 2
```

## Estimated Time: 60 minutes

---

## Task 1: Create Network Infrastructure

### Instructions

#### Step 1: Create Resource Group

1. **Open VS Code Terminal:**
   - Press `Ctrl+`` (backtick)

2. **Create Resource Group:**
   ```powershell
   $resourceGroup = "rg-appgw-lab"
   $location = "eastus"
   
   az group create --name $resourceGroup --location $location
   ```

#### Step 2: Create Virtual Network

1. **Create VNet with Subnets:**
   ```powershell
   # Create VNet
   az network vnet create `
     --resource-group $resourceGroup `
     --name vnet-appgw `
     --address-prefix 10.0.0.0/16 `
     --location $location
   
   # Create subnet for Application Gateway
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-appgw `
     --name snet-appgw `
     --address-prefix 10.0.1.0/24
   
   # Create subnet for backend VMs
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-appgw `
     --name snet-backend `
     --address-prefix 10.0.2.0/24
   ```

2. **Verify Subnets:**
   ```powershell
   az network vnet subnet list `
     --resource-group $resourceGroup `
     --vnet-name vnet-appgw `
     --output table
   ```

### Verification

- [ ] Resource group created
- [ ] VNet created (10.0.0.0/16)
- [ ] Application Gateway subnet exists (10.0.1.0/24)
- [ ] Backend subnet exists (10.0.2.0/24)

---

## Task 2: Create Backend Web Servers

### Instructions

#### Step 1: Create Availability Set

1. **Create Availability Set:**
   ```powershell
   az vm availability-set create `
     --resource-group $resourceGroup `
     --name avset-web `
     --platform-fault-domain-count 2 `
     --platform-update-domain-count 5 `
     --location $location
   ```

#### Step 2: Create Web Server VMs

1. **Create First Web Server:**
   ```powershell
   az vm create `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --vnet-name vnet-appgw `
     --subnet snet-backend `
     --availability-set avset-web `
     --image Ubuntu2204 `
     --admin-username azureuser `
     --generate-ssh-keys `
     --size Standard_B2s `
     --public-ip-address ""
   ```

2. **Create Second Web Server:**
   ```powershell
   az vm create `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --vnet-name vnet-appgw `
     --subnet snet-backend `
     --availability-set avset-web `
     --image Ubuntu2204 `
     --admin-username azureuser `
     --generate-ssh-keys `
     --size Standard_B2s `
     --public-ip-address ""
   ```

3. **Get VM Private IPs:**
   ```powershell
   $web01Ip = az vm show `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --show-details `
     --query privateIps `
     --output tsv
   
   $web02Ip = az vm show `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --show-details `
     --query privateIps `
     --output tsv
   
   Write-Host "Web01 IP: $web01Ip"
   Write-Host "Web02 IP: $web02Ip"
   ```

#### Step 3: Install and Configure Web Servers

1. **Install Nginx on Both VMs:**
   ```powershell
   # Install nginx on web01
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "sudo apt-get update && sudo apt-get install -y nginx"
   
   # Install nginx on web02
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --command-id RunShellScript `
     --scripts "sudo apt-get update && sudo apt-get install -y nginx"
   ```

2. **Create Custom Index Pages:**
   ```powershell
   # Create custom page for web01
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "echo '<html><body><h1>Web Server 01</h1><p>Backend: vm-web01</p></body></html>' | sudo tee /var/www/html/index.html"
   
   # Create custom page for web02
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --command-id RunShellScript `
     --scripts "echo '<html><body><h1>Web Server 02</h1><p>Backend: vm-web02</p></body></html>' | sudo tee /var/www/html/index.html"
   ```

3. **Create Health Probe Page:**
   ```powershell
   # Create health check endpoint on both servers
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "echo 'OK' | sudo tee /var/www/html/health.html"
   
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --command-id RunShellScript `
     --scripts "echo 'OK' | sudo tee /var/www/html/health.html"
   ```

### Verification

- [ ] Availability set created
- [ ] Two web server VMs created
- [ ] Nginx installed on both VMs
- [ ] Custom index pages created
- [ ] Health probe endpoints created

---

## Task 3: Deploy Application Gateway

### Instructions

#### Step 1: Create Public IP for Application Gateway

1. **Create Public IP:**
   ```powershell
   az network public-ip create `
     --resource-group $resourceGroup `
     --name pip-appgw `
     --sku Standard `
     --allocation-method Static `
     --location $location
   ```

2. **Get Public IP Address:**
   ```powershell
   $appgwIp = az network public-ip show `
     --resource-group $resourceGroup `
     --name pip-appgw `
     --query ipAddress `
     --output tsv
   
   Write-Host "Application Gateway Public IP: $appgwIp"
   ```

#### Step 2: Create Application Gateway with WAF

1. **Create Application Gateway (takes 5-10 minutes):**
   ```powershell
   az network application-gateway create `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --location $location `
     --vnet-name vnet-appgw `
     --subnet snet-appgw `
     --public-ip-address pip-appgw `
     --capacity 2 `
     --sku WAF_v2 `
     --http-settings-cookie-based-affinity Disabled `
     --frontend-port 80 `
     --http-settings-port 80 `
     --http-settings-protocol Http `
     --priority 100 `
     --servers $web01Ip $web02Ip
   ```

2. **Monitor Deployment:**
   ```powershell
   az network application-gateway show `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --query provisioningState `
     --output tsv
   ```
   
   Expected: "Succeeded"

#### Step 3: Verify in Portal

1. **Open Azure Portal:**
   - Navigate to "Application gateways"
   - Click on `appgw-lab`
   - Verify Status: "Running"
   - Note the Frontend IP configuration

### Verification

- [ ] Public IP created
- [ ] Application Gateway deployed with WAF_v2 SKU
- [ ] Gateway status shows "Succeeded"
- [ ] Backend pool contains both web servers
- [ ] Can view gateway in Azure Portal

---

## Task 4: Configure Health Probes

### Instructions

#### Step 1: Create Custom Health Probe

1. **Create Health Probe:**
   ```powershell
   az network application-gateway probe create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name health-probe `
     --protocol Http `
     --path "/health.html" `
     --interval 30 `
     --timeout 30 `
     --threshold 3 `
     --host "127.0.0.1"
   ```

#### Step 2: Update Backend HTTP Settings

1. **Update HTTP Settings to Use Custom Probe:**
   ```powershell
   az network application-gateway http-settings update `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name appGatewayBackendHttpSettings `
     --probe health-probe
   ```

#### Step 3: Verify Health Probe

1. **Check Backend Health:**
   ```powershell
   az network application-gateway show-backend-health `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --output table
   ```
   
   Expected: Both backends show "Healthy" status

2. **View in Portal:**
   - Navigate to `appgw-lab`
   - Click "Backend health" in left menu
   - Verify both servers show green (Healthy)

### Verification

- [ ] Custom health probe created
- [ ] HTTP settings updated to use probe
- [ ] Both backend servers show "Healthy"
- [ ] Backend health visible in portal

---

## Task 5: Test Application Gateway

### Instructions

#### Step 1: Test Basic Load Balancing

1. **Test HTTP Access:**
   ```powershell
   # Test multiple times to see load balancing
   1..10 | ForEach-Object {
       $response = Invoke-WebRequest -Uri "http://$appgwIp" -UseBasicParsing
       $response.Content | Select-String -Pattern "Web Server"
       Start-Sleep -Milliseconds 500
   }
   ```
   
   Expected: Responses alternate between "Web Server 01" and "Web Server 02"

2. **Test in Browser:**
   - Open browser
   - Navigate to: `http://<Application-Gateway-IP>`
   - Refresh multiple times
   - Should see different backend servers

#### Step 2: Test Health Probe

1. **Stop Nginx on One Server:**
   ```powershell
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "sudo systemctl stop nginx"
   ```

2. **Wait for Health Probe to Detect (90 seconds):**
   ```powershell
   Start-Sleep -Seconds 90
   ```

3. **Check Backend Health:**
   ```powershell
   az network application-gateway show-backend-health `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --output table
   ```
   
   Expected: vm-web01 shows "Unhealthy"

4. **Test Traffic:**
   ```powershell
   1..5 | ForEach-Object {
       $response = Invoke-WebRequest -Uri "http://$appgwIp" -UseBasicParsing
       $response.Content | Select-String -Pattern "Web Server"
   }
   ```
   
   Expected: All responses from "Web Server 02" only

5. **Restart Nginx:**
   ```powershell
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "sudo systemctl start nginx"
   
   Start-Sleep -Seconds 60
   ```

### Verification

- [ ] Can access application via gateway public IP
- [ ] Load balancing works (traffic to both servers)
- [ ] Health probe detects unhealthy backend
- [ ] Traffic redirected when backend is down
- [ ] Backend recovers when service restarted

---

## Task 6: Configure Path-Based Routing

### Instructions

#### Step 1: Create Additional Backend Pools

1. **Create Images Backend Pool:**
   ```powershell
   az network application-gateway address-pool create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name pool-images `
     --servers $web01Ip
   ```

2. **Create Videos Backend Pool:**
   ```powershell
   az network application-gateway address-pool create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name pool-videos `
     --servers $web02Ip
   ```

#### Step 2: Create Path-Based Rule

1. **Create URL Path Map:**
   ```powershell
   az network application-gateway url-path-map create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name path-map `
     --paths "/images/*" `
     --address-pool pool-images `
     --default-address-pool appGatewayBackendPool `
     --http-settings appGatewayBackendHttpSettings `
     --default-http-settings appGatewayBackendHttpSettings
   ```

2. **Add Videos Path Rule:**
   ```powershell
   az network application-gateway url-path-map rule create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --path-map-name path-map `
     --name videos-rule `
     --paths "/videos/*" `
     --address-pool pool-videos `
     --http-settings appGatewayBackendHttpSettings
   ```

3. **Update Routing Rule:**
   ```powershell
   az network application-gateway rule update `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name rule1 `
     --url-path-map path-map
   ```

#### Step 3: Create Test Content

1. **Create Images Directory on Web01:**
   ```powershell
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web01 `
     --command-id RunShellScript `
     --scripts "sudo mkdir -p /var/www/html/images && echo '<html><body><h1>Images from Web01</h1></body></html>' | sudo tee /var/www/html/images/index.html"
   ```

2. **Create Videos Directory on Web02:**
   ```powershell
   az vm run-command invoke `
     --resource-group $resourceGroup `
     --name vm-web02 `
     --command-id RunShellScript `
     --scripts "sudo mkdir -p /var/www/html/videos && echo '<html><body><h1>Videos from Web02</h1></body></html>' | sudo tee /var/www/html/videos/index.html"
   ```

#### Step 4: Test Path-Based Routing

1. **Test Default Path:**
   ```powershell
   Invoke-WebRequest -Uri "http://$appgwIp/" -UseBasicParsing | Select-Object -ExpandProperty Content
   ```
   
   Expected: Alternates between Web01 and Web02

2. **Test Images Path:**
   ```powershell
   Invoke-WebRequest -Uri "http://$appgwIp/images/" -UseBasicParsing | Select-Object -ExpandProperty Content
   ```
   
   Expected: Always "Images from Web01"

3. **Test Videos Path:**
   ```powershell
   Invoke-WebRequest -Uri "http://$appgwIp/videos/" -UseBasicParsing | Select-Object -ExpandProperty Content
   ```
   
   Expected: Always "Videos from Web02"

### Verification

- [ ] Additional backend pools created
- [ ] URL path map configured
- [ ] Path rules created for /images/* and /videos/*
- [ ] Default path routes to default pool
- [ ] /images/* routes to web01
- [ ] /videos/* routes to web02

---

## Task 7: Configure Web Application Firewall

### Instructions

#### Step 1: Enable WAF

1. **Create WAF Policy:**
   ```powershell
   az network application-gateway waf-policy create `
     --resource-group $resourceGroup `
     --name waf-policy `
     --location $location
   ```

2. **Configure WAF in Prevention Mode:**
   ```powershell
   az network application-gateway waf-policy policy-setting update `
     --resource-group $resourceGroup `
     --policy-name waf-policy `
     --mode Prevention `
     --state Enabled `
     --max-request-body-size 128
   ```

3. **Associate WAF Policy with Gateway:**
   ```powershell
   az network application-gateway update `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --set firewallPolicy.id=$(az network application-gateway waf-policy show --resource-group $resourceGroup --name waf-policy --query id -o tsv)
   ```

#### Step 2: Configure Managed Rules

1. **Set OWASP Rule Set:**
   ```powershell
   az network application-gateway waf-policy managed-rule rule-set add `
     --resource-group $resourceGroup `
     --policy-name waf-policy `
     --type OWASP `
     --version 3.2
   ```

2. **View Managed Rules:**
   ```powershell
   az network application-gateway waf-policy managed-rule rule-set list `
     --resource-group $resourceGroup `
     --policy-name waf-policy `
     --output table
   ```

#### Step 3: Test WAF Protection

1. **Test Normal Request (Should Succeed):**
   ```powershell
   Invoke-WebRequest -Uri "http://$appgwIp/" -UseBasicParsing
   ```
   
   Expected: HTTP 200 OK

2. **Test SQL Injection (Should Block):**
   ```powershell
   try {
       Invoke-WebRequest -Uri "http://$appgwIp/?id=1' OR '1'='1" -UseBasicParsing
   } catch {
       Write-Host "Request blocked by WAF: $($_.Exception.Message)"
   }
   ```
   
   Expected: HTTP 403 Forbidden

3. **Test XSS Attack (Should Block):**
   ```powershell
   try {
       Invoke-WebRequest -Uri "http://$appgwIp/?search=<script>alert('XSS')</script>" -UseBasicParsing
   } catch {
       Write-Host "Request blocked by WAF: $($_.Exception.Message)"
   }
   ```
   
   Expected: HTTP 403 Forbidden

4. **Test Path Traversal (Should Block):**
   ```powershell
   try {
       Invoke-WebRequest -Uri "http://$appgwIp/../../etc/passwd" -UseBasicParsing
   } catch {
       Write-Host "Request blocked by WAF: $($_.Exception.Message)"
   }
   ```
   
   Expected: HTTP 403 Forbidden

#### Step 4: Create Custom WAF Rule

1. **Create Custom Rule to Block Specific User-Agent:**
   ```powershell
   az network application-gateway waf-policy custom-rule create `
     --resource-group $resourceGroup `
     --policy-name waf-policy `
     --name block-bad-bot `
     --priority 100 `
     --rule-type MatchRule `
     --action Block `
     --match-conditions '[{"matchVariables":[{"variableName":"RequestHeaders","selector":"User-Agent"}],"operator":"Contains","matchValues":["BadBot"],"negationCondition":false}]'
   ```

2. **Test Custom Rule:**
   ```powershell
   try {
       Invoke-WebRequest -Uri "http://$appgwIp/" -UserAgent "BadBot/1.0" -UseBasicParsing
   } catch {
       Write-Host "Request blocked by custom rule: $($_.Exception.Message)"
   }
   ```
   
   Expected: HTTP 403 Forbidden

### Verification

- [ ] WAF policy created
- [ ] WAF enabled in Prevention mode
- [ ] OWASP rule set configured
- [ ] SQL injection attempts blocked
- [ ] XSS attempts blocked
- [ ] Path traversal blocked
- [ ] Custom rule created and working

---

## Task 8: Configure SSL Termination

### Instructions

#### Step 1: Create Self-Signed Certificate

1. **Generate Self-Signed Certificate:**
   ```powershell
   $certPassword = ConvertTo-SecureString -String "Azure@123!" -Force -AsPlainText
   
   $cert = New-SelfSignedCertificate `
     -Subject "CN=appgw.contoso.com" `
     -DnsName "appgw.contoso.com" `
     -KeyAlgorithm RSA `
     -KeyLength 2048 `
     -NotAfter (Get-Date).AddYears(1) `
     -CertStoreLocation "Cert:\CurrentUser\My" `
     -KeyExportPolicy Exportable
   
   $certPath = "$env:USERPROFILE\Desktop\appgw-cert.pfx"
   Export-PfxCertificate -Cert $cert -FilePath $certPath -Password $certPassword
   
   Write-Host "Certificate exported to: $certPath"
   ```

#### Step 2: Add HTTPS Listener

1. **Upload Certificate to Application Gateway:**
   ```powershell
   az network application-gateway ssl-cert create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name appgw-ssl-cert `
     --cert-file $certPath `
     --cert-password "Azure@123!"
   ```

2. **Create HTTPS Frontend Port:**
   ```powershell
   az network application-gateway frontend-port create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name port443 `
     --port 443
   ```

3. **Create HTTPS Listener:**
   ```powershell
   az network application-gateway http-listener create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name https-listener `
     --frontend-port port443 `
     --ssl-cert appgw-ssl-cert
   ```

4. **Create HTTPS Routing Rule:**
   ```powershell
   az network application-gateway rule create `
     --resource-group $resourceGroup `
     --gateway-name appgw-lab `
     --name https-rule `
     --http-listener https-listener `
     --rule-type Basic `
     --address-pool appGatewayBackendPool `
     --http-settings appGatewayBackendHttpSettings `
     --priority 101
   ```

#### Step 3: Test HTTPS Access

1. **Test HTTPS Connection:**
   ```powershell
   # Ignore certificate validation for self-signed cert
   [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
   
   $response = Invoke-WebRequest -Uri "https://$appgwIp" -UseBasicParsing
   Write-Host "HTTPS Status: $($response.StatusCode)"
   $response.Content | Select-String -Pattern "Web Server"
   ```
   
   Expected: HTTP 200 OK with content

2. **Test in Browser:**
   - Navigate to: `https://<Application-Gateway-IP>`
   - Accept certificate warning (self-signed)
   - Should see web content

### Verification

- [ ] Self-signed certificate created
- [ ] Certificate uploaded to Application Gateway
- [ ] HTTPS listener configured
- [ ] HTTPS routing rule created
- [ ] Can access application via HTTPS
- [ ] SSL termination working

---

## Task 9: Monitor Application Gateway

### Instructions

#### Step 1: Enable Diagnostic Logging

1. **Create Log Analytics Workspace:**
   ```powershell
   az monitor log-analytics workspace create `
     --resource-group $resourceGroup `
     --workspace-name law-appgw `
     --location $location
   ```

2. **Enable Application Gateway Diagnostics:**
   ```powershell
   $appgwId = az network application-gateway show `
     --resource-group $resourceGroup `
     --name appgw-lab `
     --query id `
     --output tsv
   
   $workspaceId = az monitor log-analytics workspace show `
     --resource-group $resourceGroup `
     --workspace-name law-appgw `
     --query id `
     --output tsv
   
   az monitor diagnostic-settings create `
     --name appgw-diagnostics `
     --resource $appgwId `
     --workspace $workspaceId `
     --logs '[{"category":"ApplicationGatewayAccessLog","enabled":true},{"category":"ApplicationGatewayPerformanceLog","enabled":true},{"category":"ApplicationGatewayFirewallLog","enabled":true}]' `
     --metrics '[{"category":"AllMetrics","enabled":true}]'
   ```

#### Step 2: View Metrics

1. **View Metrics in Portal:**
   - Navigate to `appgw-lab`
   - Click **Metrics**
   - Add metrics:
     - **Total Requests**
     - **Failed Requests**
     - **Healthy/Unhealthy Host Count**
     - **Throughput**
     - **Response Status** (2xx, 4xx, 5xx)

#### Step 3: Query Logs

1. **Query Access Logs:**
   ```kql
   AzureDiagnostics
   | where ResourceType == "APPLICATIONGATEWAYS"
   | where Category == "ApplicationGatewayAccessLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, clientIP_s, httpMethod_s, requestUri_s, httpStatus_d
   | limit 50
   ```

2. **Query WAF Logs:**
   ```kql
   AzureDiagnostics
   | where ResourceType == "APPLICATIONGATEWAYS"
   | where Category == "ApplicationGatewayFirewallLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, clientIP_s, Message, action_s, ruleId_s
   | limit 50
   ```

3. **Query Performance Logs:**
   ```kql
   AzureDiagnostics
   | where ResourceType == "APPLICATIONGATEWAYS"
   | where Category == "ApplicationGatewayPerformanceLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, instanceId_s, totalRequests_d, throughput_d
   | limit 50
   ```

### Verification

- [ ] Log Analytics workspace created
- [ ] Diagnostic settings enabled
- [ ] Metrics visible in portal
- [ ] Can query access logs
- [ ] Can query WAF logs
- [ ] Can query performance logs

---

## Task 10: Clean Up Resources

### Instructions

1. **Delete Resource Group:**
   ```powershell
   az group delete --name $resourceGroup --yes --no-wait
   ```

2. **Remove Certificate:**
   ```powershell
   # Remove from certificate store
   Get-ChildItem Cert:\CurrentUser\My | Where-Object {$_.Subject -like "*appgw.contoso.com*"} | Remove-Item
   
   # Delete exported file
   Remove-Item "$env:USERPROFILE\Desktop\appgw-cert.pfx" -ErrorAction SilentlyContinue
   ```

### Verification

- [ ] Resource group deletion initiated
- [ ] Certificate removed from store
- [ ] Certificate file deleted

---

## Summary

In this lab, you have:
- ✅ Deployed Application Gateway with WAF v2
- ✅ Configured backend pools with health probes
- ✅ Implemented path-based routing rules
- ✅ Enabled and configured Web Application Firewall
- ✅ Tested WAF protection against common attacks
- ✅ Configured SSL termination with HTTPS listener
- ✅ Monitored Application Gateway with diagnostic logs
- ✅ Queried access, WAF, and performance logs

## Key Takeaways

- Application Gateway provides Layer 7 load balancing
- WAF protects against OWASP Top 10 vulnerabilities
- Path-based routing enables microservices architectures
- Health probes ensure traffic only goes to healthy backends
- SSL termination offloads encryption from backend servers
- Custom WAF rules provide additional security controls
- Diagnostic logs provide detailed traffic and security insights
- WAF can operate in Detection or Prevention mode

## Additional Resources

- [Application Gateway Documentation](https://docs.microsoft.com/azure/application-gateway/)
- [Web Application Firewall Documentation](https://docs.microsoft.com/azure/web-application-firewall/)
- [Application Gateway Components](https://learn.microsoft.com/azure/application-gateway/application-gateway-components)
- [WAF Rule Groups](https://learn.microsoft.com/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules)

[← Back to Module Overview](../README.md)
