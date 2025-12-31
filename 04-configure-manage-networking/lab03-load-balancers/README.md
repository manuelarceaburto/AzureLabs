# Lab 03: Implement Load Balancers

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure Azure Load Balancers
- [ ] Configure health probes and load balancing rules
- [ ] Implement traffic distribution algorithms
- [ ] Configure high availability scenarios
- [ ] Monitor load balancer performance

## Prerequisites

- Virtual machines or VM scale sets from previous labs
- Virtual network: vnet-prod
- Resource group: RG-Network
- Azure Portal or PowerShell/CLI
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation has multiple web servers that need to distribute incoming traffic for high availability. You'll implement a load balancer to distribute traffic across multiple backend servers and ensure health monitoring.

## Architecture

```
Internet
     |
     v
Public IP
     |
     v
Azure Load Balancer
     |
     +--------+--------+--------+
     |        |        |        |
    VM1      VM2      VM3      VM4
(subnet-web)
- Health Probe
- Load Balancing Rules
- Inbound NAT Rules
```

## Task 1: Create Azure Load Balancer

### Instructions

1. Navigate to **Load balancers** in Azure Portal
2. Click **+ Create**
3. Configure load balancer:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Network
   - **Name**: lb-web
   - **Region**: East US
   - **Type**: Public
   - **SKU**: Standard
   - Click **Next: Frontend IP configuration**

4. Configure frontend IP:
   - **Frontend IP configuration name**: frontend-ip
   - **Public IP address**: Create new
   - **Public IP name**: pip-lb-web
   - **Assignment**: Static
   - Click **Next: Backend pools**

5. Configure backend pool:
   - **Name**: backend-pool
   - **Virtual network**: vnet-prod
   - **Backend pool configuration**: NIC
   - **Add virtual machines to backend pool**: 
     - Select VMs from subnet-web (if available)
   - Click **Next: Inbound rules**

6. Configure inbound rules:
   - Click **+ Add inbound rule**
   - **Name**: http-rule
   - **Frontend IP**: frontend-ip
   - **Frontend port**: 80
   - **Backend port**: 80
   - **Backend pool**: backend-pool
   - **Health probe**: Create new or select existing
   - **Session persistence**: None
   - **Idle timeout**: 4 minutes
   - Click **Add**

7. Review and create

### Verification

- [ ] Load balancer is created
- [ ] Public IP is assigned
- [ ] Backend pool contains VMs
- [ ] Health probe is configured
- [ ] Inbound rule is created
- [ ] Can access service via public IP

### PowerShell Example

```powershell
# Create public IP
$publicIP = New-AzPublicIpAddress -ResourceGroupName "RG-Network" `
    -Name "pip-lb-web" `
    -Location "eastus" `
    -AllocationMethod "Static" `
    -Sku "Standard"

# Create frontend IP configuration
$frontendConfig = New-AzLoadBalancerFrontendIpConfig -Name "frontend-ip" `
    -PublicIpAddress $publicIP

# Create backend pool
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "backend-pool"

# Create health probe
$healthProbe = New-AzLoadBalancerProbeConfig -Name "health-probe" `
    -Protocol "Http" `
    -Port 80 `
    -RequestPath "/" `
    -IntervalInSeconds 15 `
    -ProbeCount 2

# Create load balancing rule
$lbRule = New-AzLoadBalancerRuleConfig -Name "http-rule" `
    -FrontendIpConfiguration $frontendConfig `
    -BackendAddressPool $backendPool `
    -Probe $healthProbe `
    -Protocol "Tcp" `
    -FrontendPort 80 `
    -BackendPort 80

# Create load balancer
$lb = New-AzLoadBalancer -ResourceGroupName "RG-Network" `
    -Name "lb-web" `
    -Location "eastus" `
    -Sku "Standard" `
    -FrontendIpConfiguration $frontendConfig `
    -BackendAddressPool $backendPool `
    -Probe $healthProbe `
    -LoadBalancingRule $lbRule
```

## Task 2: Configure Health Probes

### Instructions

1. Navigate to **lb-web** > **Health probes**
2. Review existing health probe (created during LB creation)
3. Create additional health probe for HTTPS:
   - Click **+ Add**
   - **Name**: https-health-probe
   - **Protocol**: HTTPS
   - **Port**: 443
   - **Path**: /health
   - **Interval**: 15 seconds
   - **Unhealthy threshold**: 2 probes
   - Click **Add**

4. Configure health probe settings:
   - **Interval in seconds**: How often probe runs (15-300)
   - **Probe count**: Failed probes before unhealthy (2-10)
   - **Request path**: Health check endpoint ("/health")
   - **Protocol**: HTTP/HTTPS

5. Monitor probe status:
   - Go to **Backend pools**
   - Check health status of each VM
   - Verify probes are successful

### Verification

- [ ] Health probes are configured
- [ ] Probes show healthy status for VMs
- [ ] Probes fail for unavailable services
- [ ] Multiple probes can be created
- [ ] Interval and threshold settings are correct

### PowerShell Example

```powershell
# Create HTTPS health probe
$httpsProbe = New-AzLoadBalancerProbeConfig -Name "https-health-probe" `
    -Protocol "Https" `
    -Port 443 `
    -RequestPath "/health" `
    -IntervalInSeconds 15 `
    -ProbeCount 2

# Get load balancer and add probe
$lb = Get-AzLoadBalancer -Name "lb-web" -ResourceGroupName "RG-Network"
$lb.Probes.Add($httpsProbe)
Set-AzLoadBalancer -LoadBalancer $lb

# Get health status
Get-AzLoadBalancer -Name "lb-web" -ResourceGroupName "RG-Network" | 
    Get-AzLoadBalancerBackendAddressPoolConfig | 
    Get-AzNetworkInterfaceIpConfig
```

## Task 3: Implement Inbound NAT Rules

### Instructions

1. Navigate to **lb-web** > **Inbound NAT rules**
2. Create NAT rules for RDP access to backend VMs:
   - Click **+ Add**
   - **Name**: rdp-vm1
   - **Frontend IP address**: frontend-ip
   - **Frontend port**: 3391
   - **Backend port**: 3389
   - **Backend target**: VM-Windows (if available)
   - Click **Add**

3. Create NAT rule for second VM:
   - **Frontend port**: 3392
   - **Backend port**: 3389
   - **Backend target**: VM-Windows-2 (if available)

4. Create SSH rules for Linux VMs (if applicable):
   - **Frontend port**: 2201, 2202
   - **Backend port**: 22

5. Test NAT rules:
   - RDP: `mstsc /v:public-ip:3391`
   - SSH: `ssh user@public-ip -p 2201`

### Verification

- [ ] NAT rules are created
- [ ] Each VM has unique frontend port
- [ ] Can RDP/SSH using NAT port
- [ ] Port forwarding works correctly
- [ ] Multiple instances can be accessed

### PowerShell Example

```powershell
# Get VM network interface for NAT rule
$vm = Get-AzVM -Name "VM-Windows" -ResourceGroupName "RG-Compute"
$nic = Get-AzNetworkInterface -ResourceId $vm.NetworkProfile.NetworkInterfaces[0].Id

# Create NAT rule
$natRule = New-AzLoadBalancerInboundNatRuleConfig -Name "rdp-vm1" `
    -FrontendIpConfiguration $frontendConfig `
    -Protocol "Tcp" `
    -FrontendPort 3391 `
    -BackendPort 3389 `
    -BackendIpConfiguration $nic.IpConfigurations[0]

# Add to load balancer
$lb.InboundNatRules.Add($natRule)
Set-AzLoadBalancer -LoadBalancer $lb

# Associate NAT rule with NIC
$nic.IpConfigurations[0].LoadBalancerInboundNatRules.Add($natRule)
Set-AzNetworkInterface -NetworkInterface $nic
```

## Task 4: Configure Advanced Load Balancer Features

### Instructions

1. Configure session persistence (sticky sessions):
   - Go to **lb-web** > **Load balancing rules**
   - Edit http-rule
   - **Session persistence**: Client IP
   - This ensures same client always goes to same backend VM

2. Configure outbound rules (if needed):
   - Go to **Outbound rules**
   - Create rule for backend VMs to access internet
   - Select backend pool and outbound IP

3. Configure custom probes for specific paths:
   - Create health probe for `/api/health`
   - Use in rule for API traffic

4. Implement HA Ports (for all protocols):
   - Create rule with backend port: 0
   - This enables all ports to be load balanced

### Verification

- [ ] Session persistence is configured
- [ ] Outbound rules allow backend internet access
- [ ] Custom probes work correctly
- [ ] HA ports (if used) forward all traffic
- [ ] Advanced features are working

## Task 5: Monitor Load Balancer

### Instructions

1. Navigate to **lb-web** > **Monitoring** > **Metrics**
2. Create monitoring charts:
   - **Byte count** (ingress/egress)
   - **Packet count**
   - **SYN count** (for DDoS detection)
   - **Data Path Availability** (%)
   - **Health Probe Status** (per instance)

3. Set up alerts:
   - **Condition**: Data Path Availability < 100%
   - **Severity**: Critical
   - **Action**: Send email

4. Monitor backend pool health:
   - Go to **Backend pools**
   - View health status of each VM
   - Check probe success rate

5. View detailed logs:
   - Enable diagnostic logs
   - Select storage account for logs
   - Review access logs and health probe logs

### Verification

- [ ] Metrics dashboard is created
- [ ] Charts show traffic patterns
- [ ] Alerts are configured
- [ ] Backend health is visible
- [ ] Logs are being collected

## Cleanup

Remove load balancer and public IP:

```powershell
# Remove load balancer
Remove-AzLoadBalancer -Name "lb-web" -ResourceGroupName "RG-Network" -Force

# Remove public IP
Remove-AzPublicIpAddress -Name "pip-lb-web" -ResourceGroupName "RG-Network" -Force
```

Or Azure CLI:

```bash
# Remove load balancer
az network lb delete --resource-group RG-Network --name lb-web

# Remove public IP
az network public-ip delete --resource-group RG-Network --name pip-lb-web
```

## Review

In this lab, you:
- Created a public load balancer
- Configured backend pools and health probes
- Implemented load balancing rules
- Created inbound NAT rules for management access
- Monitored load balancer health and performance

Load balancers are essential for high availability and load distribution in Azure.

## Additional Resources

- [Azure Load Balancer](https://learn.microsoft.com/en-us/azure/load-balancer/)
- [Health Probes](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-custom-probe-overview)
- [Load Balancing Rules](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-rules)
- [Inbound NAT Rules](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-inbound-nat-rules)
- [Application Gateway](https://learn.microsoft.com/en-us/azure/application-gateway/)

## Notes for Instructors

- Standard SKU is recommended for production
- Health probes should match actual service health endpoints
- Use session persistence only when necessary
- Monitor backend health regularly
- Test failover by stopping one backend VM
- Document load balancer configuration
- Consider Application Gateway for layer 7 (application layer) features
- Internal load balancers can be used for internal traffic

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
