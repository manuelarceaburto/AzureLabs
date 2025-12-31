# Lab 02: Implement Network Security Groups and Application Security Groups

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure Network Security Groups
- [ ] Implement inbound and outbound rules
- [ ] Create and manage Application Security Groups
- [ ] Implement network segmentation
- [ ] Monitor and troubleshoot NSG rules

## Prerequisites

- Virtual networks and VMs from previous labs (or create new ones)
- Azure Portal or PowerShell/CLI
- Resource group: RG-Network
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation needs to implement network security controls to restrict traffic between different tiers of their application (web, app, database). You'll create NSGs and ASGs to implement network segmentation.

## Architecture

```
Virtual Network (vnet-prod)
├── Subnet: subnet-web
│   ├── NSG: NSG-Web
│   ├── ASG: asg-web
│   └── VMs: web servers
├── Subnet: subnet-app
│   ├── NSG: NSG-App
│   ├── ASG: asg-app
│   └── VMs: app servers
└── Subnet: subnet-db
    ├── NSG: NSG-DB
    ├── ASG: asg-db
    └── VMs: database servers

Traffic Rules:
- Internet → Web (80, 443)
- Web → App (8080)
- App → DB (1433)
- DB → nothing (isolated)
```

## Task 1: Create Network Security Groups

### Instructions

1. Navigate to **Network security groups** in Azure Portal
2. Click **+ Create**
3. Create NSG-Web:
   - **Name**: NSG-Web
   - **Resource group**: RG-Network
   - **Location**: East US
   - Click **Create**

4. Create NSG-App:
   - **Name**: NSG-App
   - Same resource group and location

5. Create NSG-DB:
   - **Name**: NSG-DB
   - Same resource group and location

6. Associate NSGs with subnets:
   - Go to **NSG-Web** > **Subnets**
   - Click **+ Associate**
   - **Virtual network**: vnet-prod
   - **Subnet**: subnet-web
   - Click **OK**
   
   Repeat for NSG-App (subnet-app) and NSG-DB (subnet-db)

### Verification

- [ ] All three NSGs are created
- [ ] NSGs are associated with correct subnets
- [ ] Can see association in NSG properties
- [ ] Default rules are present

### PowerShell Example

```powershell
# Create NSGs
$nsgWeb = New-AzNetworkSecurityGroup -ResourceGroupName "RG-Network" `
    -Name "NSG-Web" `
    -Location "eastus"

$nsgApp = New-AzNetworkSecurityGroup -ResourceGroupName "RG-Network" `
    -Name "NSG-App" `
    -Location "eastus"

$nsgDb = New-AzNetworkSecurityGroup -ResourceGroupName "RG-Network" `
    -Name "NSG-DB" `
    -Location "eastus"

# Associate NSG with subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "RG-Network"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "subnet-web" -VirtualNetwork $vnet
$subnet.NetworkSecurityGroup = $nsgWeb
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

## Task 2: Configure Inbound Rules

### Instructions

1. Configure NSG-Web inbound rules:
   - Go to **NSG-Web** > **Inbound security rules**
   - Click **+ Add**
   
   Rule 1 - Allow HTTP from Internet:
   - **Source**: Any
   - **Source port ranges**: *
   - **Destination**: Any
   - **Destination port ranges**: 80
   - **Protocol**: TCP
   - **Action**: Allow
   - **Priority**: 100
   - **Name**: AllowHTTP
   - Click **Add**

   Rule 2 - Allow HTTPS:
   - Same as above but port 443
   - **Priority**: 110

   Rule 3 - Allow RDP (for management):
   - **Source**: Your IP (specific)
   - **Destination port**: 3389
   - **Priority**: 120

2. Configure NSG-App inbound rules:
   - Allow traffic from Web tier (8080):
   - **Source**: Application Security Group (asg-web - to be created)
   - **Destination port**: 8080
   - **Priority**: 100

3. Configure NSG-DB inbound rules:
   - Allow traffic from App tier (SQL - 1433):
   - **Source**: Application Security Group (asg-app)
   - **Destination port**: 1433
   - **Priority**: 100

4. Delete or update default allow rule:
   - Review default **AllowVnetInBound** rule
   - Keep it for internal VNet communication

### Verification

- [ ] All inbound rules are created
- [ ] Rules are listed in correct order
- [ ] Priorities are unique
- [ ] Rules match network requirements
- [ ] Default rules are reviewed

### PowerShell Example

```powershell
# Create inbound rules
$nsg = Get-AzNetworkSecurityGroup -Name "NSG-Web" -ResourceGroupName "RG-Network"

# HTTP rule
$rule = New-AzNetworkSecurityRuleConfig -Name "AllowHTTP" `
    -Description "Allow HTTP" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority 100 `
    -SourceAddressPrefix "*" `
    -SourcePortRange "*" `
    -DestinationAddressPrefix "*" `
    -DestinationPortRange 80

$nsg | Add-AzNetworkSecurityRuleConfig @rule | Set-AzNetworkSecurityGroup

# HTTPS rule
$rule2 = New-AzNetworkSecurityRuleConfig -Name "AllowHTTPS" `
    -Description "Allow HTTPS" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority 110 `
    -SourceAddressPrefix "*" `
    -SourcePortRange "*" `
    -DestinationAddressPrefix "*" `
    -DestinationPortRange 443

$nsg | Add-AzNetworkSecurityRuleConfig @rule2 | Set-AzNetworkSecurityGroup
```

## Task 3: Create and Configure Application Security Groups

### Instructions

1. Navigate to **Application security groups** in Azure Portal
2. Click **+ Create**
3. Create ASG-Web:
   - **Name**: asg-web
   - **Resource group**: RG-Network
   - **Location**: East US
   - Click **Create**

4. Create ASG-App and ASG-DB similarly

5. Add network interfaces to ASGs:
   - Go to **asg-web** > **Associated network interfaces**
   - Click **+ Associate**
   - Select network interfaces of web VMs
   - Click **Save**
   
   Repeat for asg-app and asg-db

6. Create NSG rule using ASG as source:
   - Go to **NSG-App** > **Inbound security rules**
   - Click **+ Add**
   - **Source**: Application Security Group
   - **Source ASG**: asg-web
   - **Destination port**: 8080
   - **Protocol**: TCP
   - **Action**: Allow
   - **Priority**: 100

### Verification

- [ ] All ASGs are created
- [ ] Network interfaces are associated
- [ ] NSG rules reference ASGs
- [ ] Can see membership in each ASG
- [ ] Traffic follows ASG-based rules

### PowerShell Example

```powershell
# Create ASG
$asgWeb = New-AzApplicationSecurityGroup -ResourceGroupName "RG-Network" `
    -Name "asg-web" `
    -Location "eastus"

# Get VM NIC
$vm = Get-AzVM -ResourceGroupName "RG-Compute" -Name "VM-Windows"
$nic = Get-AzNetworkInterface -ResourceId $vm.NetworkProfile.NetworkInterfaces[0].Id

# Associate NIC with ASG
$asg = Get-AzApplicationSecurityGroup -Name "asg-web" -ResourceGroupName "RG-Network"
$nic.IpConfigurations[0].ApplicationSecurityGroups.Add($asg)
$nic | Set-AzNetworkInterface

# Create rule using ASG
$nsgApp = Get-AzNetworkSecurityGroup -Name "NSG-App" -ResourceGroupName "RG-Network"
Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsgApp `
    -Name "AllowFromWeb" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority 100 `
    -SourceApplicationSecurityGroup $asgWeb `
    -DestinationApplicationSecurityGroup $asgApp `
    -DestinationPortRange 8080
```

## Task 4: Configure Outbound Rules

### Instructions

1. Review default outbound rules:
   - By default, all outbound traffic is allowed
   - This can be customized for security

2. Configure restrictive outbound rules:
   - Go to **NSG-Web** > **Outbound security rules**
   - Default rule allows all outbound
   - Example: Restrict web tier to only app tier

3. Add outbound rule:
   - **Name**: AllowToApp
   - **Destination**: Application Security Group (asg-app)
   - **Destination port**: 8080
   - **Priority**: 100
   - **Action**: Allow

4. Add rule to block all other outbound (if desired):
   - **Name**: DenyAllOutbound
   - **Destination**: *
   - **Priority**: 200
   - **Action**: Deny

### Verification

- [ ] Outbound rules are configured
- [ ] Rules allow necessary traffic
- [ ] Rules deny unnecessary traffic
- [ ] Default deny rule has lowest priority
- [ ] Traffic patterns are enforced

## Task 5: Test and Troubleshoot NSG Rules

### Instructions

1. Deploy test VMs in each subnet:
   - VM in subnet-web
   - VM in subnet-app
   - VM in subnet-db

2. Test connectivity:
   - From web VM, ping app VM (should succeed if rules allow)
   - From app VM, ping DB VM (should succeed if rules allow)
   - From DB VM, ping external (should fail - no outbound allowed)

3. Use Network Watcher for troubleshooting:
   - Go to **Network Watcher** > **IP flow verify**
   - Test connectivity between VMs
   - View detailed rule evaluation

4. Check effective security rules:
   - Go to VM NIC > **Effective security rules**
   - See all rules that apply to this interface
   - Identify which rules block/allow traffic

5. Enable NSG flow logs:
   - Go to **Network Watcher** > **NSG flow logs**
   - Enable for each NSG
   - Configure storage account for logs
   - Review traffic patterns

### Verification

- [ ] Can test connectivity from VMs
- [ ] Network Watcher IP flow verify works
- [ ] Effective rules are visible
- [ ] Flow logs capture traffic
- [ ] Troubleshooting identifies issues correctly

### Azure CLI Example

```bash
# Get effective security rules
az network nic list-effective-network-security-groups \
    --resource-group RG-Compute \
    --name vm-nic

# Test IP connectivity
az network watcher test-ip-flow \
    --resource-group RG-Network \
    --direction Inbound \
    --protocol TCP \
    --local 10.0.1.4:80 \
    --remote 10.0.1.1:12345
```

## Cleanup

Remove NSGs and ASGs:

```powershell
# Disassociate NSG from subnet
$subnet.NetworkSecurityGroup = $null
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Remove NSGs
Remove-AzNetworkSecurityGroup -Name "NSG-Web" -ResourceGroupName "RG-Network" -Force
Remove-AzNetworkSecurityGroup -Name "NSG-App" -ResourceGroupName "RG-Network" -Force
Remove-AzNetworkSecurityGroup -Name "NSG-DB" -ResourceGroupName "RG-Network" -Force

# Remove ASGs
Remove-AzApplicationSecurityGroup -Name "asg-web" -ResourceGroupName "RG-Network" -Force
Remove-AzApplicationSecurityGroup -Name "asg-app" -ResourceGroupName "RG-Network" -Force
Remove-AzApplicationSecurityGroup -Name "asg-db" -ResourceGroupName "RG-Network" -Force
```

## Review

In this lab, you:
- Created Network Security Groups for network segmentation
- Configured inbound and outbound rules
- Implemented Application Security Groups for dynamic rule management
- Tested connectivity and troubleshot issues
- Enabled monitoring with flow logs

NSGs and ASGs are critical for implementing network security in Azure.

## Additional Resources

- [Network Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Application Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/application-security-groups)
- [Network Watcher](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview)
- [IP Flow Verify](https://learn.microsoft.com/en-us/azure/network-watcher/ip-flow-verify-overview)
- [NSG Flow Logs](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-overview)

## Notes for Instructors

- NSG rules are evaluated in priority order (lowest number = highest priority)
- Default deny rule is always last (65535)
- ASGs simplify rule management for multiple VMs
- Flow logs are valuable for security auditing
- Test rules thoroughly before production deployment
- Document NSG rules and their purposes
- Consider using Azure Firewall for more advanced scenarios
- Review rules regularly for unnecessary permissions

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
