# Lab 01: Implement Azure Monitor

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure Azure Monitor workspaces
- [ ] Configure metrics and alerts
- [ ] Create and manage action groups
- [ ] Implement custom metrics
- [ ] Monitor Azure resources

## Prerequisites

- Resource group: RG-Network or RG-Compute
- Virtual machines or other Azure resources
- Azure Portal or PowerShell/CLI
- Email account for alert notifications
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation needs comprehensive monitoring of their Azure infrastructure. You'll implement Azure Monitor to track performance, create alerts for critical conditions, and set up notifications.

## Architecture

```
Azure Resources (VMs, Storage, etc.)
     |
     v
Azure Monitor
├── Metrics
│   ├── CPU %
│   ├── Memory
│   ├── Disk I/O
│   └── Network
├── Logs
│   ├── Resource logs
│   ├── Platform logs
│   └── Custom logs
├── Alerts
│   ├── Metric-based
│   ├── Log-based
│   └── Activity Log
└── Action Groups
    ├── Email notifications
    ├── SMS alerts
    ├── Webhooks
    └── Runbooks
```

## Task 1: Create and Configure Workspaces

### Instructions

1. Create Log Analytics Workspace:
   - Navigate to **Log Analytics workspaces** in Azure Portal
   - Click **+ Create**
   - **Subscription**: Your subscription
   - **Resource group**: RG-Network
   - **Name**: law-contoso
   - **Region**: East US
   - Click **Create**

2. Create Application Insights:
   - Navigate to **Application Insights**
   - Click **+ Create**
   - **Name**: appinsights-contoso
   - **Resource type**: ASP.NET web app
   - **Log Analytics Workspace**: law-contoso
   - Click **Create**

3. Enable diagnostics on resources:
   - Go to a VM or storage account
   - Go to **Diagnostics settings**
   - Click **+ Add diagnostic setting**
   - **Name**: send-to-law
   - **Destination details**: Send to Log Analytics Workspace
   - Select **law-contoso**
   - Select logs to collect (Activity log, Resource logs)
   - Click **Save**

### Verification

- [ ] Log Analytics workspace is created
- [ ] Application Insights is created
- [ ] Diagnostics are enabled on resources
- [ ] Data is flowing into workspace
- [ ] Can see resources in workspace

### PowerShell Example

```powershell
# Create Log Analytics workspace
$law = New-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Network" `
    -Name "law-contoso" `
    -Location "eastus"

# Create Application Insights
$ai = New-AzApplicationInsights -ResourceGroupName "RG-Network" `
    -Name "appinsights-contoso" `
    -Location "eastus" `
    -Kind "web" `
    -WorkspaceResourceId $law.ResourceId

# Enable diagnostics on VM
$vm = Get-AzVM -Name "VM-Windows" -ResourceGroupName "RG-Compute"
$diag = New-AzDiagnosticSetting -Name "send-to-law" `
    -ResourceId $vm.Id `
    -WorkspaceId $law.ResourceId `
    -Metric $true `
    -Log $true
```

## Task 2: Configure Metrics and Create Dashboards

### Instructions

1. Navigate to **Azure Monitor** > **Metrics**
2. Create metric query:
   - **Scope**: Select a resource (e.g., VM-Windows)
   - **Metric**: CPU Percentage
   - **Aggregation**: Average
   - **Time range**: Last 24 hours
   - **Granularity**: 1 minute

3. Create additional metric queries:
   - Available Memory Bytes
   - Network In/Out
   - Disk Read/Write

4. Create custom dashboard:
   - Go to **Dashboards**
   - Click **+ New dashboard**
   - **Name**: Contoso-Monitoring
   - **Sharing**: Private
   - Click **Create**

5. Pin metrics to dashboard:
   - From metric query, click **Pin to dashboard**
   - Select **Contoso-Monitoring**
   - Repeat for multiple metrics
   - Arrange tiles as needed

6. Add markdown tiles:
   - Click **+ Edit**
   - Click **+ Add** > **Markdown**
   - Add documentation about metrics

### Verification

- [ ] Metrics are displayed correctly
- [ ] Dashboard is created
- [ ] Metrics are pinned to dashboard
- [ ] Dashboard is accessible
- [ ] Metrics show current data

### PowerShell Example

```powershell
# Get metric data
$metric = Get-AzMetric -ResourceId $vm.Id `
    -MetricName "Percentage CPU" `
    -TimeGrain "PT1M" `
    -StartTime (Get-Date).AddHours(-1) `
    -EndTime (Get-Date)

# Display metric values
$metric.Data | Format-Table -Property TimeStamp, Average
```

## Task 3: Create Action Groups

### Instructions

1. Navigate to **Azure Monitor** > **Alerts** > **Action groups**
2. Click **+ Create action group**
3. Configure action group:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Network
   - **Name**: contoso-actions
   - **Display name**: Contoso Alerts
   - Click **Next: Notifications**

4. Add notifications:
   - Click **+ Add notification**
   - **Notification type**: Email/SMS Azure Resource Manager Role
   - **Azure Resource Manager Role**: Owner
   - Click **OK**

5. Add email notification:
   - Click **+ Add notification**
   - **Notification type**: Email
   - **Email address**: your-email@example.com
   - **Name**: Email Alert
   - Click **OK**

6. Add action (optional):
   - Click **+ Add action**
   - **Action type**: Automation Runbook
   - Select a runbook
   - Click **OK**

7. Review and create

### Verification

- [ ] Action group is created
- [ ] Notifications are configured
- [ ] Email address is added
- [ ] Multiple notification types work
- [ ] Actions can be triggered manually to test

### PowerShell Example

```powershell
# Create action group
$actionGroup = New-AzActionGroup -ResourceGroupName "RG-Network" `
    -Name "contoso-actions" `
    -ShortName "ContAlert"

# Add email receiver
Add-AzActionGroupReceiver -ActionGroup $actionGroup `
    -Name "Email" `
    -EmailReceiver `
    -EmailAddress "admin@contoso.com"

# Save action group
Set-AzActionGroup -ActionGroup $actionGroup
```

## Task 4: Create and Configure Alerts

### Instructions

1. Create metric alert:
   - Navigate to **Azure Monitor** > **Alerts** > **New alert rule**
   - **Scope**: Select resource (VM-Windows)
   - **Condition**: 
     - **Signal name**: Percentage CPU
     - **Operator**: Greater than
     - **Threshold**: 80
     - **Aggregation type**: Average
     - **Evaluation**: Every 5 minutes

2. Configure alert:
   - **Severity**: 2 (Warning)
   - **Alert rule name**: HighCPUAlert
   - **Description**: Alert when CPU exceeds 80%
   - Click **Next**

3. Add action:
   - **Action group**: contoso-actions
   - Click **Create**

4. Create activity log alert:
   - Go to **Alerts** > **New alert rule**
   - **Signal type**: Activity log
   - **Event category**: Administrative
   - **Operation**: Create Virtual Machine
   - Add action group
   - Click **Create**

5. Create log-based alert:
   - Go to **Log Analytics workspace**
   - **Logs** > Create new query:
     ```kusto
     Perf
     | where ObjectName == "Processor"
     | where CounterName == "% Processor Time"
     | where CounterValue > 80
     | summarize by Computer
     ```
   - Click **New alert rule**
   - Configure threshold and action

### Verification

- [ ] Metric alert is created
- [ ] Activity log alert is created
- [ ] Log-based alert is created
- [ ] Alerts show in Alerts list
- [ ] Can trigger alert manually (if available)

### PowerShell Example

```powershell
# Create metric alert
$criteria = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" `
    -TimeAggregationOperator Average `
    -Operator GreaterThan `
    -Threshold 80

$alert = Add-AzMetricAlertRuleV2 -Name "HighCPUAlert" `
    -ResourceGroupName "RG-Compute" `
    -WindowSize "00:05:00" `
    -Frequency "00:01:00" `
    -TargetResourceScope $vm.Id `
    -Criteria $criteria `
    -ActionGroup $actionGroup.Id `
    -Severity 2
```

## Task 5: Monitor and Analyze Data

### Instructions

1. Write KQL queries in Log Analytics:
   - Open **Log Analytics workspace**
   - Click **Logs**
   - Write query to analyze data:
     ```kusto
     Event
     | where EventLevelName == "Error"
     | summarize count() by Source
     | render barchart
     ```

2. Create saved queries:
   - Click **Save** after running query
   - **Save as**: Saved query
   - **Name**: Error Count by Source
   - **Category**: Performance

3. Monitor performance trends:
   - Query to show CPU trends:
     ```kusto
     Perf
     | where ObjectName == "Processor"
     | where CounterName == "% Processor Time"
     | summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
     | render timechart
     ```

4. Create workbooks:
   - Go to **Workbooks** > **New**
   - Add text, metrics, and queries
   - Create custom visualization
   - Save workbook

### Verification

- [ ] KQL queries execute successfully
- [ ] Queries return meaningful data
- [ ] Saved queries are accessible
- [ ] Performance trends are visible
- [ ] Workbooks display data correctly

## Cleanup

Remove monitoring resources:

```powershell
# Remove alerts
Get-AzMetricAlertRuleV2 -ResourceGroupName "RG-Compute" | Remove-AzMetricAlertRuleV2

# Remove action group
Remove-AzActionGroup -ResourceGroupName "RG-Network" -Name "contoso-actions" -Force

# Remove Application Insights
Remove-AzApplicationInsights -ResourceGroupName "RG-Network" -Name "appinsights-contoso" -Force

# Remove Log Analytics workspace
Remove-AzOperationalInsightsWorkspace -ResourceGroupName "RG-Network" -Name "law-contoso" -Force
```

## Review

In this lab, you:
- Created Log Analytics and Application Insights workspaces
- Configured metrics and dashboards
- Created action groups and notifications
- Implemented metric, activity log, and log-based alerts
- Analyzed monitoring data with KQL

Comprehensive monitoring is essential for maintaining healthy Azure infrastructure.

## Additional Resources

- [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/)
- [Log Analytics](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
- [Metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-supported)
- [Alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [KQL Queries](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)

## Notes for Instructors

- Set appropriate alert thresholds to avoid alert fatigue
- Test alerts and action groups regularly
- Use meaningful names for alerts and action groups
- Document alert rules and their purposes
- Review and adjust thresholds based on actual workloads
- Use log-based alerts for complex conditions
- Monitor costs of data ingestion
- Implement retention policies to manage storage

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
