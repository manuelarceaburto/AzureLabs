# Lab 02: Implement Backup and Recovery

## Objectives

By completing this lab, you will be able to:
- [ ] Create Recovery Services vaults
- [ ] Configure backup policies
- [ ] Perform backup and restore operations
- [ ] Implement site recovery
- [ ] Monitor and manage backups

## Prerequisites

- Virtual machines from previous labs
- Resource group: RG-Compute or RG-Network
- Storage account for backup storage
- Azure Portal or PowerShell/CLI
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation needs comprehensive backup and disaster recovery strategies. You'll implement backups for VMs, databases, and files, and configure recovery options.

## Architecture

```
Azure Resources
├── VMs
├── Databases
├── File Shares
└── Storage Accounts
     |
     v
Recovery Services Vault
├── Backup Policies
│   ├── Daily backups
│   ├── Weekly full backups
│   └── Monthly retention
├── Backup Items
│   ├── VM-Windows backups
│   ├── VM-Linux backups
│   └── File Share backups
└── Recovery Points
    ├── Snapshots
    ├── Full backups
    └── Incremental backups
```

## Task 1: Create Recovery Services Vault

### Instructions

1. Navigate to **Recovery Services vaults** in Azure Portal
2. Click **+ Create**
3. Configure vault:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Compute
   - **Vault name**: contoso-recovery-vault
   - **Region**: East US (same as resources)
   - Click **Create**

4. Configure vault settings:
   - Go to created vault
   - Navigate to **Properties**
   - Review replication settings
   - Set **Backup Configuration**:
     - **Storage replication type**: Geo-redundant (GRS)
     - This provides disaster recovery capability

5. Enable soft delete:
   - Go to **Backup configuration**
   - **Soft Delete**: Enable
   - **Retention period**: 14 days
   - This allows recovery of accidentally deleted backups

### Verification

- [ ] Recovery Services vault is created
- [ ] Replication type is configured (GRS recommended)
- [ ] Soft delete is enabled
- [ ] Vault is accessible from resource
- [ ] Can see vault properties

### PowerShell Example

```powershell
# Create Recovery Services vault
$vault = New-AzRecoveryServicesVault -Name "contoso-recovery-vault" `
    -ResourceGroupName "RG-Compute" `
    -Location "eastus"

# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $vault

# Configure replication
$backupConfig = Get-AzRecoveryServicesBackupProperties -Vault $vault
Set-AzRecoveryServicesBackupProperties -Vault $vault `
    -BackupStorageRedundancy "GeoRedundant"

# Enable soft delete
Enable-AzRecoveryServicesBackupProtection -Vault $vault
```

## Task 2: Create Backup Policies

### Instructions

1. Navigate to vault > **Backup policies**
2. Create VM backup policy:
   - Click **+ Add policy**
   - **Policy type**: Azure Virtual Machine
   - **Policy name**: DailyBackupPolicy
   - **Backup schedule**: Daily at 2:00 AM
   - **Frequency**: Every day
   - **Retention**:
     - **Daily backups**: Retain for 7 days
     - **Weekly backups**: Sundays, retain for 4 weeks
     - **Monthly backups**: First Sunday, retain for 12 months
     - **Yearly backups**: First Sunday of January, retain for 5 years
   - Click **Create**

3. Create file share backup policy:
   - Click **+ Add policy**
   - **Policy type**: Azure File Share
   - **Policy name**: DailyFileSharePolicy
   - **Backup frequency**: Daily
   - **Backup time**: 3:00 AM
   - **Retention**: 30 days
   - Click **Create**

4. Create SQL database policy (if applicable):
   - **Policy type**: Azure SQL Database in VM
   - **Backup frequency**: Daily (11 PM)
   - **Retention**: Daily for 7 days, Weekly for 4 weeks

### Verification

- [ ] VM backup policy is created
- [ ] File share policy is created
- [ ] Retention settings match requirements
- [ ] Policies are listed in vault
- [ ] Can select policy during enablement

## Task 3: Enable Backups

### Instructions

1. Enable backup for VM-Windows:
   - Navigate to vault > **Backup**
   - Click **+ Backup**
   - **Where is your workload running?**: Azure
   - **What do you want to backup?**: Virtual Machine
   - Click **Backup**
   - Select **VM-Windows**
   - Select **DailyBackupPolicy**
   - Click **Enable Backup**

2. Enable backup for VM-Linux:
   - Repeat steps for VM-Linux

3. Enable backup for file share:
   - **What do you want to backup?**: Azure File Share
   - Select file share from storage account
   - Select **DailyFileSharePolicy**
   - Click **Enable Backup**

4. Trigger initial backup:
   - Go to vault > **Backup items**
   - Select VM backup item
   - Click **Backup now**
   - This creates first recovery point immediately

5. Monitor backup status:
   - Watch **Backup jobs** for completion
   - Should show "Completed" status
   - Time varies based on VM size

### Verification

- [ ] Backups are enabled for VMs
- [ ] Backups are enabled for file shares
- [ ] Initial backup is triggered
- [ ] Backup jobs show completion
- [ ] Recovery points are created

### PowerShell Example

```powershell
# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $vault

# Get backup policy
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DailyBackupPolicy"

# Enable backup for VM
$vm = Get-AzVM -Name "VM-Windows" -ResourceGroupName "RG-Compute"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG-Compute" `
    -Name "VM-Windows" `
    -Policy $policy

# Trigger backup
$container = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" `
    -Status "Registered"
$item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType "AzureVM"
Backup-AzRecoveryServicesBackupItem -Item $item
```

## Task 4: Perform Restore Operations

### Instructions

1. Create test scenario:
   - Delete or modify a file on backup VM (non-critical)
   - Or plan restore for testing

2. Restore file from recovery point:
   - Go to vault > **Backup items**
   - Select **Azure Virtual Machines**
   - Click on VM with backups
   - Click **File Recovery**
   - Select recovery point
   - Download and run script to mount backup
   - Browse files and recover specific files

3. Restore entire VM (if needed):
   - Go to vault > **Backup items**
   - Click **Restore VM**
   - Select recovery point (complete backup)
   - **Restore type**: Create new VM
   - Configure VM settings (name, resource group, storage)
   - Click **Restore**

4. Restore file share:
   - Go to vault > **Backup items**
   - Select **Azure File Shares**
   - Click on file share
   - Click **Restore**
   - Select recovery point
   - Restore to original location or alternate

### Verification

- [ ] Can browse files in recovery points
- [ ] Can restore individual files
- [ ] Can restore entire VM
- [ ] Can restore file shares
- [ ] Restored items match original data

### PowerShell Example

```powershell
# Get recovery point
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item

# Restore VM to new resource
$restoreConfig = New-AzRecoveryServicesBackupResourceGroupConfig -TargetResourceGroupName "RG-Restored"
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp -StorageAccountName "storageaccount" `
    -StorageAccountResourceGroupName "RG-Network"

# Monitor restore job
Get-AzRecoveryServicesBackupJob | Where-Object Operation -eq "Restore"
```

## Task 5: Monitor and Manage Backups

### Instructions

1. Monitor backup jobs:
   - Go to vault > **Backup jobs**
   - View all backup operations
   - Check status (In progress, Completed, Failed)
   - Click on job for details

2. View backup reports:
   - Go to vault > **Backup Reports**
   - **Backup Status** dashboard
   - View protected items, backup success rate
   - Identify failed backups

3. Set alerts for backup failures:
   - Go to vault > **Alerts** > **Alert rules**
   - Click **+ New**
   - **Condition**: Backup job failed
   - **Action**: Send notification
   - Add to action group

4. Configure backup storage:
   - View storage usage in vault
   - Understand replication model
   - Monitor costs
   - Adjust retention if needed

5. Test restore (recommended):
   - Periodically test restore procedures
   - Verify recovery points are usable
   - Document recovery time objectives (RTO)

### Verification

- [ ] Can view backup jobs
- [ ] Backup reports show status
- [ ] Alerts are configured for failures
- [ ] Storage usage is monitored
- [ ] Restore testing is possible

## Cleanup

Disable backups and remove vault:

```powershell
# Disable backup for VM
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints -Force

# Delete Recovery Services vault
Remove-AzRecoveryServicesVault -Vault $vault -Force
```

## Review

In this lab, you:
- Created Recovery Services vault for centralized backup management
- Designed and implemented backup policies
- Enabled backups for VMs and file shares
- Performed restore operations
- Monitored backup status and managed recovery points

Backup and recovery are critical for business continuity and disaster recovery.

## Additional Resources

- [Azure Backup](https://learn.microsoft.com/en-us/azure/backup/)
- [Backup VM](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-introduction)
- [Backup File Share](https://learn.microsoft.com/en-us/azure/backup/azure-file-share-backup-overview)
- [Recovery and Restore](https://learn.microsoft.com/en-us/azure/backup/restore-overview)
- [Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)

## Notes for Instructors

- Test restore procedures regularly
- Document RTO and RPO requirements
- Use GRS replication for critical backups
- Enable soft delete for protection against accidental deletion
- Monitor backup costs (can be significant)
- Consider Azure Site Recovery for DR
- Implement immutable backups for compliance
- Document backup procedures and runbooks

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
