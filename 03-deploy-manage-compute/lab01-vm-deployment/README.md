# Lab 01: Deploy and Manage Virtual Machines

## Objectives

By completing this lab, you will be able to:
- [ ] Create virtual machines in Azure
- [ ] Configure VM properties and settings
- [ ] Manage VM disks and storage
- [ ] Implement VM backup and recovery
- [ ] Monitor VM performance

## Prerequisites

- Active Azure subscription
- Resource group created
- Virtual network (or use default)
- Azure Portal or PowerShell/CLI
- Basic understanding of operating systems
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation needs to deploy Windows and Linux virtual machines for their applications. You'll create VMs, configure storage, implement backup, and monitor performance.

## Architecture

```
Resource Group: RG-Compute
├── Virtual Network: vnet-prod
│   └── Subnet: subnet-01
├── Virtual Machines
│   ├── VM-Windows (Windows Server 2022)
│   │   ├── OS Disk: Premium SSD
│   │   └── Data Disk: Standard SSD
│   └── VM-Linux (Ubuntu 22.04)
│       └── OS Disk: Premium SSD
├── Network Security Groups
│   └── NSG-VM
└── Storage Account (diagnostics)
```

## Task 1: Create Virtual Machines

### Instructions

1. Navigate to **Virtual machines** in Azure Portal
2. Click **+ Create** > **Virtual machine**
3. Create Windows VM:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Compute (create if needed)
   - **VM name**: VM-Windows
   - **Region**: East US
   - **Image**: Windows Server 2022 Datacenter
   - **Size**: Standard_B2s
   - **Username**: azureuser
   - **Password**: Create secure password
   - **Licensing**: Include eligible Windows Server license
   - Click **Next: Disks**
   - **OS disk type**: Premium SSD
   - Click **Next: Networking**
   - Create new VNet: vnet-prod
   - Create new NSG: NSG-VM
   - Click **Review + create**

4. Create Linux VM:
   - **VM name**: VM-Linux
   - **Image**: Ubuntu 22.04 LTS
   - **Authentication type**: SSH public key (or password)
   - **Username**: azureuser
   - Same network and NSG as Windows VM
   - Click **Create**

5. Wait for both VMs to deploy (5-10 minutes)

### Verification

- [ ] Both VMs are created successfully
- [ ] VMs appear in Virtual machines list
- [ ] Both VMs have public IP addresses
- [ ] Network interface is attached
- [ ] OS disks are created

### PowerShell Example

```powershell
# Create resource group
New-AzResourceGroup -Name "RG-Compute" -Location "eastus"

# Create credentials
$adminUsername = "azureuser"
$adminPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential($adminUsername, $adminPassword)

# Create Windows VM
$vm = New-AzVM -ResourceGroupName "RG-Compute" `
    -Name "VM-Windows" `
    -Location "eastus" `
    -Image "Win2022Datacenter" `
    -Size "Standard_B2s" `
    -Credential $cred `
    -PublicIpAddressName "pip-win" `
    -OpenPorts 3389, 80, 443

# Create Linux VM
New-AzVM -ResourceGroupName "RG-Compute" `
    -Name "VM-Linux" `
    -Location "eastus" `
    -Image "UbuntuLTS" `
    -Size "Standard_B2s" `
    -Credential $cred `
    -PublicIpAddressName "pip-linux" `
    -OpenPorts 22, 80, 443
```

## Task 2: Configure VM Disks and Storage

### Instructions

1. Navigate to **VM-Windows** > **Disks**
2. Add data disk:
   - Click **+ Add data disk**
   - **Name**: VM-Windows-data
   - **Size**: 32 GB
   - **Type**: Standard SSD
   - Click **Create and attach**

3. Repeat for **VM-Linux**:
   - Add data disk (32 GB)

4. Initialize disk on Windows VM:
   - Connect to VM-Windows via RDP
   - Open Disk Management
   - Initialize new disk
   - Create new volume
   - Format as NTFS

5. Initialize disk on Linux VM:
   - Connect to VM-Linux via SSH
   - Run: `sudo lsblk` (list disks)
   - Partition disk: `sudo parted /dev/sdc`
   - Format: `sudo mkfs.ext4 /dev/sdc1`
   - Mount: `sudo mount /dev/sdc1 /data`

6. Verify disk usage:
   - Windows: Check Disk Management
   - Linux: Check `df -h`

### Verification

- [ ] Data disks are attached to both VMs
- [ ] Disks are initialized and formatted
- [ ] Volumes are accessible from OS
- [ ] Can write test files to new volumes
- [ ] Storage capacity is correct

### PowerShell Example

```powershell
# Add data disk to Windows VM
$vm = Get-AzVM -ResourceGroupName "RG-Compute" -Name "VM-Windows"
Add-AzVMDataDisk -VM $vm `
    -Name "VM-Windows-data" `
    -DiskSizeGB 32 `
    -LUN 0 `
    -StorageAccountType "StandardSSD_LRS" `
    -CreateOption "Empty"
Update-AzVM -ResourceGroupName "RG-Compute" -VM $vm

# Get disk information
Get-AzDisk -ResourceGroupName "RG-Compute" | Format-Table
```

## Task 3: Implement VM Backup

### Instructions

1. Create Recovery Services Vault:
   - Navigate to **Recovery Services vaults**
   - Click **+ Create**
   - **Name**: contoso-backup-vault
   - **Resource group**: RG-Compute
   - **Region**: Same as VMs
   - Click **Create**

2. Enable backup for VM-Windows:
   - Open vault > **Backup** (getting started)
   - **Where is your workload running?**: Azure
   - **What do you want to backup?**: Virtual machine
   - Click **Backup**
   - Select **VM-Windows**
   - Create backup policy (daily backups, 30-day retention)

3. Enable backup for VM-Linux:
   - Repeat same steps for VM-Linux

4. Trigger backup:
   - Go to vault > **Backup items**
   - Click on VM-Windows
   - Click **Backup now**
   - Wait for backup to complete

### Verification

- [ ] Recovery Services vault is created
- [ ] Both VMs are protected
- [ ] Backup policy is configured
- [ ] Initial backup is completed
- [ ] Can see backup items and status

### PowerShell Example

```powershell
# Create Recovery Services vault
New-AzRecoveryServicesVault -Name "contoso-backup-vault" `
    -ResourceGroupName "RG-Compute" `
    -Location "eastus"

# Set vault context
$vault = Get-AzRecoveryServicesVault -Name "contoso-backup-vault"
Set-AzRecoveryServicesVaultContext -Vault $vault

# Enable backup for VM
$vm = Get-AzVM -ResourceGroupName "RG-Compute" -Name "VM-Windows"
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG-Compute" `
    -Name "VM-Windows" `
    -Policy $policy

# Trigger backup
Backup-AzRecoveryServicesBackupItem -Item (Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered")[0]
```

## Task 4: Monitor VM Performance

### Instructions

1. Navigate to **VM-Windows** > **Monitoring** > **Metrics**
2. Create metric charts:
   - **CPU Percentage** (aggregate: Average)
   - **Network In/Out** (total bytes)
   - **Disk Read/Write** (bytes per second)
   - **Available Memory** (bytes)

3. Set up alerts:
   - Go to **Alerts** > **Create alert rule**
   - **Metric**: CPU Percentage
   - **Condition**: Greater than 80%
   - **Window size**: 5 minutes
   - **Frequency**: Every 1 minute
   - **Action**: Send email

4. View diagnostic logs:
   - Go to **Diagnostics settings**
   - Enable boot diagnostics
   - Enable guest OS diagnostics (if enabled)
   - Select storage account for logs

5. Generate load to test alerts (optional):
   - Use stress testing tool
   - Monitor CPU usage in metrics
   - Verify alert notification

### Verification

- [ ] Metrics dashboard is created
- [ ] Charts show performance data
- [ ] Alerts are configured
- [ ] Alert thresholds are appropriate
- [ ] Diagnostics are enabled
- [ ] Can download diagnostic logs

## Task 5: Advanced VM Management

### Instructions

1. Configure VM extensions (Windows only):
   - Go to **VM-Windows** > **Extensions + applications**
   - Install Microsoft Monitoring Agent
   - This enables deeper monitoring

2. Create VM image (snapshot):
   - Go to **VM-Windows** > **Export template**
   - Download the template for reuse

3. Implement Run command:
   - Go to **VM-Windows** > **Run command**
   - Run PowerShell command: `Get-ComputerInfo`
   - View output without RDP

4. Configure auto-shutdown (optional):
   - Go to **Auto-shutdown**
   - Set shutdown time to save costs
   - Example: 7 PM daily

### Verification

- [ ] Extensions are installed
- [ ] Template is exported
- [ ] Run commands execute successfully
- [ ] Auto-shutdown is configured

## Cleanup

Remove VMs and resources:

```powershell
# Remove VMs
Remove-AzVM -ResourceGroupName "RG-Compute" -Name "VM-Windows" -Force
Remove-AzVM -ResourceGroupName "RG-Compute" -Name "VM-Linux" -Force

# Remove disks
Get-AzDisk -ResourceGroupName "RG-Compute" | Remove-AzDisk -Force

# Remove resource group
Remove-AzResourceGroup -Name "RG-Compute" -Force
```

## Review

In this lab, you:
- Created Windows and Linux VMs
- Configured storage disks
- Implemented backup and recovery
- Monitored VM performance
- Managed VM lifecycle

Proper VM management ensures reliability and performance of cloud infrastructure.

## Additional Resources

- [Azure Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [VM Sizes](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes)
- [Managed Disks](https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview)
- [Backup and Restore](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-introduction)
- [Monitoring VMs](https://learn.microsoft.com/en-us/azure/azure-monitor/vm/monitor-vm-azure)

## Notes for Instructors

- VM sizes should be chosen based on workload requirements
- Premium SSD provides better performance but costs more
- Always set up backups before production deployment
- Use managed disks for simplicity
- Monitor costs closely as VMs are billable resources
- Consider reserved instances for long-term cost savings
- Use update management to patch systems
- Encrypt sensitive data on VMs

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
