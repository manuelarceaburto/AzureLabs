# Lab 03: Implement Azure Files and Azure File Sync

## Objectives

By completing this lab, you will be able to:
- [ ] Create and manage Azure file shares
- [ ] Mount file shares on Windows and Linux
- [ ] Implement Azure File Sync
- [ ] Configure cloud tiering
- [ ] Manage file share snapshots and backups

## Prerequisites

- Storage account from Lab 01 or new storage account
- Windows Server VM or local Windows machine (for File Sync)
- Azure CLI or PowerShell
- Access to on-premises or VM environment
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation has file servers that need to sync with Azure cloud storage. Employees need access to files from on-premises servers and the cloud. You'll implement Azure File Sync to keep on-premises and cloud storage synchronized.

## Architecture

```
Hybrid File Sharing Solution
├── On-Premises Server
│   └── Local Share (C:\SharedFiles)
│       └── Azure File Sync Agent → Sync Group
└── Azure
    ├── Storage Account
    │   ├── File Share: company-files
    │   └── File Share: backups
    └── Azure File Sync Service
        └── Sync Group
            ├── Server Endpoint (On-Prem)
            └── Cloud Endpoint (Azure)
```

## Task 1: Create and Manage Azure File Shares

### Instructions

1. Navigate to storage account **contosodata** from Lab 01
2. Go to **File shares** under **Data storage**
3. Create file shares:
   - **company-files** (1000 GB quota)
   - **backups** (500 GB quota)
   - **projects** (250 GB quota)

4. Create directory structure in file shares:
   - In company-files: /Documents, /Presentations, /Spreadsheets
   - In backups: /Daily, /Weekly, /Monthly
   - In projects: /Active, /Archive

5. Upload sample files:
   - Create 3-5 text files locally
   - Upload to different directories in file shares

6. Configure share permissions:
   - Set file share access level
   - Configure SMB security settings

### Verification

- [ ] All three file shares are created
- [ ] Directory structure is organized
- [ ] Sample files are uploaded
- [ ] File shares have correct quota settings
- [ ] Can browse and access files

### PowerShell Example

```powershell
# Get storage context
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

# Create file shares
New-AzStorageShare -Name "company-files" -Context $ctx -QuotaGiB 1000
New-AzStorageShare -Name "backups" -Context $ctx -QuotaGiB 500
New-AzStorageShare -Name "projects" -Context $ctx -QuotaGiB 250

# Create directories in file share
$share = Get-AzStorageShare -Name "company-files" -Context $ctx
$rootDir = $share.RootDirectoryProperties
$rootDir.GetDirectoryReference("Documents").Create()
$rootDir.GetDirectoryReference("Presentations").Create()
$rootDir.GetDirectoryReference("Spreadsheets").Create()

# Get file share properties
Get-AzStorageShare -Context $ctx | Format-Table
```

## Task 2: Mount File Shares

### Instructions

**For Windows:**

1. Navigate to storage account > File share > **Connect**
2. Copy the mount command for Windows
3. Open PowerShell as Administrator on your machine
4. Run the mount command (example):
   ```
   net use Z: \\contosodata.file.core.windows.net\company-files /user:Azure\contosodata ABC123...
   ```
5. Verify access: `dir Z:`
6. Create/modify files in the mounted share

**For Linux (Optional):**

1. Navigate to storage account > File share > **Connect**
2. Copy the mount command for Linux
3. On Linux machine, install cifs-utils:
   ```
   sudo apt-get install cifs-utils
   ```
4. Mount the share:
   ```
   sudo mount -t cifs //contosodata.file.core.windows.net/company-files /mnt/files -o vers=3.0,username=Azure\contosodata,password=KEY,dir_mode=0777,file_mode=0777
   ```
5. Test access: `ls /mnt/files`

### Verification

- [ ] File share is mounted successfully
- [ ] Can read and write files to mounted share
- [ ] File share appears in File Explorer (Windows)
- [ ] Can navigate directories
- [ ] Performance is acceptable

### PowerShell Example

```powershell
# Get storage account keys
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$storageKey = (Get-AzStorageAccountKey -ResourceGroupName "RG-Storage" -AccountName $storageAccount.StorageAccountName)[0].Value

# Mount file share
$storageAccountName = $storageAccount.StorageAccountName
net use Z: \\$storageAccountName.file.core.windows.net\company-files /user:localhost\$storageAccountName $storageKey

# Verify mount
Get-PSDrive -Name Z

# Create test file
"Test content" | Out-File -FilePath "Z:\test.txt"
```

## Task 3: Implement Azure File Sync

### Instructions

1. Create Azure File Sync Service:
   - Navigate to **Azure File Sync**
   - Click **Create storage sync service**
   - **Resource group**: RG-Storage
   - **Storage Sync Service name**: contoso-sync
   - **Region**: Same as storage account
   - Click **Create**

2. Create Sync Group:
   - Open storage sync service **contoso-sync**
   - Click **+ Sync group**
   - **Sync group name**: company-files-sync
   - **Storage account**: Select contosodata
   - **Azure file share**: company-files
   - Click **Create**

3. Register Server:
   - Go to sync group > **Registered servers**
   - Click **Register server**
   - Download and install Azure File Sync agent
   - Follow installation wizard
   - Sign in with Azure account
   - Register to sync group

4. Create Server Endpoint:
   - Go to sync group > **Server endpoints**
   - Click **Add server endpoint**
   - **Registered server**: Select registered server
   - **Path**: C:\SharedFiles (create this folder first)
   - **Cloud tiering**: Enable (optional, for cost optimization)
   - Click **Create**

### Verification

- [ ] Storage sync service is created
- [ ] Sync group is configured
- [ ] Server is registered successfully
- [ ] Server endpoint is created
- [ ] Sync status shows healthy
- [ ] Files are syncing between on-prem and cloud

### PowerShell Example

```powershell
# Register storage sync service (requires Azure File Sync agent installed)
$storageSyncService = Get-AzStorageSyncService -ResourceGroupName "RG-Storage" -Name "contoso-sync"

# Create sync group
New-AzStorageSyncGroup -ParentObject $storageSyncService `
    -Name "company-files-sync"

# Get sync group
$syncGroup = Get-AzStorageSyncGroup -ParentObject $storageSyncService -Name "company-files-sync"

# Add cloud endpoint
New-AzStorageSyncCloudEndpoint -ParentObject $syncGroup `
    -Name "company-files-endpoint" `
    -AzureFileShareName "company-files"

# Get registered servers
Get-AzStorageSyncServer -ParentObject $storageSyncService
```

## Task 4: Configure Cloud Tiering

### Instructions

1. Open Azure File Sync agent on registered server
2. Go to **Server Endpoints** settings
3. Select server endpoint and click **Configure cloud tiering**
4. Enable cloud tiering:
   - **Free space threshold**: 20% of disk volume
   - **Cache policy**: Intelligent tiering
   - **Date policy**: Files not accessed in 7 days are tiered

5. Monitor tiering:
   - Check disk space savings
   - Verify files are recalled when accessed
   - Monitor sync status

6. Test cloud tiering:
   - Create large file in local folder
   - Wait for sync (should tier if conditions met)
   - Access tiered file (should recall from cloud)
   - Observe recall performance

### Verification

- [ ] Cloud tiering is enabled on server endpoint
- [ ] Policies are configured correctly
- [ ] Local disk space is freed up
- [ ] Files are tiered to cloud appropriately
- [ ] Files are recalled when accessed
- [ ] Sync performance is acceptable

## Task 5: Manage File Share Snapshots

### Instructions

1. Navigate to file share **company-files**
2. Go to **Snapshots** in Azure Portal
3. Click **+ Add snapshot**
4. Review snapshot details:
   - Snapshot time
   - Files included
   - Storage size

5. Restore from snapshot:
   - Right-click file in snapshot
   - Click **Restore**
   - Choose target location
   - Verify file is restored

6. Delete old snapshots:
   - Go to Snapshots list
   - Delete snapshots older than 30 days
   - Verify deletion

### Verification

- [ ] File share snapshots are created
- [ ] Can browse snapshot contents
- [ ] Files can be restored from snapshots
- [ ] Old snapshots are deleted
- [ ] Snapshot storage is managed

### PowerShell Example

```powershell
# Get storage context
$storageAccount = Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage"
$ctx = $storageAccount.Context

# Create file share snapshot
$share = Get-AzStorageShare -Name "company-files" -Context $ctx
$snapshot = $share.CreateSnapshot()

# List snapshots
Get-AzStorageShare -Context $ctx -SnapshotTime | Format-Table

# Get snapshot size
$snapshot.Properties.Quota
```

## Cleanup

Remove Azure File Sync and file shares:

```powershell
# Remove server endpoint
Remove-AzStorageSyncServerEndpoint -InputObject $serverEndpoint

# Remove sync group
Remove-AzStorageSyncGroup -InputObject $syncGroup

# Remove storage sync service
Remove-AzStorageSyncService -InputObject $storageSyncService

# Remove file shares
$ctx = (Get-AzStorageAccount -Name "contosodata*" -ResourceGroupName "RG-Storage").Context
Remove-AzStorageShare -Name "company-files" -Context $ctx
Remove-AzStorageShare -Name "backups" -Context $ctx
Remove-AzStorageShare -Name "projects" -Context $ctx
```

## Review

In this lab, you:
- Created and managed Azure file shares
- Mounted file shares on Windows and Linux
- Implemented Azure File Sync for hybrid scenarios
- Configured cloud tiering for cost optimization
- Managed file share snapshots for data protection

Azure Files and File Sync enable seamless hybrid file sharing between on-premises and cloud.

## Additional Resources

- [Azure Files Overview](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction)
- [Azure File Sync](https://learn.microsoft.com/en-us/azure/storage/file-sync/file-sync-introduction)
- [Cloud Tiering](https://learn.microsoft.com/en-us/azure/storage/file-sync/file-sync-cloud-tiering-overview)
- [Mount File Shares](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-windows)
- [File Share Snapshots](https://learn.microsoft.com/en-us/azure/storage/files/storage-snapshots-files)

## Notes for Instructors

- Azure File Sync agent requires Windows Server 2016 or later
- File share snapshots are read-only and immutable
- Cloud tiering requires Server with at least 2 vCPU and 2 GB RAM
- Monitor bandwidth and network connectivity for syncing
- Test recovery procedures regularly
- Document sync group policies and settings
- Consider backup strategy for production deployments
- File sync can sync to only one cloud endpoint per sync group

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
