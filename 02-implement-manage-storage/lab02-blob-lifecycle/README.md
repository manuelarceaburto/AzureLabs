# Lab 02: Implement Azure Blob Storage Lifecycle Management

## Objectives

By completing this lab, you will be able to:
- [ ] Understand blob lifecycle management policies
- [ ] Create and apply lifecycle management rules
- [ ] Implement tiering strategies (Hot, Cool, Archive)
- [ ] Automate blob deletion based on age
- [ ] Monitor lifecycle policy execution

## Prerequisites

- Completed Lab 01 or existing storage account with blob containers
- Azure Portal or Azure PowerShell
- Blob storage containers with data
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation stores large amounts of data in Azure Blob Storage. To optimize costs, they need to automatically move older data to cheaper storage tiers and delete data after a retention period. You'll implement a lifecycle management policy.

## Architecture

```
Storage Account (Blob Storage)
├── Hot Tier (Recent data - expensive, frequent access)
│   └── documents (0-30 days)
├── Cool Tier (Infrequent access - medium cost)
│   └── documents (31-90 days)
├── Archive Tier (Rare access - very cheap, requires rehydration)
│   └── documents (91+ days)
└── Deleted (After 365 days)
```

## Task 1: Understand Storage Tiers

### Instructions

1. Review storage access tiers:
   - **Hot**: High availability, high access costs, low transaction costs
   - **Cool**: Lower availability, lower access costs, higher transaction costs
   - **Archive**: Lowest cost, very low availability, requires rehydration (3-15 hours)

2. Navigate to storage account from Lab 01
3. Go to **Overview** tab
4. Check current access tier setting
5. Review access tier costs in pricing documentation

### Cost Analysis

Create a cost comparison:
- 100 GB in Hot tier: ~$2/month (storage) + access costs
- 100 GB in Cool tier: ~$1/month (storage) + access costs
- 100 GB in Archive tier: ~$0.50/month (storage) + rehydration costs

## Task 2: Create Lifecycle Management Policy

### Instructions

1. Navigate to storage account **contosodata** from Lab 01
2. Go to **Data management** > **Lifecycle management**
3. Click **Add a rule**
4. Configure Rule 1 - Move to Cool:
   - **Rule name**: MoveToCoolfter30Days
   - **Rule scope**: Apply to all blobs
   - **Blob type filters**: Block blobs, Page blobs
   - **More options**:
     - Base blob last modified: Yes
     - Days ago: 30
     - Action: Move to cool storage
   - Click **Add**

5. Click **Add a rule** again
6. Configure Rule 2 - Move to Archive:
   - **Rule name**: MoveToArchiveAfter90Days
   - **Rule scope**: Apply to all blobs
   - **More options**:
     - Base blob last modified: Yes
     - Days ago: 90
     - Action: Move to archive storage

7. Click **Add a rule** again
8. Configure Rule 3 - Delete:
   - **Rule name**: DeleteAfter365Days
   - **Rule scope**: Apply to all blobs
   - **More options**:
     - Base blob last modified: Yes
     - Days ago: 365
     - Action: Delete the blob

9. Review and save the policy

### Verification

- [ ] All three rules are created
- [ ] Rules are in correct order (Cool → Archive → Delete)
- [ ] Rule conditions are properly configured
- [ ] Policy is saved and enabled
- [ ] Can see policy JSON representation

### PowerShell Example

```powershell
# Create lifecycle management policy
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"

# Define lifecycle rules
$rule1 = New-Object -TypeName "Microsoft.Azure.Storage.Blob.Models.ManagementPolicyBaseBlob"
$rule1.Name = "MoveToCoolfter30Days"
$rule1.Enabled = $true
$rule1.Type = "Lifecycle"
$rule1.Definition = @{
    actions = @{
        baseBlob = @{
            tierToCool = @{
                daysAfterModificationGreaterThan = 30
            }
        }
    }
    filters = @{
        blobTypes = @("blockBlob")
    }
}

# Set management policy
Set-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName $storageAccount.StorageAccountName `
    -Policy $policy
```

## Task 3: Test Lifecycle Policy with Snapshots

### Instructions

1. Since waiting 30+ days is impractical, create test blobs with modified timestamps:
   - Create a test blob in the documents container
   - Download the blob
   - Upload it again with a modified date

2. Monitor lifecycle execution:
   - Go to **Lifecycle management**
   - Click on a rule to see last run time
   - Policy typically runs once per day
   - You can manually trigger a test run

3. Create test blobs for each tier:
   - **Hot tier blob**: Recent (created today)
   - **Cool tier blob**: Simulate 45 days old (using PowerShell)
   - **Archive tier blob**: Simulate 120 days old (using PowerShell)

4. Manually set blob tier (for testing):
   - Go to blob properties
   - Change access tier manually to see policy impact
   - Observe cost changes

### Verification

- [ ] Test blobs are created
- [ ] Access tiers can be manually changed
- [ ] Blob properties show correct tier
- [ ] Lifecycle policy rules are visible
- [ ] Can see rule execution status

### PowerShell Example

```powershell
# Get storage context
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

# Create test blob
$testData = "Test data for lifecycle policy"
$testData | Out-File -FilePath "C:\testblob.txt"
Set-AzStorageBlobContent -File "C:\testblob.txt" `
    -Container "documents" `
    -Blob "testblob.txt" `
    -Context $ctx

# Get blob and check tier
$blob = Get-AzStorageBlob -Container "documents" -Blob "testblob.txt" -Context $ctx
$blob.BlobProperties.AccessTier

# Manually set blob tier (for testing)
Set-AzStorageBlobTier -Container "documents" `
    -Blob "testblob.txt" `
    -Tier "Cool" `
    -Context $ctx

# Verify tier change
Get-AzStorageBlob -Container "documents" -Blob "testblob.txt" -Context $ctx | Select-Object Name, @{N="Tier";E={$_.BlobProperties.AccessTier}}
```

## Task 4: Monitor and Optimize Policy

### Instructions

1. Navigate to **Metrics** in storage account
2. Create metric for:
   - **Capacity by tier**:
     - Hot capacity
     - Cool capacity
     - Archive capacity
   - **Transaction count by tier** (to track access patterns)

3. Set up alerts:
   - Alert when Hot tier capacity increases unexpectedly
   - Alert when Archive tier exceeds 500 GB (costly to rehydrate)

4. Review policy effectiveness:
   - Calculate current spend vs. optimized spend
   - Identify blobs that should have been tiered
   - Adjust lifecycle rules if needed

5. Test archive rehydration (optional):
   - Download a blob from archive tier
   - Observe rehydration process (3-15 hours)
   - Note additional cost

### Verification

- [ ] Metrics show capacity by tier
- [ ] Alerts are configured
- [ ] Policy rules are executing correctly
- [ ] Data is moving to appropriate tiers
- [ ] Cost optimization is visible in metrics

### PowerShell Example

```powershell
# Get blob tier distribution
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

Get-AzStorageBlob -Container "documents" -Context $ctx | 
    Group-Object -Property { $_.BlobProperties.AccessTier } | 
    Select-Object Name, Count, @{N="TotalSize";E={($_.Group | Measure-Object -Property Length -Sum).Sum / 1GB}}

# View lifecycle management policy
Get-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName $storageAccount.StorageAccountName | 
    ConvertTo-Json -Depth 10

# Rehydrate archive blob
Set-AzStorageBlobTier -Container "documents" `
    -Blob "archiveblob.txt" `
    -Tier "Hot" `
    -Context $ctx
```

## Task 5: Advanced Policy Configuration

### Instructions

1. Add blob version filtering (optional):
   - Go to **Lifecycle management** > **Add a rule**
   - Filter by blob version (previous versions only)
   - Delete previous versions after 30 days

2. Add snapshot filtering:
   - Filter by blob snapshots
   - Delete snapshots after 60 days

3. Add deleted blob filtering:
   - Permanently delete soft-deleted blobs after 7 days
   - (Soft delete must be enabled first)

4. Fine-tune existing rules:
   - Adjust days based on business requirements
   - Consider compliance retention periods
   - Balance cost vs. accessibility

### Verification

- [ ] All filters are correctly applied
- [ ] Policy handles versions and snapshots
- [ ] Retention periods meet compliance requirements
- [ ] Policy is optimized for cost and performance

## Cleanup

Remove test blobs and adjust lifecycle policy:

```powershell
# Remove test blobs
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

Remove-AzStorageBlob -Container "documents" -Blob "testblob.txt" -Context $ctx

# Disable lifecycle policy (if needed)
Remove-AzStorageAccountManagementPolicy -ResourceGroupName "RG-Storage" `
    -StorageAccountName $storageAccount.StorageAccountName
```

## Review

In this lab, you:
- Understood Azure storage access tiers and costs
- Created lifecycle management policies
- Configured rules for tiering and deletion
- Monitored policy execution
- Optimized storage costs through automation

Lifecycle management is critical for cost optimization in large-scale storage deployments.

## Additional Resources

- [Blob Lifecycle Management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)
- [Access Tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)
- [Cost Optimization](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-lifecycle-management-concepts)
- [Archive Tier Rehydration](https://learn.microsoft.com/en-us/azure/storage/blobs/archive-rehydrate-overview)
- [Pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

## Notes for Instructors

- Lifecycle policies run approximately once per day
- Changes may not be visible immediately
- Archive tier requires 3-15 hours for rehydration
- Always plan retention periods based on compliance requirements
- Test policies with a few blobs before applying broadly
- Monitor costs closely during optimization
- Soft delete and versioning add costs but provide recovery options

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
