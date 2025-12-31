# Module 00: Getting Started with GitHub and VS Code

## Overview

This module will guide you through setting up your development environment to work with the AZ104 labs. You'll learn how to clone this repository from GitHub and install the necessary VS Code extensions.

## Prerequisites

- Windows 10/11 computer with administrator access
- Internet connection
- Web browser

## What You'll Learn

- Install Git for Windows
- Install Visual Studio Code
- Clone a GitHub repository
- Install and configure VS Code extensions
- Navigate the lab repository structure

---

## Task 1: Install Git for Windows

### Instructions

#### Step 1: Download Git

1. **Open your web browser**
   - Go to: [https://git-scm.com/download/win](https://git-scm.com/download/win)
   - Download should start automatically
   - File name: `Git-2.x.x-64-bit.exe`

2. **Run the Installer**
   - Open Downloads folder
   - Double-click the installer
   - Click **Yes** when prompted by User Account Control

#### Step 2: Install Git (Important Settings)

Follow these settings during installation:

1. **Select Destination Location**
   - Keep default: `C:\Program Files\Git`
   - Click **Next**

2. **Select Components**
   - ✅ Windows Explorer integration
   - ✅ Git Bash Here
   - ✅ Git GUI Here
   - ✅ Associate .git* configuration files
   - ✅ Associate .sh files to be run with Bash
   - Click **Next**

3. **Select Start Menu Folder**
   - Keep default: `Git`
   - Click **Next**

4. **Choose Default Editor**
   - Select: **Use Visual Studio Code as Git's default editor**
   - Click **Next**

5. **Adjust PATH Environment**
   - Select: **Git from the command line and also from 3rd-party software** (Recommended)
   - Click **Next**

6. **Choose HTTPS Transport Backend**
   - Select: **Use the OpenSSL library**
   - Click **Next**

7. **Configure Line Ending Conversions**
   - Select: **Checkout Windows-style, commit Unix-style line endings** (Recommended)
   - Click **Next**

8. **Configure Terminal Emulator**
   - Select: **Use Windows' default console window**
   - Click **Next**

9. **Choose Default Behavior**
   - Select: **Default (fast-forward or merge)**
   - Click **Next**

10. **Choose Credential Helper**
    - Select: **Git Credential Manager**
    - Click **Next**

11. **Configure Extra Options**
    - ✅ Enable file system caching
    - ✅ Enable symbolic links
    - Click **Next**

12. **Complete Installation**
    - Click **Install**
    - Wait for installation to complete
    - Click **Finish**

#### Step 3: Verify Git Installation

1. **Open PowerShell:**
   - Press `Win + X`
   - Select **Windows PowerShell** or **Terminal**

2. **Check Git Version:**
   ```powershell
   git --version
   ```
   
   Expected output:
   ```
   git version 2.x.x.windows.1
   ```

3. **Configure Git (First Time Setup):**
   ```powershell
   # Set your name (replace with your actual name)
   git config --global user.name "Your Name"
   
   # Set your email (replace with your email)
   git config --global user.email "your.email@example.com"
   
   # Verify configuration
   git config --list
   ```

### Verification Checklist

- [ ] Git installer completed successfully
- [ ] `git --version` command works in PowerShell
- [ ] Git user name is configured
- [ ] Git email is configured

---

## Task 2: Install Visual Studio Code

### Instructions

#### Step 1: Download VS Code

1. **Go to VS Code Website:**
   - Open browser: [https://code.visualstudio.com/](https://code.visualstudio.com/)
   - Click **Download for Windows** button
   - File downloads: `VSCodeUserSetup-x64-*.exe`

2. **Run the Installer:**
   - Go to Downloads folder
   - Double-click the installer
   - Click **Yes** for User Account Control

#### Step 2: Install VS Code (Recommended Settings)

1. **Accept License Agreement:**
   - Read the agreement
   - Click **I accept the agreement**
   - Click **Next**

2. **Select Destination:**
   - Keep default location
   - Click **Next**

3. **Select Start Menu Folder:**
   - Keep default
   - Click **Next**

4. **Select Additional Tasks (IMPORTANT):**
   - ✅ **Add "Open with Code" action to Windows Explorer file context menu**
   - ✅ **Add "Open with Code" action to Windows Explorer directory context menu**
   - ✅ **Register Code as an editor for supported file types**
   - ✅ **Add to PATH** (very important!)
   - Click **Next**

5. **Install:**
   - Review your choices
   - Click **Install**
   - Wait for installation to complete
   - ✅ **Launch Visual Studio Code**
   - Click **Finish**

#### Step 3: Verify VS Code Installation

1. **VS Code Should Launch Automatically**
   - Welcome screen appears
   - Explore the interface

2. **Verify from Command Line:**
   - Open PowerShell (new window if already open)
   ```powershell
   code --version
   ```
   
   Expected output:
   ```
   1.x.x
   <commit-id>
   x64
   ```

3. **Test Opening Folders:**
   ```powershell
   # Navigate to any folder
   cd C:\
   
   # Open VS Code in current folder
   code .
   ```
   - VS Code should open with the folder loaded

### Verification Checklist

- [ ] VS Code installed successfully
- [ ] Welcome screen appeared
- [ ] `code --version` works in PowerShell
- [ ] `code .` opens VS Code from command line

---

## Task 3: Clone the AZ104 Labs Repository

### Instructions

#### Step 1: Choose a Location for Your Labs

1. **Create a Working Directory:**
   ```powershell
   # Create a folder for all your Azure labs
   mkdir C:\AzureLabs
   
   # Navigate to it
   cd C:\AzureLabs
   ```

   Or choose a different location:
   ```powershell
   # Alternative: Use Documents folder
   cd $HOME\Documents
   mkdir AzureLabs
   cd AzureLabs
   ```

#### Step 2: Clone the Repository

1. **Clone Using Git Command:**
   ```powershell
   # Clone the repository (replace with actual repo URL)
   git clone https://github.com/YOUR-USERNAME/AzureLabs.git
   ```
   
   Expected output:
   ```
   Cloning into 'AzureLabs'...
   remote: Enumerating objects: 150, done.
   remote: Counting objects: 100% (150/150), done.
   remote: Compressing objects: 100% (95/95), done.
   Receiving objects: 100% (150/150), 250.00 KiB | 2.5 MiB/s, done.
   Resolving deltas: 100% (50/50), done.
   ```

2. **Navigate into Repository:**
   ```powershell
   cd AzureLabs
   ```

3. **List Contents:**
   ```powershell
   # List all folders
   dir
   ```
   
   Expected output:
   ```
   Directory: C:\AzureLabs\AzureLabs

   Mode                 LastWriteTime         Length Name
   ----                 -------------         ------ ----
   d-----                                            00-getting-started
   d-----                                            01-manage-identities-governance
   d-----                                            02-implement-manage-storage
   d-----                                            03-deploy-manage-compute
   d-----                                            04-configure-manage-networking
   d-----                                            05-monitor-backup-resources
   d-----                                            06-infrastructure-as-code-bicep
   -a----                                            README.md
   ```

#### Step 3: Open Repository in VS Code

1. **Open Using Command:**
   ```powershell
   code .
   ```
   
   OR

2. **Open Using VS Code UI:**
   - Launch VS Code
   - File > Open Folder
   - Navigate to `C:\AzureLabs\AzureLabs`
   - Click **Select Folder**

3. **Trust the Workspace:**
   - A dialog appears: "Do you trust the authors..."
   - Click **Yes, I trust the authors**
   - This enables all features

4. **Explore the Repository:**
   - **Explorer panel** (left) shows folder structure
   - Click on `README.md` to view main documentation
   - Expand folders to see individual labs

### Verification Checklist

- [ ] Repository cloned successfully
- [ ] Can navigate to repository folder
- [ ] Folder structure matches expected layout
- [ ] Repository opens in VS Code
- [ ] Can see all module folders
- [ ] README.md is readable

---

## Task 4: Install Required VS Code Extensions

### Instructions

#### Step 1: Open Extensions Panel

1. **Access Extensions:**
   - Press `Ctrl+Shift+X`
   
   OR
   
   - Click the **Extensions** icon in left sidebar (4 squares icon)

#### Step 2: Install Essential Extensions

Install each extension one by one:

##### 1. Azure Account

```
Extension Name: Azure Account
Publisher: Microsoft
ID: ms-vscode.azure-account
```

**To Install:**
- Search for: `Azure Account`
- Find the one by **Microsoft**
- Click **Install**
- Wait for green checkmark

**What it does:**
- Sign in to Azure from VS Code
- Access your subscriptions
- Manage Azure resources

##### 2. Azure Resources

```
Extension Name: Azure Resources
Publisher: Microsoft
ID: ms-azuretools.vscode-azureresourcegroups
```

**To Install:**
- Search for: `Azure Resources`
- Publisher: **Microsoft**
- Click **Install**

**What it does:**
- View all Azure resources
- Create and manage resources
- Access resource properties

##### 3. Azure Resource Manager (ARM) Tools

```
Extension Name: Azure Resource Manager (ARM) Tools
Publisher: Microsoft
ID: msazurermtools.azurerm-vscode-tools
```

**To Install:**
- Search for: `Azure Resource Manager`
- Select **Azure Resource Manager (ARM) Tools**
- Click **Install**

**What it does:**
- ARM/Bicep template support
- Template validation
- IntelliSense for Azure resources
- Deployment functionality

##### 4. Bicep

```
Extension Name: Bicep
Publisher: Microsoft
ID: ms-azuretools.vscode-bicep
```

**To Install:**
- Search for: `Bicep`
- Publisher: **Microsoft**
- Click **Install**

**What it does:**
- Bicep language support
- Syntax highlighting
- Code completion
- Built-in Bicep compiler

##### 5. PowerShell

```
Extension Name: PowerShell
Publisher: Microsoft
ID: ms-vscode.powershell
```

**To Install:**
- Search for: `PowerShell`
- Publisher: **Microsoft**
- Click **Install**

**What it does:**
- PowerShell script editing
- Integrated debugging
- Cmdlet IntelliSense

##### 6. Azure CLI Tools

```
Extension Name: Azure CLI Tools
Publisher: Microsoft
ID: ms-vscode.azurecli
```

**To Install:**
- Search for: `Azure CLI Tools`
- Publisher: **Microsoft**
- Click **Install**

**What it does:**
- Azure CLI command completion
- Syntax highlighting for .azcli files
- Command snippets

##### 7. Markdown All in One (Optional but Recommended)

```
Extension Name: Markdown All in One
Publisher: Yu Zhang
ID: yzhang.markdown-all-in-one
```

**To Install:**
- Search for: `Markdown All in One`
- Publisher: **Yu Zhang**
- Click **Install**

**What it does:**
- Better Markdown editing
- Table of contents generation
- Preview markdown files

#### Step 3: Verify Extensions

1. **Check Installed Extensions:**
   - In Extensions panel, click on filter icon
   - Select **Installed**
   - Should see all 7 extensions with green checkmarks

2. **Reload VS Code (if prompted):**
   - Some extensions require reload
   - Click **Reload** button if it appears
   
   OR
   
   - Press `Ctrl+Shift+P`
   - Type: `Developer: Reload Window`
   - Press Enter

### Verification Checklist

- [ ] Azure Account extension installed
- [ ] Azure Resources extension installed
- [ ] Azure Resource Manager Tools installed
- [ ] Bicep extension installed
- [ ] PowerShell extension installed
- [ ] Azure CLI Tools installed
- [ ] Markdown All in One installed (optional)
- [ ] All extensions show green checkmarks
- [ ] VS Code reloaded if necessary

---

## Task 5: Configure Azure Authentication

### Instructions

#### Step 1: Install Azure CLI

1. **Download Azure CLI:**
   - Go to: [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows)
   - Download `azure-cli-latest.msi`

2. **Install Azure CLI:**
   - Run the downloaded installer
   - Click **Next** through all prompts
   - Click **Install**
   - Wait for completion
   - Click **Finish**

3. **Restart VS Code:**
   - Close VS Code completely
   - Reopen it (to load Azure CLI in PATH)

4. **Verify Installation:**
   ```powershell
   # Open terminal in VS Code (Ctrl+`)
   az version
   ```
   
   Expected output:
   ```json
   {
     "azure-cli": "2.x.x",
     "azure-cli-core": "2.x.x",
     ...
   }
   ```

#### Step 2: Sign in to Azure (VS Code Method)

1. **Sign in Using VS Code:**
   - Press `Ctrl+Shift+P` to open Command Palette
   - Type: `Azure: Sign In`
   - Press Enter

2. **Complete Authentication:**
   - Browser window opens
   - Sign in with your Azure credentials
   - After successful sign-in, you see: "You are signed in"
   - Close browser tab

3. **Verify Sign-In in VS Code:**
   - Look at **bottom-left corner** of VS Code
   - Should display your Azure account email
   - Click on it to see your subscriptions

#### Step 3: Sign in to Azure (CLI Method - Alternative)

1. **Sign in Using CLI:**
   ```powershell
   # In VS Code terminal (Ctrl+`)
   az login
   ```

2. **Complete Authentication:**
   - Browser opens automatically
   - Sign in with Azure credentials
   - Return to VS Code
   - Terminal shows your subscriptions

3. **Verify Sign-In:**
   ```powershell
   # List your subscriptions
   az account list --output table
   
   # Show current subscription
   az account show
   ```

#### Step 4: Set Default Subscription (if you have multiple)

1. **List Available Subscriptions:**
   ```powershell
   az account list --output table
   ```

2. **Set Default Subscription:**
   ```powershell
   # Replace with your subscription ID
   az account set --subscription "YOUR-SUBSCRIPTION-ID"
   
   # Or use subscription name
   az account set --subscription "Visual Studio Enterprise"
   ```

3. **Verify:**
   ```powershell
   az account show --query name -o tsv
   ```

### Verification Checklist

- [ ] Azure CLI installed (`az version` works)
- [ ] Signed in to Azure (email shows in VS Code)
- [ ] Can see subscriptions in Azure panel
- [ ] Default subscription is set
- [ ] `az account show` displays current subscription

---

## Task 6: Explore the Repository Structure

### Instructions

#### Step 1: Understand the Repository Layout

1. **Open README.md:**
   - In VS Code Explorer, click on `README.md`
   - Read the main documentation
   - Understand the learning objectives

2. **Repository Structure:**
   ```
   AzureLabs/
   ├── 00-getting-started/          ← You are here!
   ├── 01-manage-identities-governance/
   │   ├── lab01-entra-users-groups/
   │   ├── lab02-rbac-custom-roles/
   │   └── lab03-subscriptions-management-groups/
   ├── 02-implement-manage-storage/
   │   ├── lab01-storage-accounts/
   │   ├── lab02-blob-lifecycle/
   │   └── lab03-azure-files-sync/
   ├── 03-deploy-manage-compute/
   │   ├── lab01-vm-deployment/
   │   ├── lab02-vm-scale-sets/
   │   └── lab03-containers-aci/
   ├── 04-configure-manage-networking/
   │   ├── lab01-vnets-peering/
   │   ├── lab02-nsg-asg/
   │   └── lab03-load-balancers/
   ├── 05-monitor-backup-resources/
   │   ├── lab01-azure-monitor/
   │   ├── lab02-backup-recovery/
   │   └── lab03-log-analytics/
   ├── 06-infrastructure-as-code-bicep/
   │   ├── lab01-bicep-basics/
   │   ├── lab02-bicep-storage-vms/
   │   └── lab03-bicep-advanced/
   └── README.md
   ```

#### Step 2: Navigate the Labs

1. **Browse Module Folders:**
   - Click on `01-manage-identities-governance`
   - Expand to see lab folders
   - Each lab has its own `README.md`

2. **Open a Lab README:**
   - Navigate to: `01-manage-identities-governance/lab01-entra-users-groups/`
   - Click on `README.md`
   - Review the lab structure

3. **Use Quick Open:**
   - Press `Ctrl+P`
   - Type: `lab01` 
   - See all files matching "lab01"
   - Select one to open

#### Step 3: Use VS Code Features

1. **Markdown Preview:**
   - While viewing a README.md file
   - Press `Ctrl+Shift+V`
   - Preview panel opens showing formatted markdown
   - OR click preview icon (top-right, split view icon)

2. **Side-by-Side View:**
   - Press `Ctrl+K V` while viewing README.md
   - Opens preview side-by-side
   - Edit on left, see preview on right

3. **Search Across Files:**
   - Press `Ctrl+Shift+F`
   - Type search term (e.g., "storage account")
   - See all occurrences across all labs

4. **Navigate Files:**
   - `Ctrl+P` - Quick open file
   - `Ctrl+Tab` - Switch between open files
   - `Ctrl+\` - Split editor

### Verification Checklist

- [ ] Can navigate folder structure
- [ ] Can open README files
- [ ] Markdown preview works
- [ ] Can search across files
- [ ] Understand repository organization

---

## Task 7: Test Your Setup

### Instructions

#### Step 1: Create a Test File

1. **Create a Test Bicep File:**
   - In Explorer, right-click on `00-getting-started` folder
   - Select **New File**
   - Name it: `test.bicep`

2. **Add Test Code:**
   ```bicep
   param location string = 'eastus'
   
   resource storageTest 'Microsoft.Storage/storageAccounts@2023-01-01' = {
     name: 'sttest${uniqueString(resourceGroup().id)}'
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'StorageV2'
   }
   ```

3. **Test IntelliSense:**
   - Type `param` - suggestions should appear
   - Type `resource` - templates should appear
   - Hover over `Microsoft.Storage` - see documentation

4. **Verify No Errors:**
   - Check bottom-left: should show "Bicep: ✓"
   - No red squiggly lines in code

5. **Delete Test File:**
   - Right-click on `test.bicep`
   - Select **Delete**
   - Confirm deletion

#### Step 2: Test Azure Connectivity

1. **Open Azure Resources Panel:**
   - Click Azure icon in left sidebar
   - Should see your subscriptions

2. **Expand Subscription:**
   - Click arrow next to your subscription
   - Should see resource groups

3. **Test CLI Commands:**
   ```powershell
   # Test Azure CLI
   az group list --output table
   
   # Should list your resource groups
   ```

#### Step 3: Test Git Integration

1. **Check Git Status:**
   - Click **Source Control** icon (left sidebar, looks like branch)
   - Should show "No changes"

2. **Make a Test Change:**
   - Open `README.md` in `00-getting-started`
   - Add a line at the end: `<!-- Test -->`
   - Save file

3. **See Git Detect Change:**
   - Source Control icon shows "1"
   - Click on it
   - See `README.md` listed under Changes

4. **Discard Change:**
   - Hover over `README.md` in Source Control
   - Click discard icon (↶)
   - Confirm discard

### Verification Checklist

- [ ] Bicep file shows syntax highlighting
- [ ] IntelliSense works (autocomplete)
- [ ] No Bicep errors appear
- [ ] Azure panel shows subscriptions
- [ ] Can list resource groups via CLI
- [ ] Git detects file changes
- [ ] Source Control panel works

---

## Troubleshooting

### Issue: Git Command Not Found

**Symptoms:**
```
'git' is not recognized as an internal or external command
```

**Solutions:**
1. Restart PowerShell/VS Code after Git installation
2. Verify Git is in PATH:
   ```powershell
   $env:PATH -split ';' | Select-String -Pattern 'Git'
   ```
3. Reinstall Git with "Add to PATH" option checked

---

### Issue: Cannot Clone Repository

**Symptoms:**
```
fatal: repository not found
```

**Solutions:**
1. Verify repository URL is correct
2. Check internet connection
3. If private repo, authenticate:
   ```powershell
   git config --global credential.helper manager
   ```
4. Try cloning again - will prompt for credentials

---

### Issue: VS Code Extensions Not Working

**Symptoms:**
- Extensions installed but features don't work
- No IntelliSense or syntax highlighting

**Solutions:**
1. Reload VS Code:
   - `Ctrl+Shift+P` → "Developer: Reload Window"
2. Check extension is enabled:
   - Extensions panel → Find extension → Ensure enabled
3. Reinstall extension:
   - Right-click extension → Uninstall → Reinstall

---

### Issue: Azure Sign-In Fails

**Symptoms:**
- "Failed to sign in" error
- Browser doesn't open

**Solutions:**
1. Check firewall/proxy settings
2. Try device code flow:
   ```powershell
   az login --use-device-code
   ```
3. Follow instructions in terminal
4. Clear Azure credentials:
   ```powershell
   az account clear
   az login
   ```

---

### Issue: Bicep CLI Not Found

**Symptoms:**
```
'bicep' is not recognized as a command
```

**Solutions:**
1. Install Bicep CLI:
   ```powershell
   az bicep install
   ```
2. Verify installation:
   ```powershell
   az bicep version
   ```
3. If Azure CLI is not installed, install it first

---

## Next Steps

🎉 **Congratulations!** Your development environment is ready.

### Recommended Learning Path:

1. **Start with Module 01**: [Manage Identities and Governance](../01-manage-identities-governance/README.md)
2. **Work through modules sequentially**
3. **Complete all labs in each module before moving on**
4. **Review Azure documentation** for deeper understanding

### Additional Resources:

- [Azure Documentation](https://docs.microsoft.com/azure/)
- [AZ-104 Exam Skills Outline](https://learn.microsoft.com/certifications/exams/az-104)
- [VS Code Documentation](https://code.visualstudio.com/docs)
- [Git Documentation](https://git-scm.com/doc)
- [Azure CLI Reference](https://learn.microsoft.com/cli/azure/)

---

## Summary

You have successfully:
- ✅ Installed Git for Windows
- ✅ Installed Visual Studio Code
- ✅ Cloned the AZ104 Labs repository
- ✅ Installed required VS Code extensions
- ✅ Configured Azure authentication
- ✅ Explored the repository structure
- ✅ Tested your setup

**You're now ready to begin the AZ-104 labs!** 🚀

[Proceed to Module 01: Manage Identities and Governance →](../01-manage-identities-governance/README.md)
