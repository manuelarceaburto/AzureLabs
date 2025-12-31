# Lab 03: Implement Log Analytics and Diagnostics

## Objectives

By completing this lab, you will be able to:
- [ ] Configure Log Analytics for data collection
- [ ] Enable diagnostics on Azure resources
- [ ] Write and execute KQL queries
- [ ] Create alerts from logs
- [ ] Implement data retention policies

## Prerequisites

- Log Analytics workspace from Lab 01
- Azure resources (VMs, storage, app services)
- Azure Portal or PowerShell/CLI
- Basic understanding of log analysis
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation needs detailed logging and analysis of their Azure infrastructure for security, performance, and compliance. You'll implement comprehensive logging with Log Analytics.

## Architecture

```
Azure Resources
├── VMs → Guest Agent → Log Analytics
├── Storage → Diagnostics → Log Analytics
├── Network → Flow Logs → Log Analytics
├── App Service → Application Logs → Log Analytics
└── Azure AD → Sign-in Logs → Log Analytics
     |
     v
Log Analytics Workspace
├── Tables (Data)
│   ├── Perf (Performance)
│   ├── Event (Windows Events)
│   ├── Syslog (Linux logs)
│   ├── SecurityEvent (Security)
│   └── Custom tables
├── Queries (KQL)
├── Alerts
└── Reports
```

## Task 1: Configure Log Analytics Workspace

### Instructions

1. Review workspace from previous lab (law-contoso)

2. Enable data collection:
   - Go to workspace > **Agents configuration**
   - Review connection settings
   - Note workspace ID and primary key (for manual agent installation)

3. Configure data sources:
   - Go to **Data sources**
   - Click **+ Add**
   - **Data source type**: Windows Event Logs
   - **Event types to collect**: Error, Warning
   - Click **Add**

4. Add Linux syslog:
   - Click **+ Add**
   - **Data source type**: Syslog
   - **Facilities**: auth, authpriv, syslog
   - **Severities**: err, crit, alert, emerg
   - Click **Add**

5. Configure performance counters:
   - Click **+ Add**
   - **Data source type**: Windows Performance Counters
   - **Add counters**:
     - Processor: % Processor Time
     - Memory: % Committed Bytes in Use
     - PhysicalDisk: % Disk Time
   - Click **Add**

### Verification

- [ ] Data sources are configured
- [ ] Windows Event Logs are enabled
- [ ] Linux Syslog is enabled
- [ ] Performance counters are configured
- [ ] Can see workspace settings

### PowerShell Example

```powershell
# Get workspace
$workspace = Get-AzOperationalInsightsWorkspace -Name "law-contoso" `
    -ResourceGroupName "RG-Network"

# Create data source
$dataSource = New-AzOperationalInsightsWindowsEventDataSource -ResourceGroupName "RG-Network" `
    -WorkspaceName $workspace.Name `
    -Name "WindowsEvents" `
    -EventLogName "System" `
    -CollectErrors `
    -CollectWarnings

# View data sources
Get-AzOperationalInsightsDataSource -ResourceGroupName "RG-Network" `
    -WorkspaceName $workspace.Name
```

## Task 2: Enable Diagnostics on Resources

### Instructions

1. Enable diagnostics on VM:
   - Navigate to VM > **Diagnostics settings**
   - Click **+ Add diagnostic setting**
   - **Name**: vm-diagnostics
   - **Logs**: Select all available logs
     - Activity log
     - Resource logs (if available)
   - **Destination details**: Send to Log Analytics Workspace
   - **Log Analytics Workspace**: law-contoso
   - Click **Save**

2. Enable diagnostics on storage account:
   - Storage account > **Diagnostics settings**
   - Click **+ Add diagnostic setting**
   - **Name**: storage-diagnostics
   - Enable logging for:
     - Blob storage read/write/delete
   - Select **law-contoso** workspace
   - Click **Save**

3. Enable diagnostics on app service (if available):
   - App Service > **Diagnostics settings**
   - Enable:
     - Application Logging
     - Web Server Logging
     - Detailed Error Messages
     - Failed Request Tracing
   - Configure log retention (30 days)

4. Enable network flow logs (optional):
   - Network Watcher > **NSG flow logs**
   - Enable for NSGs
   - Configure storage account
   - Enable traffic analytics with Log Analytics

### Verification

- [ ] Diagnostics are enabled on all resources
- [ ] Logs are flowing to Log Analytics
- [ ] Data appears in workspace after 5-10 minutes
- [ ] Can see data in table preview

### Azure CLI Example

```bash
# Enable diagnostics on VM
az monitor diagnostic-settings create \
    --name vm-diagnostics \
    --resource /subscriptions/{sub-id}/resourceGroups/RG-Compute/providers/Microsoft.Compute/virtualMachines/VM-Windows \
    --logs '[{"category": "ActivityLog", "enabled": true}]' \
    --workspace /subscriptions/{sub-id}/resourceGroups/RG-Network/providers/Microsoft.OperationalInsights/workspaces/law-contoso
```

## Task 3: Write KQL Queries

### Instructions

1. Open Log Analytics workspace > **Logs**
2. Write query to check activity:
   ```kusto
   // Get all operations in last hour
   AzureActivity
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, Caller, OperationName, ResourceType
   | sort by TimeGenerated desc
   ```

3. Write performance query:
   ```kusto
   // Get average CPU by computer for last 24 hours
   Perf
   | where ObjectName == "Processor" 
     and CounterName == "% Processor Time"
     and TimeGenerated > ago(24h)
   | summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
   | render timechart
   ```

4. Write security query:
   ```kusto
   // Find failed logons in last 24 hours
   SecurityEvent
   | where TimeGenerated > ago(24h)
     and EventID == 4625  // Failed logon
   | summarize FailureCount = count() by Account, Computer
   | sort by FailureCount desc
   ```

5. Write custom query:
   ```kusto
   // Get error events with details
   Event
   | where EventLevelName == "Error"
   | where TimeGenerated > ago(7d)
   | project TimeGenerated, Computer, Source, EventID, RenderedDescription
   | top 100 by TimeGenerated desc
   ```

6. Save queries:
   - Click **Save**
   - **Save as**: Saved query
   - **Name**: Performance Summary
   - **Category**: Performance
   - Click **Save**

### Verification

- [ ] Queries execute successfully
- [ ] Results display correctly
- [ ] Can filter and aggregate data
- [ ] Visualizations render properly
- [ ] Saved queries are accessible

## Task 4: Create Log-Based Alerts

### Instructions

1. Create alert from query:
   - Write KQL query for alert condition
   - Click **+ New alert rule**
   - **Alert rule name**: HighCPUFromLogs
   - **Severity**: 2 (Warning)
   - **Frequency**: Every 5 minutes
   - **Evaluation period**: Last 5 minutes
   - **Threshold**: CPU average > 80

2. Create security alert:
   ```kusto
   SecurityEvent
   | where EventID == 4625  // Failed logon
   | summarize count() by Account
   | where count_ > 5  // More than 5 failed attempts
   ```
   - Alert rule name: FailedLogonAlert
   - Severity: 1 (Critical)

3. Create compliance alert:
   ```kusto
   AuditLogs
   | where OperationName == "Delete Role Definition"
   | summarize count() by Actor
   ```
   - Track role definition deletions

4. Add action group:
   - Select **contoso-actions** action group
   - This will trigger email/SMS notifications
   - Click **Create alert rule**

### Verification

- [ ] Log-based alerts are created
- [ ] Alerts have appropriate thresholds
- [ ] Action groups are assigned
- [ ] Alerts will trigger based on conditions
- [ ] Can test alert notifications

## Task 5: Implement Retention and Management

### Instructions

1. Set data retention policy:
   - Go to workspace > **Usage and estimated costs**
   - Click **Data retention**
   - **Retention period**: 30 days (default) to 730 days
   - Note: Longer retention increases costs

2. Configure table retention:
   - Go to **Tables** section
   - Review different table retention options
   - Set custom retention for critical logs (90 days)
   - Set shorter retention for verbose logs (7 days)

3. Implement data archival:
   - Enable **Archive** for data retention >30 days
   - This moves older data to cheaper storage
   - Costs less but longer query times

4. Monitor workspace growth:
   - **Usage and estimated costs** > **Workspace usage by data type**
   - Identify high-volume data sources
   - Consider filtering unnecessary logs

5. Implement search capacity reservation:
   - Go to **Pricing tier**
   - Consider **Capacity Reservation** if consistent usage
   - Can be 100 GB - 10,000 GB per day
   - Provides cost savings vs. pay-as-you-go

### Verification

- [ ] Retention policies are configured
- [ ] Data archival is enabled
- [ ] Can see workspace growth trends
- [ ] Understand cost factors
- [ ] Capacity is appropriate for needs

## Cleanup

Remove diagnostics settings:

```bash
# Remove diagnostics setting
az monitor diagnostic-settings delete \
    --name vm-diagnostics \
    --resource /subscriptions/{sub-id}/resourceGroups/RG-Compute/providers/Microsoft.Compute/virtualMachines/VM-Windows

# Clean up saved queries
# Done through Portal - delete each saved query
```

## Review

In this lab, you:
- Configured Log Analytics workspace and data sources
- Enabled diagnostics across Azure resources
- Wrote complex KQL queries for analysis
- Created log-based alerts for monitoring
- Implemented retention and cost management

Log Analytics provides comprehensive insights into Azure infrastructure.

## Additional Resources

- [Log Analytics](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
- [KQL Documentation](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Data Sources](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources)
- [Diagnostics Settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)
- [Query Examples](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/query-language)

## Notes for Instructors

- Start with simple queries before complex ones
- Use sample queries from Microsoft docs
- Monitor workspace size and costs regularly
- Set appropriate retention based on compliance needs
- Use archive tier for cost optimization
- Test alerts thoroughly before deployment
- Document common queries for team use
- Consider advanced analytics for pattern detection
- Implement Role-Based Access Control (RBAC) on workspace

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
