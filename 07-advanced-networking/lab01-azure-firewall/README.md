# Lab 01: Azure Firewall and Firewall Manager

## Lab Overview

In this lab, you'll deploy and configure Azure Firewall to secure network traffic. You'll create firewall rules, implement forced tunneling through the firewall, and monitor traffic logs.

## Objectives

After completing this lab, you will be able to:
- Deploy Azure Firewall
- Configure application rules to control outbound HTTP/HTTPS traffic
- Configure network rules for non-HTTP traffic
- Route traffic through the firewall using route tables
- Monitor and analyze firewall logs

## Prerequisites

- Azure subscription with Contributor access
- VS Code with Azure extensions
- Basic understanding of VNets and subnets

## Architecture

```
Internet
    ↓
Azure Firewall (10.0.1.0/26)
    ↓
Workload Subnet (10.0.2.0/24)
    ↓
Virtual Machines
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
   # Set variables
   $resourceGroup = "rg-firewall-lab"
   $location = "eastus"
   
   # Create resource group
   az group create --name $resourceGroup --location $location
   ```

#### Step 2: Create Virtual Network

1. **Create VNet with Multiple Subnets:**
   ```powershell
   # Create VNet
   az network vnet create `
     --resource-group $resourceGroup `
     --name vnet-firewall `
     --address-prefix 10.0.0.0/16 `
     --location $location
   
   # Create AzureFirewallSubnet (required name)
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall `
     --name AzureFirewallSubnet `
     --address-prefix 10.0.1.0/26
   
   # Create workload subnet
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall `
     --name snet-workload `
     --address-prefix 10.0.2.0/24
   ```

2. **Verify Subnets:**
   ```powershell
   az network vnet subnet list `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall `
     --output table
   ```

### Verification

- [ ] Resource group created
- [ ] VNet created with 10.0.0.0/16 address space
- [ ] AzureFirewallSubnet exists (10.0.1.0/26)
- [ ] Workload subnet exists (10.0.2.0/24)

---

## Task 2: Deploy Azure Firewall

### Instructions

#### Step 1: Create Public IP for Firewall

1. **Create Public IP:**
   ```powershell
   az network public-ip create `
     --resource-group $resourceGroup `
     --name pip-firewall `
     --sku Standard `
     --allocation-method Static `
     --location $location
   ```

2. **Get Public IP Address:**
   ```powershell
   az network public-ip show `
     --resource-group $resourceGroup `
     --name pip-firewall `
     --query ipAddress `
     --output tsv
   ```
   
   Note this IP address for later use.

#### Step 2: Deploy Azure Firewall

1. **Create Firewall:**
   ```powershell
   # Create firewall (takes 5-10 minutes)
   az network firewall create `
     --resource-group $resourceGroup `
     --name fw-lab `
     --location $location `
     --enable-dns-proxy true
   ```

2. **Configure Firewall IP:**
   ```powershell
   az network firewall ip-config create `
     --firewall-name fw-lab `
     --name fw-config `
     --public-ip-address pip-firewall `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall
   ```

3. **Get Firewall Private IP:**
   ```powershell
   $fwPrivateIp = az network firewall show `
     --resource-group $resourceGroup `
     --name fw-lab `
     --query 'ipConfigurations[0].privateIPAddress' `
     --output tsv
   
   Write-Host "Firewall Private IP: $fwPrivateIp"
   ```

#### Step 3: Verify Deployment in Portal

1. **Open Azure Portal:**
   - Go to [https://portal.azure.com](https://portal.azure.com)
   - Search for "Firewalls"
   - Click on `fw-lab`

2. **Verify Configuration:**
   - Check Status: Should show "Provisioning succeeded"
   - Check Public IP address
   - Check Private IP address
   - Note the Firewall Policy (will configure next)

### Verification

- [ ] Public IP created with Standard SKU
- [ ] Azure Firewall deployed successfully
- [ ] Firewall has both public and private IP addresses
- [ ] Firewall status shows "Succeeded"

---

## Task 3: Configure Firewall Application Rules

### Instructions

#### Step 1: Create Application Rule Collection

1. **Allow Access to Microsoft Sites:**
   ```powershell
   # Create application rule collection
   az network firewall application-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name app-coll-01 `
     --name allow-microsoft `
     --protocols Https=443 Http=80 `
     --source-addresses 10.0.2.0/24 `
     --target-fqdns "*.microsoft.com" "*.windows.com" `
     --priority 100 `
     --action Allow
   ```

2. **Allow Access to Azure Services:**
   ```powershell
   az network firewall application-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name app-coll-01 `
     --name allow-azure `
     --protocols Https=443 `
     --source-addresses 10.0.2.0/24 `
     --target-fqdns "*.azure.com" "*.azure.net"
   ```

3. **Allow Access to Ubuntu Updates:**
   ```powershell
   az network firewall application-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name app-coll-01 `
     --name allow-ubuntu `
     --protocols Http=80 Https=443 `
     --source-addresses 10.0.2.0/24 `
     --target-fqdns "*.ubuntu.com" "security.ubuntu.com" "archive.ubuntu.com"
   ```

#### Step 2: Verify Application Rules

1. **List Application Rules:**
   ```powershell
   az network firewall application-rule list `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name app-coll-01 `
     --output table
   ```

2. **Verify in Portal:**
   - Navigate to firewall `fw-lab`
   - Click "Rules (classic)" in left menu
   - Click "Application rule collection"
   - Verify `app-coll-01` exists with 3 rules

### Verification

- [ ] Application rule collection created
- [ ] Microsoft sites rule configured
- [ ] Azure services rule configured
- [ ] Ubuntu updates rule configured
- [ ] Rules visible in portal

---

## Task 4: Configure Firewall Network Rules

### Instructions

#### Step 1: Create Network Rule Collection

1. **Allow DNS Traffic:**
   ```powershell
   az network firewall network-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name net-coll-01 `
     --name allow-dns `
     --protocols UDP `
     --source-addresses 10.0.2.0/24 `
     --destination-addresses 168.63.129.16 `
     --destination-ports 53 `
     --priority 100 `
     --action Allow
   ```

2. **Allow NTP Traffic:**
   ```powershell
   az network firewall network-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name net-coll-01 `
     --name allow-ntp `
     --protocols UDP `
     --source-addresses 10.0.2.0/24 `
     --destination-addresses "*" `
     --destination-ports 123
   ```

#### Step 2: Verify Network Rules

1. **List Network Rules:**
   ```powershell
   az network firewall network-rule list `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name net-coll-01 `
     --output table
   ```

2. **Verify in Portal:**
   - Navigate to firewall rules
   - Click "Network rule collection"
   - Verify `net-coll-01` exists with DNS and NTP rules

### Verification

- [ ] Network rule collection created
- [ ] DNS rule configured (port 53)
- [ ] NTP rule configured (port 123)
- [ ] Rules visible in portal

---

## Task 5: Create Test VM and Route Table

### Instructions

#### Step 1: Create Test Virtual Machine

1. **Create VM:**
   ```powershell
   # Create VM in workload subnet
   az vm create `
     --resource-group $resourceGroup `
     --name vm-test `
     --vnet-name vnet-firewall `
     --subnet snet-workload `
     --image Ubuntu2204 `
     --admin-username azureuser `
     --generate-ssh-keys `
     --size Standard_B2s `
     --public-ip-address "" `
     --nsg ""
   ```
   
   Note: No public IP or NSG (traffic goes through firewall)

2. **Get VM Private IP:**
   ```powershell
   az vm show `
     --resource-group $resourceGroup `
     --name vm-test `
     --show-details `
     --query privateIps `
     --output tsv
   ```

#### Step 2: Create Route Table

1. **Create Route Table:**
   ```powershell
   az network route-table create `
     --resource-group $resourceGroup `
     --name rt-firewall `
     --location $location
   ```

2. **Create Route to Force Traffic Through Firewall:**
   ```powershell
   # Get firewall private IP
   $fwPrivateIp = az network firewall show `
     --resource-group $resourceGroup `
     --name fw-lab `
     --query 'ipConfigurations[0].privateIPAddress' `
     --output tsv
   
   # Create default route
   az network route-table route create `
     --resource-group $resourceGroup `
     --route-table-name rt-firewall `
     --name route-to-internet `
     --address-prefix 0.0.0.0/0 `
     --next-hop-type VirtualAppliance `
     --next-hop-ip-address $fwPrivateIp
   ```

3. **Associate Route Table with Subnet:**
   ```powershell
   az network vnet subnet update `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall `
     --name snet-workload `
     --route-table rt-firewall
   ```

#### Step 3: Verify Routing

1. **Check Effective Routes:**
   ```powershell
   # Get VM NIC name
   $nicName = az vm nic list `
     --resource-group $resourceGroup `
     --vm-name vm-test `
     --query '[0].id' `
     --output tsv | Split-Path -Leaf
   
   # View effective routes
   az network nic show-effective-route-table `
     --resource-group $resourceGroup `
     --name $nicName `
     --output table
   ```
   
   Look for route with:
   - Address Prefix: 0.0.0.0/0
   - Next Hop Type: VirtualAppliance
   - Next Hop IP: (Firewall private IP)

### Verification

- [ ] Test VM created without public IP
- [ ] Route table created
- [ ] Default route points to firewall
- [ ] Route table associated with workload subnet
- [ ] Effective routes show firewall as next hop

---

## Task 6: Test Firewall Rules

### Instructions

#### Step 1: Deploy Azure Bastion for Access

1. **Create Bastion Subnet:**
   ```powershell
   az network vnet subnet create `
     --resource-group $resourceGroup `
     --vnet-name vnet-firewall `
     --name AzureBastionSubnet `
     --address-prefix 10.0.3.0/26
   ```

2. **Create Bastion Public IP:**
   ```powershell
   az network public-ip create `
     --resource-group $resourceGroup `
     --name pip-bastion `
     --sku Standard `
     --location $location
   ```

3. **Create Bastion (takes 5-10 minutes):**
   ```powershell
   az network bastion create `
     --resource-group $resourceGroup `
     --name bastion-lab `
     --public-ip-address pip-bastion `
     --vnet-name vnet-firewall `
     --location $location
   ```

#### Step 2: Connect to VM via Bastion

1. **Connect Using Portal:**
   - Go to Azure Portal
   - Navigate to `vm-test`
   - Click **Connect** → **Bastion**
   - Enter username: `azureuser`
   - Select SSH Private Key
   - Paste your private key (from `~/.ssh/id_rsa`)
   - Click **Connect**

2. **Wait for Connection:**
   - Bastion opens browser-based SSH session

#### Step 3: Test Allowed Traffic

1. **Test Microsoft.com (Should Succeed):**
   ```bash
   curl -I https://www.microsoft.com
   ```
   
   Expected: HTTP 200 response

2. **Test Azure.com (Should Succeed):**
   ```bash
   curl -I https://azure.microsoft.com
   ```
   
   Expected: HTTP 200 response

3. **Test Ubuntu Updates (Should Succeed):**
   ```bash
   sudo apt update
   ```
   
   Expected: Successfully fetches package lists

#### Step 4: Test Blocked Traffic

1. **Test Blocked Website (Should Fail):**
   ```bash
   curl -I https://www.google.com --connect-timeout 5
   ```
   
   Expected: Connection timeout (not in firewall rules)

2. **Test SSH to External IP (Should Fail):**
   ```bash
   ssh user@8.8.8.8 -o ConnectTimeout=5
   ```
   
   Expected: Connection timeout (no network rule for SSH)

### Verification

- [ ] Azure Bastion deployed
- [ ] Can connect to VM via Bastion
- [ ] Can access Microsoft.com
- [ ] Can access Azure.com
- [ ] Can run apt update
- [ ] Cannot access google.com
- [ ] Cannot SSH to external IPs

---

## Task 7: Monitor Firewall Logs

### Instructions

#### Step 1: Enable Diagnostic Settings

1. **Create Log Analytics Workspace:**
   ```powershell
   az monitor log-analytics workspace create `
     --resource-group $resourceGroup `
     --workspace-name law-firewall `
     --location $location
   ```

2. **Get Workspace ID:**
   ```powershell
   $workspaceId = az monitor log-analytics workspace show `
     --resource-group $resourceGroup `
     --workspace-name law-firewall `
     --query id `
     --output tsv
   ```

3. **Enable Firewall Diagnostics:**
   ```powershell
   $fwId = az network firewall show `
     --resource-group $resourceGroup `
     --name fw-lab `
     --query id `
     --output tsv
   
   az monitor diagnostic-settings create `
     --name firewall-diagnostics `
     --resource $fwId `
     --workspace $workspaceId `
     --logs '[{"category":"AzureFirewallApplicationRule","enabled":true},{"category":"AzureFirewallNetworkRule","enabled":true}]' `
     --metrics '[{"category":"AllMetrics","enabled":true}]'
   ```

#### Step 2: Generate Traffic and View Logs

1. **Generate Some Traffic:**
   - Reconnect to VM via Bastion
   - Run several curl commands:
   ```bash
   curl https://www.microsoft.com
   curl https://azure.microsoft.com
   curl https://www.google.com
   ```

2. **Wait for Logs (5-10 minutes):**
   - Logs take time to populate

3. **Query Application Rule Logs:**
   ```powershell
   # In Azure Portal
   # Navigate to firewall → Logs
   # Run this KQL query:
   ```
   
   KQL Query:
   ```kql
   AzureDiagnostics
   | where Category == "AzureFirewallApplicationRule"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, msg_s
   | limit 20
   ```

4. **Query Network Rule Logs:**
   ```kql
   AzureDiagnostics
   | where Category == "AzureFirewallNetworkRule"
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, msg_s
   | limit 20
   ```

#### Step 3: View Firewall Metrics

1. **View in Portal:**
   - Navigate to firewall `fw-lab`
   - Click **Metrics** in left menu
   - Add metrics:
     - Application rules hit count
     - Network rules hit count
     - Data processed
     - Firewall health state

2. **Create Alert (Optional):**
   - Click **New alert rule**
   - Condition: Application rules hit count > 100
   - Action: Send email notification

### Verification

- [ ] Log Analytics workspace created
- [ ] Diagnostic settings enabled on firewall
- [ ] Can see application rule logs
- [ ] Can see network rule logs
- [ ] Metrics visible in Azure Monitor
- [ ] Logs show allowed and denied traffic

---

## Task 8: Implement NAT Rules (Optional)

### Instructions

#### Step 1: Create DNAT Rule

1. **Create NAT Rule Collection:**
   ```powershell
   az network firewall nat-rule create `
     --firewall-name fw-lab `
     --resource-group $resourceGroup `
     --collection-name nat-coll-01 `
     --name ssh-nat `
     --protocols TCP `
     --source-addresses "*" `
     --destination-addresses $(az network public-ip show --resource-group $resourceGroup --name pip-firewall --query ipAddress -o tsv) `
     --destination-ports 2222 `
     --translated-address 10.0.2.4 `
     --translated-port 22 `
     --priority 100 `
     --action Dnat
   ```
   
   This allows SSH to VM via firewall public IP on port 2222.

2. **Test DNAT Rule:**
   ```powershell
   # From your local machine (if allowed by your NSG)
   $firewallPublicIp = az network public-ip show `
     --resource-group $resourceGroup `
     --name pip-firewall `
     --query ipAddress `
     --output tsv
   
   ssh azureuser@$firewallPublicIp -p 2222
   ```

### Verification

- [ ] NAT rule collection created
- [ ] DNAT rule allows SSH via firewall public IP
- [ ] Can connect to VM via translated address

---

## Task 9: Clean Up Resources

### Instructions

1. **Delete Resource Group:**
   ```powershell
   # This deletes ALL resources in the group
   az group delete --name $resourceGroup --yes --no-wait
   ```

2. **Verify Deletion:**
   ```powershell
   # Check deletion status
   az group show --name $resourceGroup
   ```
   
   Expected: ResourceGroupNotFound error

### Verification

- [ ] Resource group deletion initiated
- [ ] All resources will be removed

---

## Summary

In this lab, you have:
- ✅ Created virtual network infrastructure for Azure Firewall
- ✅ Deployed Azure Firewall with public and private IPs
- ✅ Configured application rules to control HTTP/HTTPS traffic
- ✅ Configured network rules for DNS and NTP
- ✅ Implemented forced tunneling using route tables
- ✅ Tested allowed and blocked traffic
- ✅ Enabled and queried firewall diagnostic logs
- ✅ Monitored firewall metrics in Azure Monitor

## Key Takeaways

- Azure Firewall requires a dedicated subnet named `AzureFirewallSubnet`
- Application rules control HTTP/HTTPS traffic based on FQDNs
- Network rules control non-HTTP traffic based on IP/port
- Route tables force traffic through the firewall
- Logs and metrics provide visibility into firewall activity
- Azure Bastion provides secure access without public IPs

## Additional Resources

- [Azure Firewall Documentation](https://docs.microsoft.com/azure/firewall/)
- [Firewall Rule Processing Logic](https://learn.microsoft.com/azure/firewall/rule-processing)
- [Azure Firewall Logs and Metrics](https://learn.microsoft.com/azure/firewall/logs-and-metrics)

[← Back to Module Overview](../README.md) | [Next: Lab 02 - VPN Gateway →](../lab02-vpn-gateway/README.md)
