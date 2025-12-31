# Lab 01: Create and Manage Storage Accounts

## Objectives

By completing this lab, you will be able to:
- [ ] Create and configure storage accounts
- [ ] Implement storage account access keys and SAS tokens
- [ ] Configure storage account firewalls and virtual networks
- [ ] Enable encryption and security features
- [ ] Monitor storage account performance and usage

## Prerequisites

- Active Azure subscription
- Resource group created (or ability to create one)
- Azure Portal or Azure CLI/PowerShell
- Basic understanding of Azure storage concepts
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation needs to implement a centralized storage solution for application data, backups, and file sharing. You'll create storage accounts with different access patterns, configure security controls, and enable monitoring.

## Architecture

```
Azure Subscription
├── Resource Group: RG-Storage
│   ├── Storage Account: contosodata (General Purpose v2)
│   │   ├── Blob Container: documents
│   │   ├── Blob Container: backups
│   │   ├── File Share: company-files
│   │   └── Table Storage: logs
│   └── Storage Account: contosomedia (Blob Storage)
│       ├── Blob Container: images
│       └── Blob Container: videos
└── Virtual Network (for firewall rules)
```

## Task 1: Create Storage Accounts

### Instructions

1. Navigate to **Storage accounts** in Azure Portal
2. Click **+ Create**
3. Create Storage Account 1 - General Purpose:
   - **Subscription**: Your subscription
   - **Resource Group**: Create new "RG-Storage"
   - **Storage account name**: contosodata[unique-id]
   - **Region**: East US
   - **Performance**: Standard
   - **Redundancy**: Locally-redundant storage (LRS)
   - **Advanced**: Enable public access, keep defaults

4. Create Storage Account 2 - Blob Storage:
   - **Storage account name**: contosomedia[unique-id]
   - **Account kind**: BlobStorage
   - **Access tier**: Hot
   - **Redundancy**: Geo-redundant storage (GRS)

5. Review settings and create both accounts

### Verification

- [ ] Both storage accounts are created successfully
- [ ] Storage accounts appear in Resource Group
- [ ] Access keys are visible
- [ ] Connection string is available
- [ ] Replication status shows appropriate tier

### PowerShell Example

```powershell
# Create resource group
New-AzResourceGroup -Name "RG-Storage" -Location "eastus"

# Create storage account (General Purpose v2)
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "contosodata$(Get-Random)" `
    -Location "eastus" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2"

# Create storage account (Blob)
New-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -Name "contosomedia$(Get-Random)" `
    -Location "eastus" `
    -SkuName "Standard_GRS" `
    -Kind "BlobStorage" `
    -AccessTier "Hot"

# Get storage account details
Get-AzStorageAccount -ResourceGroupName "RG-Storage" | Format-Table
```

## Task 2: Create Containers and Configure Access

### Instructions

1. Navigate to storage account **contosodata**
2. Go to **Containers** under Data storage
3. Create containers:
   - **documents** (access level: Private)
   - **backups** (access level: Private)
   - **logs** (access level: Blob)

4. Upload test files:
   - Create a sample text file locally
   - Upload to the "documents" container
   - Upload same file to "logs" container

5. Test access:
   - Try accessing Private container without credentials (should fail)
   - Try accessing Blob container with direct link (should work)
   - Generate Shared Access Signature (SAS) for documents container
   - Use SAS token to access file

6. Create file share:
   - Navigate to **File shares**
   - Click **+ File share**
   - Name: company-files
   - Quota: 100 GB

### Verification

- [ ] All three blob containers are created
- [ ] Test files are uploaded successfully
- [ ] Access levels are correctly configured
- [ ] SAS token allows temporary access
- [ ] File share is created and accessible

### PowerShell Example

```powershell
# Get storage account context
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

# Create containers
New-AzStorageContainer -Name "documents" -Context $ctx -Permission Off
New-AzStorageContainer -Name "backups" -Context $ctx -Permission Off
New-AzStorageContainer -Name "logs" -Context $ctx -Permission Blob

# Upload blob
Set-AzStorageBlobContent -File "C:\sample.txt" `
    -Container "documents" `
    -Blob "sample.txt" `
    -Context $ctx

# Generate SAS token (valid for 1 hour)
$sasToken = New-AzStorageContainerSASToken -Name "documents" `
    -Permission "racwd" `
    -ExpiryTime (Get-Date).AddHours(1) `
    -Context $ctx

# Create file share
New-AzStorageShare -Name "company-files" -Context $ctx
```

## Task 3: Configure Storage Account Security

### Instructions

1. Navigate to storage account **contosodata**
2. Go to **Security + networking**
3. Configure access keys:
   - View current access keys
   - Generate new key
   - Rotate old key (delete it)
   - Note the new connection string

4. Configure firewall:
   - Go to **Networking** > **Firewalls and virtual networks**
   - Change default action from "Allow" to "Deny"
   - Add your client IP address to allowed list
   - Add virtual network rules (if VNet exists)
   - Test access from your IP (should work)
   - Test from blocked IP (should fail)

5. Enable encryption:
   - Go to **Encryption**
   - Verify that Microsoft-managed keys are enabled
   - Optionally configure customer-managed keys

6. Configure HTTPS enforcement:
   - Go to **Configuration**
   - Enable "Secure transfer required"

### Verification

- [ ] Access keys are rotated
- [ ] Firewall allows only specified IPs
- [ ] Virtual network rules are applied
- [ ] HTTPS is enforced
- [ ] Encryption is enabled
- [ ] Access denied from blocked locations

### PowerShell Example

```powershell
# Rotate storage account keys
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
New-AzStorageAccountKey -ResourceGroupName "RG-Storage" `
    -AccountName $storageAccount.StorageAccountName `
    -KeyName "key1"

# Get updated context with new key
$ctx = (Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage").Context

# Update firewall rules
Update-AzStorageAccountNetworkRuleSet -ResourceGroupName "RG-Storage" `
    -Name $storageAccount.StorageAccountName `
    -DefaultAction Deny

# Add IP to allowed list
Add-AzStorageAccountNetworkRule -ResourceGroupName "RG-Storage" `
    -AccountName $storageAccount.StorageAccountName `
    -IPAddressOrRange "203.0.113.0/24"

# Enable HTTPS only
Set-AzStorageAccount -ResourceGroupName "RG-Storage" `
    -AccountName $storageAccount.StorageAccountName `
    -EnableHttpsTrafficOnly $true
```

## Task 4: Monitor Storage Account Usage

### Instructions

1. Navigate to storage account **contosodata**
2. Go to **Monitoring** > **Metrics**
3. Configure metrics:
   - Metric: Used capacity
   - Time range: Last 24 hours
   - Granularity: 1 hour
   - Create chart

4. Create additional metric views:
   - Ingress (data coming in)
   - Egress (data going out)
   - Transaction count
   - Availability

5. Configure alerts:
   - Go to **Alerts** > **Create alert rule**
   - Metric: Used capacity
   - Condition: Greater than 80% of quota
   - Action: Send email notification
   - Severity: Warning

6. Review storage analytics:
   - Go to **Logs** in Monitoring
   - Query storage access logs
   - Identify top operations

### Verification

- [ ] Metrics dashboard displays usage
- [ ] Charts show ingress/egress trends
- [ ] Alerts are configured
- [ ] Alert notifications work
- [ ] Storage logs are accessible

### PowerShell Example

```powershell
# Get storage account metrics
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"

# Get blob container properties
Get-AzStorageContainer -Context $storageAccount.Context | Format-Table

# Get blob sizes
Get-AzStorageBlob -Container "documents" -Context $storageAccount.Context | Measure-Object -Property Length -Sum

# Configure metrics alert
Add-AzMetricAlertRule -Name "StorageCapacityAlert" `
    -ResourceGroupName "RG-Storage" `
    -ResourceId $storageAccount.Id `
    -MetricName "UsedCapacity" `
    -Operator "GreaterThan" `
    -Threshold 80000000000 `
    -WindowSize "01:00:00"
```

## Cleanup

Remove storage accounts and resources created in this lab:

```powershell
# Remove storage accounts
Remove-AzStorageAccount -ResourceGroupName "RG-Storage" -Name "contosodata*" -Force
Remove-AzStorageAccount -ResourceGroupName "RG-Storage" -Name "contosomedia*" -Force

# Remove resource group
Remove-AzResourceGroup -Name "RG-Storage" -Force
```

Or via Azure CLI:

```bash
# Remove storage accounts
az storage account delete --resource-group "RG-Storage" --name "contosodata*"
az storage account delete --resource-group "RG-Storage" --name "contosomedia*"

# Remove resource group
az group delete --name "RG-Storage" --yes
```

## Review

In this lab, you:
- Created storage accounts with different configurations
- Configured containers and access levels
- Managed access keys and SAS tokens
- Implemented security controls (firewall, HTTPS)
- Monitored storage usage and performance

Proper storage account configuration is essential for data protection and compliance.

## Additional Resources

- [Azure Storage Accounts](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Storage Account Security](https://learn.microsoft.com/en-us/azure/storage/common/security-recommendations)
- [SAS Tokens](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
- [Storage Metrics](https://learn.microsoft.com/en-us/azure/storage/common/storage-analytics-metrics)
- [Firewall and Virtual Networks](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security)

## Notes for Instructors

- Storage account names must be globally unique (lowercase, 3-24 characters)
- Changing redundancy type is not possible after creation
- Access keys should be rotated regularly
- SAS tokens should have minimal required permissions
- Default firewall to Deny is more secure
- Monitor costs carefully as storage can accumulate quickly
- Regional redundancy increases storage costs

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
