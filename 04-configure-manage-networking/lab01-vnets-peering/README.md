# Lab 01: Implement Virtual Networks and Peering

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure virtual networks
- [ ] Create and manage subnets
- [ ] Implement network peering
- [ ] Configure network routing
- [ ] Manage virtual network security

## Prerequisites

- Active Azure subscription
- Resource group created
- Basic understanding of networking concepts
- Azure Portal or PowerShell/CLI
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation has multiple business units that need separate virtual networks but require secure communication between them. You'll create virtual networks, implement peering, and configure routing.

## Architecture

```
Azure Subscription
├── Virtual Network: vnet-prod (10.0.0.0/16)
│   ├── Subnet: subnet-web (10.0.1.0/24)
│   ├── Subnet: subnet-db (10.0.2.0/24)
│   └── Subnet: GatewaySubnet (10.0.3.0/24)
├── Virtual Network: vnet-dev (10.1.0.0/16)
│   ├── Subnet: subnet-web (10.1.1.0/24)
│   └── Subnet: subnet-db (10.1.2.0/24)
├── VNet Peering: prod-dev-peer (vnet-prod <-> vnet-dev)
└── Route Tables
    ├── rt-prod
    └── rt-dev
```

## Task 1: Create Virtual Networks

### Instructions

1. Navigate to **Virtual networks** in Azure Portal
2. Click **+ Create**
3. Create vnet-prod:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Network (create new)
   - **Name**: vnet-prod
   - **Region**: East US
   - **IPv4 address space**: 10.0.0.0/16
   - Click **Next: IP Addresses**
   
4. Add subnets for vnet-prod:
   - **Subnet 1**: subnet-web (10.0.1.0/24)
   - **Subnet 2**: subnet-db (10.0.2.0/24)
   - **Subnet 3**: GatewaySubnet (10.0.3.0/24)
   - Click **Next: Security**
   - Leave defaults
   - Click **Review + create**

5. Create vnet-dev:
   - Repeat steps with these settings:
   - **Name**: vnet-dev
   - **IPv4 address space**: 10.1.0.0/16
   - **Subnets**:
     - subnet-web (10.1.1.0/24)
     - subnet-db (10.1.2.0/24)

### Verification

- [ ] Both virtual networks are created
- [ ] Subnets are created with correct address ranges
- [ ] No overlapping address spaces
- [ ] Can see all subnets in each VNet
- [ ] Resource group contains all VNets

### PowerShell Example

```powershell
# Create resource group
New-AzResourceGroup -Name "RG-Network" -Location "eastus"

# Create vnet-prod with subnets
$subnet1 = New-AzVirtualNetworkSubnetConfig -Name "subnet-web" `
    -AddressPrefix "10.0.1.0/24"
$subnet2 = New-AzVirtualNetworkSubnetConfig -Name "subnet-db" `
    -AddressPrefix "10.0.2.0/24"
$subnet3 = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" `
    -AddressPrefix "10.0.3.0/24"

$vnetProd = New-AzVirtualNetwork -ResourceGroupName "RG-Network" `
    -Name "vnet-prod" `
    -AddressPrefix "10.0.0.0/16" `
    -Location "eastus" `
    -Subnet $subnet1, $subnet2, $subnet3

# Create vnet-dev similarly
$subnet1Dev = New-AzVirtualNetworkSubnetConfig -Name "subnet-web" `
    -AddressPrefix "10.1.1.0/24"
$subnet2Dev = New-AzVirtualNetworkSubnetConfig -Name "subnet-db" `
    -AddressPrefix "10.1.2.0/24"

$vnetDev = New-AzVirtualNetwork -ResourceGroupName "RG-Network" `
    -Name "vnet-dev" `
    -AddressPrefix "10.1.0.0/16" `
    -Location "eastus" `
    -Subnet $subnet1Dev, $subnet2Dev
```

## Task 2: Implement Virtual Network Peering

### Instructions

1. Navigate to **vnet-prod**
2. Go to **Peerings** under **Settings**
3. Click **+ Add**
4. Configure peering:
   - **Peering name**: prod-dev-peer
   - **Virtual network deployment model**: Resource Manager
   - **Subscription**: Your subscription
   - **Virtual network**: vnet-dev
   - **Allow virtual network access**: Checked
   - **Allow forwarded traffic**: Checked
   - **Allow gateway transit**: Unchecked (not using gateways)
   - Click **OK**

5. Verify reciprocal peering:
   - Go to **vnet-dev** > **Peerings**
   - Observe that peering appears from dev side as well
   - Check status is "Connected"

### Verification

- [ ] Peering is created between vnet-prod and vnet-dev
- [ ] Status shows "Connected"
- [ ] Peering is visible from both sides
- [ ] Can ping VMs across peered networks (if deployed)
- [ ] Traffic flows between vnets

### PowerShell Example

```powershell
# Create vnet peering from prod to dev
Add-AzVirtualNetworkPeering -Name "prod-dev-peer" `
    -VirtualNetwork $vnetProd `
    -RemoteVirtualNetworkId $vnetDev.Id

# Verify peering status
Get-AzVirtualNetworkPeering -VirtualNetworkName "vnet-prod" `
    -ResourceGroupName "RG-Network"

# Get peering properties
$peering = Get-AzVirtualNetworkPeering -VirtualNetworkName "vnet-prod" `
    -ResourceGroupName "RG-Network" `
    -Name "prod-dev-peer"
$peering.PeeringState
```

## Task 3: Configure Network Routing

### Instructions

1. Create route table for vnet-prod:
   - Navigate to **Route tables**
   - Click **+ Create**
   - **Name**: rt-prod
   - **Resource group**: RG-Network
   - **Location**: East US
   - Click **Create**

2. Add routes to rt-prod:
   - Go to created route table > **Routes**
   - Click **+ Add**
   - **Route name**: route-to-dev
   - **Address prefix**: 10.1.0.0/16
   - **Next hop type**: Virtual network peering (or Virtual network gateway)
   - **Virtual network**: vnet-dev
   - Click **OK**

3. Associate route table with subnet:
   - Go to route table > **Subnets**
   - Click **+ Associate**
   - **Virtual network**: vnet-prod
   - **Subnet**: subnet-web
   - Click **OK**

4. Repeat for vnet-dev:
   - Create rt-dev route table
   - Add route to 10.0.0.0/16 pointing to vnet-prod
   - Associate with subnet-web in vnet-dev

### Verification

- [ ] Route tables are created
- [ ] Routes are configured correctly
- [ ] Route tables are associated with subnets
- [ ] Effective routes show in VMs
- [ ] Traffic follows configured routes

### PowerShell Example

```powershell
# Create route table
$routeTable = New-AzRouteTable -ResourceGroupName "RG-Network" `
    -Name "rt-prod" `
    -Location "eastus"

# Add route
Add-AzRouteConfig -Name "route-to-dev" `
    -RouteTable $routeTable `
    -AddressPrefix "10.1.0.0/16" `
    -NextHopType "VirtualNetworkPeering"

# Save route table
Set-AzRouteTable -RouteTable $routeTable

# Associate with subnet
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "subnet-web" `
    -VirtualNetwork $vnetProd
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnetProd `
    -Name "subnet-web" `
    -AddressPrefix "10.0.1.0/24" `
    -RouteTable $routeTable
$vnetProd | Set-AzVirtualNetwork
```

## Task 4: Manage VNet Connectivity

### Instructions

1. Deploy test VMs (optional):
   - Deploy one VM in vnet-prod/subnet-web
   - Deploy one VM in vnet-dev/subnet-web
   - Use VMs from previous compute labs or create new ones

2. Test connectivity:
   - From VM in prod, ping IP of VM in dev
   - From VM in dev, ping IP of VM in prod
   - Verify peering enables communication

3. Configure DNS:
   - Go to **vnet-prod** > **DNS servers**
   - Select "Custom" to add custom DNS
   - Add DNS server IPs if needed

4. Monitor VNet connectivity:
   - Use **Network Watcher** > **Connection troubleshoot**
   - Test connectivity between VMs
   - Identify any network issues

### Verification

- [ ] VMs can communicate across peered VNets
- [ ] Ping is successful
- [ ] DNS resolution works (if configured)
- [ ] Network Watcher troubleshooting is available

### Azure CLI Example

```bash
# Test connectivity using Network Watcher
az network watcher test-connectivity \
    --resource-group RG-Network \
    --source-resource vm-prod \
    --destination-resource vm-dev \
    --protocol TCP \
    --destination-port 80
```

## Task 5: Implement Advanced VNet Features (Optional)

### Instructions

1. Enable service endpoints:
   - Go to **subnet-db** in vnet-prod
   - Click **Service endpoints**
   - Enable: Microsoft.Storage
   - This allows direct access to storage without public IP

2. Configure network security:
   - Go to **vnet-prod** > **Subnets**
   - Configure Network Policies for subnets
   - Enforce network rules

3. Set up VNet flow logs:
   - Go to **Network Watcher** > **Flow logs**
   - Create flow log
   - Select vnet-prod
   - Capture traffic patterns

### Verification

- [ ] Service endpoints are enabled
- [ ] Network policies are applied
- [ ] Flow logs are collecting data
- [ ] Can analyze traffic patterns

## Cleanup

Remove VNets and related resources:

```powershell
# Remove peering
Remove-AzVirtualNetworkPeering -Name "prod-dev-peer" `
    -VirtualNetworkName "vnet-prod" `
    -ResourceGroupName "RG-Network" `
    -Force

# Remove VNets
Remove-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "vnet-prod" -Force
Remove-AzVirtualNetwork -ResourceGroupName "RG-Network" -Name "vnet-dev" -Force

# Remove route tables
Get-AzRouteTable -ResourceGroupName "RG-Network" | Remove-AzRouteTable -Force
```

## Review

In this lab, you:
- Created multiple virtual networks with subnets
- Implemented virtual network peering for cross-VNet communication
- Configured custom routing
- Tested connectivity between peered networks
- Managed VNet security and monitoring

Virtual networks are the foundation of Azure infrastructure and security.

## Additional Resources

- [Virtual Networks](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [Routing](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
- [Service Endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Network Watcher](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview)

## Notes for Instructors

- Plan IP address ranges carefully to avoid overlap
- Use standard IP ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- VNet peering is low-cost and low-latency
- Transitive peering is not supported by default
- Always test connectivity between peered networks
- Use Network Watcher for troubleshooting
- Document VNet topology and peering relationships
- Consider using VNet gateways for on-premises connectivity

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
