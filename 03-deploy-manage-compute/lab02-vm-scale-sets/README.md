# Lab 02: Implement Virtual Machine Scale Sets

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure Virtual Machine Scale Sets
- [ ] Implement auto-scaling rules
- [ ] Configure load balancing
- [ ] Deploy applications to scale sets
- [ ] Monitor and manage scale sets

## Prerequisites

- Completed Lab 01 or existing Resource Group (RG-Compute)
- Azure Portal or PowerShell/CLI
- Basic understanding of load balancing
- VM image or custom image
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation has a web application that experiences varying traffic levels. They need to automatically scale the number of instances up or down based on demand while distributing traffic across all instances.

## Architecture

```
Resource Group: RG-Compute
├── Virtual Machine Scale Set: VMSS-Web
│   ├── Instance 1 (VM)
│   ├── Instance 2 (VM)
│   ├── Instance 3 (VM) [scales up/down]
│   └── Instance 4 (VM) [scales up/down]
├── Load Balancer: lb-web
│   ├── Frontend IP: public
│   ├── Backend Pool: VMSS instances
│   └── Health Probe: HTTP /health
├── Network Security Group: NSG-VMSS
└── Application Gateway (optional)
```

## Task 1: Create Virtual Machine Scale Set

### Instructions

1. Navigate to **Virtual Machine Scale Sets** in Azure Portal
2. Click **+ Create**
3. Configure basics:
   - **Resource group**: RG-Compute
   - **Name**: VMSS-Web
   - **Region**: East US
   - **Image**: Windows Server 2022 Datacenter (or Ubuntu)
   - **Size**: Standard_B2s
   - **Administrator account**: azureuser / password
   - **Licensing**: Include license (Windows)

4. Configure scaling:
   - **Initial instance count**: 2
   - Click **Next: Disks**

5. Configure disks:
   - **OS disk type**: Premium SSD
   - **Use managed disks**: Yes
   - Click **Next: Networking**

6. Configure networking:
   - **Virtual network**: vnet-prod (from Lab 01)
   - **Subnet**: subnet-01
   - **Public IP**: Create new
   - **Network security group**: NSG-VM (existing)
   - Click **Next: Management**

7. Configure management:
   - **Enable automatic repair policy**: Yes
   - **Enable automatic OS upgrade**: No
   - Click **Next: Health**

8. Configure health:
   - **Enable application health monitoring**: Yes
   - **Grace period**: 300 seconds
   - Click **Review + create**

9. Create scale set

### Verification

- [ ] Scale set is created successfully
- [ ] Initial instances are deployed
- [ ] Load balancer is created automatically
- [ ] Public IP is accessible
- [ ] Can SSH/RDP to instances

### PowerShell Example

```powershell
# Create scale set
$vmss = New-AzVmss -ResourceGroupName "RG-Compute" `
    -VMScaleSetName "VMSS-Web" `
    -ImageName "Win2022Datacenter" `
    -InstanceCount 2 `
    -VirtualNetworkName "vnet-prod" `
    -SubnetName "subnet-01" `
    -PublicIpAddressName "pip-vmss" `
    -LoadBalancerName "lb-vmss" `
    -UpgradePolicyMode "Automatic" `
    -Credential (Get-Credential)

# Get scale set details
Get-AzVmss -ResourceGroupName "RG-Compute" -VMScaleSetName "VMSS-Web"
```

## Task 2: Deploy Application to Scale Set

### Instructions

1. Create deployment script:
   - For Windows: PowerShell script
   - For Linux: Bash script

2. Example Windows deployment script:
   ```powershell
   # Install IIS
   Install-WindowsFeature -name Web-Server -IncludeManagementTools
   
   # Create health check endpoint
   $html = @"
   <html>
   <body>
   <h1>Health Check OK</h1>
   </body>
   </html>
   "@
   
   $html | Out-File -FilePath "C:\inetpub\wwwroot\health.html"
   ```

3. Add script to scale set:
   - Go to **VMSS-Web** > **Extensions**
   - Click **+ Add extension**
   - Select **Custom Script Extension** (Windows) or **Custom Script for Linux**
   - Upload or paste script
   - Click **Create**

4. Verify deployment:
   - Navigate to scale set instances
   - Check each instance health status
   - Verify application is running

### Verification

- [ ] Custom Script Extension is installed
- [ ] Script executes on all instances
- [ ] Application is running on all instances
- [ ] Health check endpoint responds
- [ ] Load balancer routes traffic to all instances

### PowerShell Example

```powershell
# Create custom script extension
$settings = @{
    "fileUris" = @("https://storageaccount.blob.core.windows.net/scripts/deploy.ps1")
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File deploy.ps1"
}

Add-AzVmssExtension -VirtualMachineScaleSetName "VMSS-Web" `
    -ResourceGroupName "RG-Compute" `
    -Name "CustomScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -Setting $settings
```

## Task 3: Configure Auto-Scaling Rules

### Instructions

1. Navigate to **VMSS-Web** > **Scaling**
2. Click **Enable autoscale**
3. Configure autoscale settings:
   - **Autoscale setting name**: VMSS-Web-scale
   - **Resource group**: RG-Compute
   - **Minimum instance count**: 2
   - **Maximum instance count**: 4
   - **Default instance count**: 2

4. Create scaling rule (Scale Out):
   - Click **+ Add a rule**
   - **Metric name**: CPU Percentage
   - **Metric statistic**: Average
   - **Operator**: Greater than
   - **Metric threshold**: 70%
   - **Duration**: 5 minutes
   - **Operation**: Increase count by 1
   - **Cool down**: 5 minutes

5. Create scaling rule (Scale In):
   - Click **+ Add a rule**
   - **Metric name**: CPU Percentage
   - **Operator**: Less than
   - **Metric threshold**: 30%
   - **Duration**: 5 minutes
   - **Operation**: Decrease count by 1
   - **Cool down**: 5 minutes

6. Click **Save**

### Verification

- [ ] Autoscale is enabled
- [ ] Scaling rules are configured
- [ ] Instance count respects min/max limits
- [ ] Can see scaling history

### PowerShell Example

```powershell
# Get autoscale settings
Get-AzAutoscaleSetting -ResourceGroupName "RG-Compute" -Name "VMSS-Web-scale"

# Create scale out rule
$scaleRule = New-AzAutoscaleRule -MetricName "Percentage CPU" `
    -MetricResourceId "/subscriptions/xxx/resourceGroups/RG-Compute/providers/Microsoft.Compute/virtualMachineScaleSets/VMSS-Web" `
    -Operator "GreaterThan" `
    -MetricStatistic "Average" `
    -Threshold 70 `
    -TimeGrain 00:01:00 `
    -TimeWindow 00:05:00 `
    -ScaleActionCooldown 00:05:00 `
    -ScaleActionDirection "Increase" `
    -ScaleActionValue 1
```

## Task 4: Configure Load Balancer

### Instructions

1. Navigate to load balancer created with scale set
2. Configure health probe:
   - Go to **Health probes**
   - Click **+ Add**
   - **Name**: health-check
   - **Protocol**: HTTP
   - **Port**: 80
   - **Path**: /health.html
   - **Interval**: 15 seconds
   - **Unhealthy threshold**: 2

3. Configure load balancing rule:
   - Go to **Load balancing rules**
   - Click **+ Add**
   - **Name**: http-rule
   - **Frontend IP**: Select public IP
   - **Frontend port**: 80
   - **Backend pool**: VMSS instances
   - **Health probe**: health-check
   - **Session persistence**: None

4. Test load balancing:
   - Get public IP of load balancer
   - Access web application
   - Refresh multiple times
   - Verify requests go to different instances

### Verification

- [ ] Health probe is configured
- [ ] Load balancing rule is active
- [ ] All healthy instances receive traffic
- [ ] Unhealthy instances are removed
- [ ] Can access application via public IP

## Task 5: Monitor Scale Set

### Instructions

1. Navigate to **VMSS-Web** > **Metrics**
2. Create monitoring charts:
   - **CPU Percentage** (across all instances)
   - **Network In/Out**
   - **Disk Read/Write**
   - **Instance count** (to see scaling activity)

3. Set up alerts:
   - Go to **Alerts**
   - **Condition**: CPU > 80%
   - **Action**: Send email

4. Monitor scaling activity:
   - Go to **Instances** tab
   - View instance list and status
   - Watch for scaling events
   - Check power state

5. Review autoscale history:
   - Go to **Scaling** > **Run history**
   - See when scaling actions occurred
   - Verify rules are working

### Verification

- [ ] Metrics dashboard shows data
- [ ] Can see all instances and their status
- [ ] Alerts are triggered appropriately
- [ ] Scaling history is available
- [ ] Performance data is collected

## Cleanup

Remove scale set and resources:

```powershell
# Remove scale set
Remove-AzVmss -ResourceGroupName "RG-Compute" -VMScaleSetName "VMSS-Web" -Force

# Remove load balancer
Get-AzLoadBalancer -ResourceGroupName "RG-Compute" | Remove-AzLoadBalancer -Force

# Remove public IPs
Get-AzPublicIpAddress -ResourceGroupName "RG-Compute" | Remove-AzPublicIpAddress -Force
```

## Review

In this lab, you:
- Created a Virtual Machine Scale Set with multiple instances
- Deployed applications using custom script extensions
- Implemented auto-scaling based on CPU metrics
- Configured load balancing for high availability
- Monitored scale set performance

Scale sets enable automatic scaling for applications with variable demand.

## Additional Resources

- [Virtual Machine Scale Sets](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview)
- [Autoscaling](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
- [Load Balancer](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)
- [Custom Script Extension](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows)

## Notes for Instructors

- Start with small scale sets for testing (2-4 instances)
- Test scaling rules with controlled load
- Monitor costs as scaling can increase expenses
- Use appropriate cooldown periods to prevent flapping
- Ensure health checks are robust
- Test recovery of unhealthy instances
- Document scaling thresholds and reasons
- Consider Application Gateway for more advanced scenarios

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
