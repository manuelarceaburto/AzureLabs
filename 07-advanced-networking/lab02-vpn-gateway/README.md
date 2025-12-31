# Lab 02: VPN Gateway and Site-to-Site Connectivity

## Lab Overview

In this lab, you'll create VPN Gateway connections to enable secure hybrid connectivity between Azure and on-premises networks. You'll configure both site-to-site (S2S) and point-to-site (P2S) VPN connections.

## Objectives

After completing this lab, you will be able to:
- Deploy Azure VPN Gateway
- Configure site-to-site VPN for hybrid connectivity
- Implement point-to-site VPN for remote users
- Troubleshoot VPN connectivity issues
- Monitor VPN gateway metrics

## Prerequisites

- Azure subscription with Contributor access
- VS Code with Azure extensions
- Understanding of VNets and subnets

## Architecture

```
On-Premises Network          Azure VNet
    (Simulated)
         |                       |
    Local Gateway           VPN Gateway
         |                       |
         +------- VPN Tunnel ----+
              (IPsec/IKE)
```

## Estimated Time: 75 minutes

---

## Task 1: Create Network Infrastructure

### Instructions

#### Step 1: Create Resource Group

1. **Open VS Code Terminal:**
   - Press `Ctrl+`` (backtick)

2. **Create Resource Group:**
   ```powershell
   $resourceGroup = "rg-vpn-lab"
   $location = "eastus"
   
   az group create --name $resourceGroup --location $location
   ```

#### Step 2: Create Azure Virtual Network

1. **Create VNet:**
   ```powershell
   # Create VNet for Azure resources
   az network vnet create `
     --resource-group $resourceGroup `
     --name vnet-azure `
     --address-prefix 10.1.0.0/16 `
     --location $location
   
   # Create GatewaySubnet (required name for VPN Gateway)
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-azure `
     --name GatewaySubnet `
     --address-prefix 10.1.255.0/27
   
   # Create workload subnet
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-azure `
     --name snet-workload `
     --address-prefix 10.1.1.0/24
   ```

#### Step 3: Create Simulated On-Premises Network

1. **Create On-Premises VNet:**
   ```powershell
   # Create VNet to simulate on-premises
   az network vnet create `
     --resource-group $resourceGroup `
     --name vnet-onprem `
     --address-prefix 192.168.0.0/16 `
     --location $location
   
   # Create GatewaySubnet for on-prem VPN Gateway
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-onprem `
     --name GatewaySubnet `
     --address-prefix 192.168.255.0/27
   
   # Create on-prem workload subnet
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-onprem `
     --name snet-onprem-workload `
     --address-prefix 192.168.1.0/24
   ```

2. **Verify Networks:**
   ```powershell
   az network vnet list `
     --resource-group $resourceGroup `
     --output table
   ```

### Verification

- [ ] Resource group created
- [ ] Azure VNet created (10.1.0.0/16)
- [ ] Azure GatewaySubnet exists
- [ ] On-premises VNet created (192.168.0.0/16)
- [ ] On-premises GatewaySubnet exists

---

## Task 2: Deploy VPN Gateways

### Instructions

#### Step 1: Create Public IPs for VPN Gateways

1. **Create Public IPs:**
   ```powershell
   # Public IP for Azure VPN Gateway
   az network public-ip create `
     --resource-group $resourceGroup `
     --name pip-vpn-azure `
     --allocation-method Static `
     --sku Standard `
     --location $location
   
   # Public IP for On-Prem VPN Gateway
   az network public-ip create `
     --resource-group $resourceGroup `
     --name pip-vpn-onprem `
     --allocation-method Static `
     --sku Standard `
     --location $location
   ```

2. **Get Public IP Addresses:**
   ```powershell
   $azureGwIp = az network public-ip show `
     --resource-group $resourceGroup `
     --name pip-vpn-azure `
     --query ipAddress `
     --output tsv
   
   $onpremGwIp = az network public-ip show `
     --resource-group $resourceGroup `
     --name pip-vpn-onprem `
     --query ipAddress `
     --output tsv
   
   Write-Host "Azure VPN Gateway IP: $azureGwIp"
   Write-Host "On-Prem VPN Gateway IP: $onpremGwIp"
   ```

#### Step 2: Create Azure VPN Gateway

1. **Create Azure VPN Gateway (takes 20-45 minutes):**
   ```powershell
   az network vnet-gateway create `
     --resource-group $resourceGroup `
     --name vpn-gateway-azure `
     --vnet vnet-azure `
     --public-ip-address pip-vpn-azure `
     --gateway-type Vpn `
     --vpn-type RouteBased `
     --sku VpnGw1 `
     --location $location `
     --no-wait
   ```

2. **Create On-Premises VPN Gateway (takes 20-45 minutes):**
   ```powershell
   az network vnet-gateway create `
     --resource-group $resourceGroup `
     --name vpn-gateway-onprem `
     --vnet vnet-onprem `
     --public-ip-address pip-vpn-onprem `
     --gateway-type Vpn `
     --vpn-type RouteBased `
     --sku VpnGw1 `
     --location $location `
     --no-wait
   ```

3. **Monitor Deployment Progress:**
   ```powershell
   # Check Azure gateway status
   az network vnet-gateway show `
     --resource-group $resourceGroup `
     --name vpn-gateway-azure `
     --query provisioningState `
     --output tsv
   
   # Check on-prem gateway status
   az network vnet-gateway show `
     --resource-group $resourceGroup `
     --name vpn-gateway-onprem `
     --query provisioningState `
     --output tsv
   ```
   
   Expected: "Succeeded" when complete

4. **Wait for Both Gateways (20-45 minutes):**
   - This is a good time for a break ☕
   - Both gateways must be in "Succeeded" state before continuing

#### Step 3: Verify Gateways in Portal

1. **Open Azure Portal:**
   - Navigate to "Virtual network gateways"
   - Verify both gateways show "Succeeded"
   - Note the gateway types and SKUs

### Verification

- [ ] Public IPs created for both gateways
- [ ] Azure VPN Gateway deployment started
- [ ] On-Prem VPN Gateway deployment started
- [ ] Both gateways show "Succeeded" status
- [ ] Can view gateways in Azure Portal

---

## Task 3: Configure Site-to-Site VPN Connection

### Instructions

#### Step 1: Create Local Network Gateways

1. **Create Local Network Gateway for On-Premises:**
   ```powershell
   # Represents on-prem network from Azure's perspective
   az network local-gateway create `
     --resource-group $resourceGroup `
     --name lng-onprem `
     --gateway-ip-address $onpremGwIp `
     --local-address-prefixes 192.168.0.0/16 `
     --location $location
   ```

2. **Create Local Network Gateway for Azure:**
   ```powershell
   # Represents Azure network from on-prem's perspective
   az network local-gateway create `
     --resource-group $resourceGroup `
     --name lng-azure `
     --gateway-ip-address $azureGwIp `
     --local-address-prefixes 10.1.0.0/16 `
     --location $location
   ```

#### Step 2: Create VPN Connections

1. **Create Connection from Azure to On-Prem:**
   ```powershell
   $sharedKey = "Azure@VPN@Lab@2025!"
   
   az network vpn-connection create `
     --resource-group $resourceGroup `
     --name conn-azure-to-onprem `
     --vnet-gateway1 vpn-gateway-azure `
     --location $location `
     --shared-key $sharedKey `
     --local-gateway2 lng-onprem
   ```

2. **Create Connection from On-Prem to Azure:**
   ```powershell
   az network vpn-connection create `
     --resource-group $resourceGroup `
     --name conn-onprem-to-azure `
     --vnet-gateway1 vpn-gateway-onprem `
     --location $location `
     --shared-key $sharedKey `
     --local-gateway2 lng-azure
   ```

#### Step 3: Verify Connection Status

1. **Check Connection Status (wait 2-3 minutes):**
   ```powershell
   # Check Azure side
   az network vpn-connection show `
     --resource-group $resourceGroup `
     --name conn-azure-to-onprem `
     --query connectionStatus `
     --output tsv
   
   # Check on-prem side
   az network vpn-connection show `
     --resource-group $resourceGroup `
     --name conn-onprem-to-azure `
     --query connectionStatus `
     --output tsv
   ```
   
   Expected: "Connected" for both

2. **View Detailed Connection Info:**
   ```powershell
   az network vpn-connection list `
     --resource-group $resourceGroup `
     --output table
   ```

3. **Verify in Portal:**
   - Navigate to `vpn-gateway-azure`
   - Click "Connections" in left menu
   - Should show connection with status "Connected"
   - Green checkmark indicates successful connection

### Verification

- [ ] Local network gateways created
- [ ] VPN connections created on both sides
- [ ] Connection status shows "Connected"
- [ ] Connections visible in Azure Portal

---

## Task 4: Test Site-to-Site Connectivity

### Instructions

#### Step 1: Create Test VMs

1. **Create VM in Azure:**
   ```powershell
   az vm create `
     --resource-group $resourceGroup `
     --name vm-azure `
     --vnet-name vnet-azure `
     --subnet snet-workload `
     --image Ubuntu2204 `
     --admin-username azureuser `
     --generate-ssh-keys `
     --size Standard_B2s `
     --public-ip-address pip-vm-azure
   ```

2. **Create VM in On-Premises:**
   ```powershell
   az vm create `
     --resource-group $resourceGroup `
     --name vm-onprem `
     --vnet-name vnet-onprem `
     --subnet snet-onprem-workload `
     --image Ubuntu2204 `
     --admin-username azureuser `
     --generate-ssh-keys `
     --size Standard_B2s `
     --public-ip-address pip-vm-onprem
   ```

3. **Get VM Private IPs:**
   ```powershell
   $azureVmIp = az vm show `
     --resource-group $resourceGroup `
     --name vm-azure `
     --show-details `
     --query privateIps `
     --output tsv
   
   $onpremVmIp = az vm show `
     --resource-group $resourceGroup `
     --name vm-onprem `
     --show-details `
     --query privateIps `
     --output tsv
   
   Write-Host "Azure VM IP: $azureVmIp"
   Write-Host "On-Prem VM IP: $onpremVmIp"
   ```

#### Step 2: Test Connectivity via VPN

1. **SSH to Azure VM:**
   ```powershell
   $azureVmPublicIp = az vm show `
     --resource-group $resourceGroup `
     --name vm-azure `
     --show-details `
     --query publicIps `
     --output tsv
   
   ssh azureuser@$azureVmPublicIp
   ```

2. **Ping On-Premises VM from Azure VM:**
   ```bash
   # From vm-azure, ping on-prem VM
   ping 192.168.1.4 -c 4
   ```
   
   Expected: Successful ping responses

3. **Test in Reverse:**
   - SSH to on-prem VM
   - Ping Azure VM:
   ```bash
   ping 10.1.1.4 -c 4
   ```
   
   Expected: Successful ping responses

4. **Test TCP Connectivity:**
   ```bash
   # From vm-azure, test SSH port on on-prem VM
   nc -zv 192.168.1.4 22
   ```
   
   Expected: "Connection succeeded"

### Verification

- [ ] Test VMs created in both networks
- [ ] Can ping from Azure VM to on-prem VM
- [ ] Can ping from on-prem VM to Azure VM
- [ ] TCP connectivity works across VPN tunnel

---

## Task 5: Configure Point-to-Site VPN

### Instructions

#### Step 1: Generate Certificates

1. **Create Root Certificate (Windows):**
   ```powershell
   # Create self-signed root certificate
   $cert = New-SelfSignedCertificate `
     -Type Custom `
     -KeySpec Signature `
     -Subject "CN=P2SRootCert" `
     -KeyExportPolicy Exportable `
     -HashAlgorithm sha256 `
     -KeyLength 2048 `
     -CertStoreLocation "Cert:\CurrentUser\My" `
     -KeyUsageProperty Sign `
     -KeyUsage CertSign
   
   Write-Host "Root Certificate Thumbprint: $($cert.Thumbprint)"
   ```

2. **Create Client Certificate:**
   ```powershell
   $clientCert = New-SelfSignedCertificate `
     -Type Custom `
     -DnsName "P2SClientCert" `
     -KeySpec Signature `
     -Subject "CN=P2SClientCert" `
     -KeyExportPolicy Exportable `
     -HashAlgorithm sha256 `
     -KeyLength 2048 `
     -CertStoreLocation "Cert:\CurrentUser\My" `
     -Signer $cert `
     -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
   
   Write-Host "Client Certificate Thumbprint: $($clientCert.Thumbprint)"
   ```

3. **Export Root Certificate Public Key:**
   ```powershell
   $rootCertPath = "$env:USERPROFILE\Desktop\P2SRootCert.cer"
   Export-Certificate -Cert $cert -FilePath $rootCertPath
   
   # Get base64 encoded certificate
   $rootCertData = [Convert]::ToBase64String([System.IO.File]::ReadAllBytes($rootCertPath))
   $rootCertData = $rootCertData -replace "`r`n", ""
   
   Write-Host "Root certificate exported to: $rootCertPath"
   ```

#### Step 2: Configure Point-to-Site VPN

1. **Set VPN Client Address Pool:**
   ```powershell
   az network vnet-gateway update `
     --resource-group $resourceGroup `
     --name vpn-gateway-azure `
     --address-prefixes 172.16.0.0/24 `
     --client-protocol OpenVPN
   ```

2. **Upload Root Certificate:**
   ```powershell
   az network vnet-gateway root-cert create `
     --resource-group $resourceGroup `
     --gateway-name vpn-gateway-azure `
     --name P2SRootCert `
     --public-cert-data $rootCertData
   ```

3. **Download VPN Client Configuration:**
   ```powershell
   $vpnClientUrl = az network vnet-gateway vpn-client generate `
     --resource-group $resourceGroup `
     --name vpn-gateway-azure `
     --processor-architecture Amd64 `
     --output tsv
   
   Write-Host "VPN Client Download URL: $vpnClientUrl"
   ```

4. **Download and Extract VPN Client:**
   ```powershell
   $vpnClientZip = "$env:USERPROFILE\Desktop\VPNClient.zip"
   Invoke-WebRequest -Uri $vpnClientUrl -OutFile $vpnClientZip
   
   Expand-Archive -Path $vpnClientZip -DestinationPath "$env:USERPROFILE\Desktop\VPNClient"
   
   Write-Host "VPN Client extracted to: $env:USERPROFILE\Desktop\VPNClient"
   ```

#### Step 3: Install and Test VPN Client

1. **Install VPN Client:**
   - Navigate to: `C:\Users\YourName\Desktop\VPNClient\WindowsAmd64`
   - Right-click `VpnClientSetup*.exe`
   - Select **Run as administrator**
   - Click **Yes** to install

2. **Connect to Point-to-Site VPN:**
   - Press `Win + I` (Settings)
   - Go to **Network & Internet** → **VPN**
   - Click on `vnet-azure`
   - Click **Connect**

3. **Verify Connection:**
   ```powershell
   # Check VPN connection status
   Get-VpnConnection
   
   # Should show ConnectionStatus: Connected
   ```

4. **Test Access to Azure Resources:**
   ```powershell
   # Ping Azure VM via private IP
   ping 10.1.1.4
   
   # Expected: Successful ping
   ```

### Verification

- [ ] Root and client certificates created
- [ ] Root certificate uploaded to VPN gateway
- [ ] VPN client configuration downloaded
- [ ] VPN client installed on local machine
- [ ] Successfully connected via Point-to-Site VPN
- [ ] Can access Azure resources via private IP

---

## Task 6: Monitor VPN Gateway

### Instructions

#### Step 1: Enable Diagnostic Logging

1. **Create Log Analytics Workspace:**
   ```powershell
   az monitor log-analytics workspace create `
     --resource-group $resourceGroup `
     --workspace-name law-vpn `
     --location $location
   ```

2. **Get Workspace ID:**
   ```powershell
   $workspaceId = az monitor log-analytics workspace show `
     --resource-group $resourceGroup `
     --workspace-name law-vpn `
     --query id `
     --output tsv
   ```

3. **Enable Gateway Diagnostics:**
   ```powershell
   $gatewayId = az network vnet-gateway show `
     --resource-group $resourceGroup `
     --name vpn-gateway-azure `
     --query id `
     --output tsv
   
   az monitor diagnostic-settings create `
     --name vpn-diagnostics `
     --resource $gatewayId `
     --workspace $workspaceId `
     --logs '[{"category":"GatewayDiagnosticLog","enabled":true},{"category":"TunnelDiagnosticLog","enabled":true},{"category":"RouteDiagnosticLog","enabled":true},{"category":"IKEDiagnosticLog","enabled":true}]' `
     --metrics '[{"category":"AllMetrics","enabled":true}]'
   ```

#### Step 2: View Gateway Metrics

1. **View Metrics in Portal:**
   - Navigate to `vpn-gateway-azure`
   - Click **Metrics** in left menu
   - Add metrics:
     - **Gateway Bandwidth**
     - **Tunnel Bandwidth**
     - **Tunnel Egress Bytes**
     - **Tunnel Ingress Bytes**
     - **BGP Peer Status**

2. **Create Dashboard:**
   - Click **Pin to dashboard**
   - Create new dashboard: "VPN Monitoring"
   - Add multiple metric charts

#### Step 3: Query VPN Logs

1. **Navigate to Logs:**
   - Go to `vpn-gateway-azure`
   - Click **Logs** in left menu

2. **Query Gateway Diagnostic Logs:**
   ```kql
   AzureDiagnostics
   | where Category == "GatewayDiagnosticLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, Message
   | limit 50
   ```

3. **Query Tunnel Diagnostic Logs:**
   ```kql
   AzureDiagnostics
   | where Category == "TunnelDiagnosticLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, remoteIP_s, status_s, Message
   | limit 50
   ```

4. **Query IKE Diagnostic Logs:**
   ```kql
   AzureDiagnostics
   | where Category == "IKEDiagnosticLog"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, Message
   | limit 50
   ```

### Verification

- [ ] Log Analytics workspace created
- [ ] Diagnostic settings enabled
- [ ] Gateway metrics visible
- [ ] Can query diagnostic logs
- [ ] Dashboard created for monitoring

---

## Task 7: Troubleshoot VPN Connectivity

### Instructions

#### Step 1: Check Connection Status

1. **Verify Connection State:**
   ```powershell
   az network vpn-connection show `
     --resource-group $resourceGroup `
     --name conn-azure-to-onprem `
     --query '{Status:connectionStatus, IngressBytes:ingressBytesTransferred, EgressBytes:egressBytesTransferred}' `
     --output table
   ```

2. **Check for Connection Issues:**
   ```powershell
   # Get connection details
   az network vpn-connection list `
     --resource-group $resourceGroup `
     --query '[].{Name:name, Status:connectionStatus, Type:connectionType}' `
     --output table
   ```

#### Step 2: Test Effective Routes

1. **Check Effective Routes on Azure VM:**
   ```powershell
   $nicName = az vm nic list `
     --resource-group $resourceGroup `
     --vm-name vm-azure `
     --query '[0].id' `
     --output tsv | Split-Path -Leaf
   
   az network nic show-effective-route-table `
     --resource-group $resourceGroup `
     --name $nicName `
     --output table
   ```
   
   Look for routes to 192.168.0.0/16 via VirtualNetworkGateway

2. **Verify NSG Rules:**
   ```powershell
   az network nic list-effective-nsg `
     --resource-group $resourceGroup `
     --name $nicName `
     --output table
   ```

#### Step 3: Reset VPN Connection (if needed)

1. **Reset Connection:**
   ```powershell
   az network vpn-connection reset `
     --resource-group $resourceGroup `
     --name conn-azure-to-onprem
   ```

2. **Wait and Check Status:**
   ```powershell
   Start-Sleep -Seconds 30
   
   az network vpn-connection show `
     --resource-group $resourceGroup `
     --name conn-azure-to-onprem `
     --query connectionStatus
   ```

#### Step 4: Capture Packet Traces (Advanced)

1. **Start Packet Capture on Gateway:**
   ```powershell
   # Requires Azure CLI 2.15.0+
   az network watcher packet-capture create `
     --resource-group $resourceGroup `
     --name vpn-capture `
     --target $(az network vnet-gateway show --resource-group $resourceGroup --name vpn-gateway-azure --query id -o tsv) `
     --filters '[{"protocol":"TCP","localIPAddress":"10.1.0.0/16","remoteIPAddress":"192.168.0.0/16"}]'
   ```

### Verification

- [ ] Connection status verified
- [ ] Effective routes checked
- [ ] NSG rules reviewed
- [ ] Know how to reset connections
- [ ] Understand packet capture capability

---

## Task 8: Clean Up Resources

### Instructions

1. **Disconnect Point-to-Site VPN:**
   - Settings → Network & Internet → VPN
   - Click **Disconnect**

2. **Delete Resource Group:**
   ```powershell
   az group delete --name $resourceGroup --yes --no-wait
   ```
   
   Note: VPN Gateways take 10-20 minutes to delete

3. **Remove VPN Client:**
   - Settings → Apps → Apps & features
   - Find VPN client
   - Click **Uninstall**

4. **Remove Certificates (Optional):**
   ```powershell
   # Remove certificates from personal store
   Get-ChildItem Cert:\CurrentUser\My | Where-Object {$_.Subject -like "*P2S*"} | Remove-Item
   ```

### Verification

- [ ] Disconnected from Point-to-Site VPN
- [ ] Resource group deletion initiated
- [ ] VPN client uninstalled
- [ ] Certificates removed

---

## Summary

In this lab, you have:
- ✅ Deployed Azure VPN Gateway infrastructure
- ✅ Configured site-to-site VPN between Azure and simulated on-premises
- ✅ Tested connectivity across VPN tunnel
- ✅ Configured point-to-site VPN for remote access
- ✅ Generated and uploaded certificates for P2S authentication
- ✅ Monitored VPN gateway metrics and diagnostic logs
- ✅ Troubleshot VPN connectivity issues

## Key Takeaways

- VPN Gateway requires a dedicated subnet named `GatewaySubnet`
- VPN Gateway deployment takes 20-45 minutes
- Site-to-site VPN requires local network gateways and matching shared keys
- Point-to-site VPN uses certificate-based authentication
- Effective routes show VPN routes as VirtualNetworkGateway next hop
- Diagnostic logs provide detailed troubleshooting information
- VPN Gateway SKUs determine throughput and feature availability

## Additional Resources

- [VPN Gateway Documentation](https://docs.microsoft.com/azure/vpn-gateway/)
- [Plan and Design VPN Gateway](https://learn.microsoft.com/azure/vpn-gateway/design)
- [VPN Gateway Configuration Settings](https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)
- [Troubleshoot VPN Gateway](https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-troubleshoot)

[← Back to Module Overview](../README.md) | [Next: Lab 03 - Application Gateway →](../lab03-application-gateway/README.md)
