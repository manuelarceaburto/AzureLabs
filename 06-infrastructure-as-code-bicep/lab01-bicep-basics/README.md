# Lab 01: Bicep Basics - Resource Groups and Storage Accounts

## Objectives

By completing this lab, you will be able to:
- [ ] Set up VS Code for Bicep development
- [ ] Create your first Bicep file
- [ ] Understand Bicep syntax and structure
- [ ] Use parameters and variables
- [ ] Deploy resources using Bicep
- [ ] Understand outputs

## Prerequisites

- VS Code installed with Bicep extension
- Azure CLI installed and configured
- Active Azure subscription
- Basic understanding of Azure resources
- Estimated time: 30 minutes

## Lab Scenario

Contoso Corporation wants to standardize resource deployment using Infrastructure as Code. You'll create a Bicep template to deploy a resource group and storage account consistently across environments.

## Task 1: Install and Set Up VS Code for Bicep

### Instructions

#### Step 1: Install Visual Studio Code

1. **Download VS Code:**
   - Go to [https://code.visualstudio.com/](https://code.visualstudio.com/)
   - Click **Download for Windows**
   - Run the installer (`VSCodeUserSetup-x64-*.exe`)

2. **Install VS Code:**
   - Accept the license agreement
   - Choose installation location
   - ✅ Check **Add to PATH** (important!)
   - ✅ Check **Create a desktop icon**
   - ✅ Check **Add "Open with Code" action to Windows Explorer context menu**
   - Click **Install**
   - Click **Finish** to launch VS Code

#### Step 2: Install Required Extensions

1. **Open Extensions Panel:**
   - Launch VS Code
   - Press `Ctrl+Shift+X` (or click Extensions icon in left sidebar)
   - The Extensions panel opens

2. **Install Bicep Extension:**
   - In the search box, type: `Bicep`
   - Find **"Bicep"** by Microsoft
   - Click **Install** button
   - Wait for installation to complete (green checkmark appears)
   
   **What this does:**
   - Syntax highlighting for .bicep files
   - IntelliSense (autocomplete)
   - Validation and error checking
   - Built-in Bicep compiler

3. **Install Azure Resource Manager (ARM) Tools:**
   - In the search box, type: `Azure Resource Manager`
   - Find **"Azure Resource Manager (ARM) Tools"** by Microsoft
   - Click **Install**
   
   **What this does:**
   - Deployment pane for easy deployments
   - Template validation
   - Parameter file support
   - Visualization of resources

4. **Install Azure Account Extension:**
   - Search for: `Azure Account`
   - Find **"Azure Account"** by Microsoft
   - Click **Install**
   
   **What this does:**
   - Azure authentication from VS Code
   - Subscription management
   - Resource group access

5. **Install Azure Resources Extension (Optional):**
   - Search for: `Azure Resources`
   - Find **"Azure Resources"** by Microsoft
   - Click **Install**
   
   **What this does:**
   - View Azure resources in VS Code
   - Right-click deploy from explorer

#### Step 3: Install Bicep CLI

1. **Open Terminal in VS Code:**
   - Press `Ctrl+`` (backtick) or View > Terminal
   - A terminal opens at the bottom of VS Code

2. **Install Bicep CLI:**
   ```powershell
   # Install Bicep CLI via Azure CLI
   az bicep install
   ```
   
   Output should show:
   ```
   Installing Bicep CLI...
   Successfully installed Bicep CLI
   ```

3. **Verify Installation:**
   ```powershell
   # Check Bicep version
   az bicep version
   ```
   
   Expected output:
   ```
   Bicep CLI version 0.x.x (abcd1234)
   ```

#### Step 4: Sign in to Azure

1. **Open Command Palette:**
   - Press `Ctrl+Shift+P`
   - Type: `Azure: Sign In`
   - Press Enter

2. **Authenticate:**
   - Browser window opens
   - Sign in with your Azure credentials
   - Close browser when you see "You are signed in"

3. **Verify Sign-In:**
   - Look at the bottom left of VS Code
   - Should show your Azure account email
   - Click on it to see subscriptions

4. **Alternative: Sign in via CLI:**
   ```powershell
   # In VS Code terminal
   az login
   ```
   - Browser opens for authentication
   - Sign in with Azure credentials

#### Step 5: Create Project Folder

1. **Create Folder Structure:**
   ```powershell
   # In VS Code terminal
   mkdir C:\MyBicepProject
   cd C:\MyBicepProject
   ```

2. **Open Folder in VS Code:**
   - File > Open Folder
   - Navigate to `C:\MyBicepProject`
   - Click **Select Folder**

   OR via terminal:
   ```powershell
   code .
   ```

3. **Verify Setup:**
   - Explorer panel (left side) shows the folder
   - Terminal is open at the bottom
   - Status bar (bottom) shows Azure account

### Verification Checklist

- [ ] VS Code is installed and running
- [ ] Bicep extension shows in Extensions panel (green checkmark)
- [ ] Azure Account extension shows in Extensions panel
- [ ] ARM Tools extension shows in Extensions panel
- [ ] Terminal opens with `Ctrl+``
- [ ] `az bicep version` command works
- [ ] Signed in to Azure (see email in bottom left)
- [ ] Project folder is open in VS Code
- [ ] Can see file explorer on left side

### Troubleshooting

**Issue: "az: command not found"**
- Azure CLI is not installed
- Install from: https://aka.ms/installazurecliwindows
- Restart VS Code after installation

**Issue: "Bicep extension not working"**
- Reload VS Code: `Ctrl+Shift+P` → "Developer: Reload Window"
- Check extension is enabled in Extensions panel

**Issue: "Can't sign in to Azure"**
- Check internet connection
- Try: `az login --use-device-code` in terminal
- Follow instructions to authenticate

## Task 2: Create Your First Bicep File

### Instructions

1. **Create main.bicep file:**
   - In VS Code Explorer (left panel), right-click in empty space
   - Select **"New File"**
   - Type the name: `main.bicep`
   - Press Enter
   - File opens in editor

   **Notice:**
   - File icon shows Bicep logo (if extension is installed)
   - Status bar shows "Bicep" language mode

2. **Add basic structure:**
   
   **Type or paste the following code:**
   ```bicep
   // ============================
   // Bicep Template for Storage Account
   // ============================
   
   // Parameters - inputs to the template
   param location string = 'eastus'
   param environment string = 'dev'
   param projectName string = 'contoso'
    Using VS Code Deployment Panel

### Method 1: Deploy Using VS Code UI (Recommended for Beginners)

#### Instructions

1. **Open Bicep File:**
   - In VS Code Explorer, click on `main.bicep`
   - File opens in editor

2. **Access Deployment Panel:**
   - Right-click anywhere in the `main.bicep` file
   - Select **"Deploy Bicep File..."** from context menu
   
   OR
   
   - Press `Ctrl+Shift+P` to open Command Palette
   - Type: `Bicep: Deploy`
   - Select **"Azure Resource Manager Tools: Deploy Template"**

3. **Configure Deployment (Step-by-Step Prompts):**

   **Prompt 1: Select Subscription**
   - Test IntelliSense (Autocomplete):**
   - Start typing `param` - suggestions appear
   - Start typing `resource` - template snippets appear
   - Press `Ctrl+Space` anywhere to trigger suggestions
   
   **Try this:**
   - Type `res` and press Tab
   - Bicep inserts a resource template snippet
   - Delete it (we already have the complete template)

4. **Save the File:**
   - Press `Ctrl+S` to save
   - OR File > Save

5. **Understand the structure:**
   - **Parameters**: Inputs to template (location, environment, etc.)
   - **Variables**: Computed values (storage account name, tags)
   - **Resources**: Azure resources to create
   - **Outputs**: Values returned after deployment

6. **Validate Syntax in VS Code:**
   - Look at the bottom left of VS Code
   - Should show: "Bicep: ✓" (green checkmark)
   - If errors exist: "Bicep: ⚠ 3" (number shows error count)
   - Click on "Problems" tab at bottom to see errors

### Verification Checklist

- [ ] File is created as `main.bicep`
- [ ] Syntax is highlighted (colors for keywords, strings)
- [ ] No red squiggly error lines
- [ ] IntelliSense suggestions appear when typing
- [ ] "Bicep: ✓" appears in status bar (bottom left)
- [ ] Hovering over resources shows documentation

### Understanding IntelliSense Features

**Autocomplete:**
- Type `param` → get parameter templates
- Type `resource` → get resource templates
- Type `.` after an object → see available properties

**Hover Information:**
- Hover over a resource type → see API version info
- Hover over a parameter → see description
- Hover over a function → see function documentation

**Go to Definition:**
- `Ctrl+Click` on a variable name → jump to definition
- `F12` on a parameter → jump to declaration
   - Press Enter

   **Prompt 3: Enter Deployment Name**
   - Enter: `StorageDeployment`
   - Press Enter

   **Prompt 4: Select Parameter File**
   - Dropdown shows: **"parameters.json"**
   - Select it (or browse to the file)
   - Press Enter

   **Prompt 5: Enter Parameter Values**
   - VS Code reads parameters.json automatically
   - If prompted for values, enter them:
     - `location`: eastus
     - `environment`: dev
     - `projectName`: contoso

4. **Monitor Deployment:**
   - **Output Panel** opens automatically (bottom of VS Code)
   - Shows deployment progress:
     ```
     Deploying template...
     Creating storage account...
     Deployment succeeded
     ```
   - Takes 1-2 minutes

5. **View Deployment Results:**
   - Output panel shows:
     - ✅ Deployment succeeded
     - Resource IDs
     - Output values
   
   - Click **"View in Portal"** link to see resources

6. **View Outputs in VS Code:**
   - Outputs appear in the Output panel:
     ```json
     {
       "storageAccountId": { "value": "/subscriptions/..." },
       "storageAccountName": { "value": "stcontosodev..." },
       "storageAccountUri": { "value": "https://..." }
     }
     ```

### Method 2: Deploy Using Terminal (Alternative)

#### Instructions

1. **Open Terminal in VS Code:**
   - Press `Ctrl+`` (backtick)
   - Terminal appears at bottom

2. **Create Resource Group:**
   ```powershell
   az group create --name rg-bicep-lab01 --location eastus
   ```

3. **Deploy using Azure CLI:**
   ```powershell
   az deployment group create `
     --name StorageDeployment `
     --resource-group rg-bicep-lab01 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

4. **Monitor in Terminal:**
   - Output shows progress
   - Wait for "Succeeded" status

5. **View Deployment Outputs:**
   ```powershell
   az deployment group show `
     --resource-group rg-bicep-lab01 `
     --name StorageDeployment `
     --query 'properties.outputs'
   ```

### Method 3: Deploy Using Azure Resources Extension

#### Instructions

1. **Open Azure Resources Panel:**
   - Click Azure icon in left sidebar (looks like "A")
   - Expands to show subscriptions

2. **Navigate to Resource Groups:**
   - Expand your subscription
   - Find "Resource Groups"
   - Right-click on resource group or empty space
   - Select **"Create Resource Group"** (if new)

3. **Deploy to Resource Group:**
   - Right-click on resource group `rg-bicep-lab01`
   - Select **"Deploy to Resource Group..."**
   - Follow prompts (same as Method 1)

### Verify Deployment

#### Option A: Verify in VS Code

1. **Using Azure Resources Extension:**
   - Click Azure icon (left sidebar)
   - Expand subscription
   - Expand resource group `rg-bicep-lab01`
   - Should see:
     - Storage Account resource
     - With generated name

2. **View Resource Properties:**
   - Right-click on storage account
   - Select **"Open in Portal"**
   - Azure Portal opens showing resource

#### Option B: Verify in Azure Portal

1. **Open Portal:**
   - Go to [https://portal.azure.com](https://portal.azure.com)
   - Sign in

2. **Navigate to Resource Group:**
   - Search for "Resource Groups"
   - Click on `rg-bicep-lab01`
   - Should see storage account listed

3. **Check Resource:**
   - Click on storage account name
   - Verify properties:
     - Tags (environment: dev, project: contoso)
     - Location (East US)
     - SKU (Standard_LRS)

#### Option C: Verify via CLI

```powershell
# List all resources in group
az resource list --resource-group rg-bicep-lab01 --output table

# Get storage account details
az storage account list --resource-group rg-bicep-lab01 --output table
```

### Verification Checklist

- [ ] Deployment completed without errors
- [ ] Output panel shows "Succeeded"
- [ ] Storage account appears in resource group
- [ ] Storage account has correct name pattern
- [ ] Tags are applied (environment, project)
- [ ] Outputs are visible in Output panel
- [ ] Can access resource in Azure Portal

### Troubleshooting Deployment Issues

**Issue: "Template validation failed"**
- Check for syntax errors in main.bicep
- Red squiggly lines in VS Code indicate errors
- Hover over errors to see details

**Issue: "Parameter file not found"**
- Ensure parameters.json is in same folder as main.bicep
- Check file name spelling

**Issue: "Insufficient permissions"**
- Verify you have Contributor role on subscription
- Check with: `az role assignment list --assignee <your-email>`

**Issue: "Deployment stuck/timeout"**
- Check Output panel for errors
- Storage account names must be globally unique
- Try deploying with different parameter value
### Instructions

1. **Create parameters.json:**
   - Right-click in Explorer
   - Select "New File"
   - Name it `parameters.json`

2. **Add parameters:**
   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "location": {
         "value": "eastus"
       },
       "environment": {
         "value": "dev"
       },
       "projectName": {
         "value": "contoso"
       }
     }
   }
   ```

3. **Understand the structure:**
   - Follows ARM template parameter schema
   - References parameters from Bicep file
   - Can change values without modifying Bicep
   - Useful for different environments (dev, prod)

### Verification

- [ ] File is created as `parameters.json`
- [ ] JSON is valid (no syntax errors)
- [ ] Parameter names match Bicep file

## Task 4: Validate and Build

### Instructions

1. **Open terminal in VS Code:**
   - Press `Ctrl+`
   - Terminal opens at bottom

2. **Validate Bicep syntax:**
   ```powershell
   az bicep build --file main.bicep
   ```
   This converts Bicep to ARM template JSON and checks for errors

3. **Review generated ARM template:**
   ```powershell
   # View the generated JSON
   Get-Content main.json | ConvertFrom-Json | ConvertTo-Json
   ```

4. **Validate deployment (without deploying):**
   ```powershell
   # First, create resource group
   az group create --name rg-bicep-lab01 --location eastus
   
   # Validate
   az deployment group validate `
     --resource-group rg-bicep-lab01 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

### Verification

- [ ] `az bicep build` completes without errors
- [ ] `main.json` file is created
- [ ] Validation succeeds with no errors
- [ ] Resource group is created

## Task 5: Deploy Resources

### Instructions

1. **Deploy using Bicep:**
   ```powershell
   az deployment group create `
     --name StorageDeployment `
     --resource-group rg-bicep-lab01 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

2. **Monitor deployment:**
   - Wait for "Succeeded" status in terminal
   - Takes 1-2 minutes typically

3. **View deployment outputs:**
   ```powershell
   # Get deployment details
   az deployment group show `
     --resource-group rg-bicep-lab01 `
     --name StorageDeployment `
     --query 'properties.outputs'
   ```

4. **Verify resources in portal:**
   - Go to Azure Portal
   - Navigate to resource group `rg-bicep-lab01`
   - Should see storage account with generated name

5. **View resources via CLI:**
   ```powershell
   # List resources in group
   az resource list --resource-group rg-bicep-lab01
   
   # Get storage account details
   az storage account show --resource-group rg-bicep-lab01 --name <storage-account-name>
   ```

### Verification

- [ ] Deployment status shows "Succeeded"
- [ ] Storage account is visible in resource group
- [ ] Storage account has correct tags
- [ ] Can retrieve deployment outputs

## Task 6: Modify and Redeploy

### Instructions

1. **Modify parameters for production:**
   - Edit `parameters.json`
   - Change environment from "dev" to "prod"
   - Save file

2. **Deploy again:**
   ```powershell
   az deployment group create `
     --name StorageDeployment-Prod `
     --resource-group rg-bicep-lab01 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

3. **Notice the changes:**
   - New storage account with "prod" in name
   - Tags updated with new environment
   - Original storage account still exists

4. **View both resources:**
   ```powershell
   az resource list --resource-group rg-bicep-lab01 `
     --query "[].name" -o table
   ```

### Verification

- [ ] Second deployment succeeds
- [ ] Two storage accounts exist (dev and prod)
- [ ] Parameters were properly substituted
- [ ] Tags show correct environment

## Understanding Key Bicep Concepts

### Parameters
```bicep
param location string = 'eastus'           // String with default
param storageSkuName string                // Required parameter
param replicaCount int = 3                 // Int with default
param tags object = {}                     // Object parameter
```

### Variables
```bicep
var resourceNamePrefix = 'app-${environment}-'
var storageName = '${resourceNamePrefix}${uniqueString(resourceGroup().id)}'
var computedValue = deployment().properties.template
```

### Resources
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  // properties...
}
```

### Functions
```bicep
uniqueString(resourceGroup().id)     // Generate unique string
utcNow('yyyy-MM-dd')                 // Current date
resourceGroup().id                    // Get resource group ID
environment().authentication.audiences  // Get Azure environment
```

## Cleanup

Remove resources and resource group:

```powershell
# Delete resource group (removes all resources)
az group delete --name rg-bicep-lab01 --yes

# Verify deletion
az group list --query "[].name" -o table
```

## Summary

In this lab, you:
- Set up VS Code with Bicep tools
- Created your first Bicep template
- Used parameters and variables
- Validated and deployed infrastructure
- Modified parameters for different environments
- Cleaned up resources

## Next Steps

- Review the generated ARM template JSON to understand what Bicep creates
- Explore [Bicep Functions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions)
- Move to Lab 02 to create more complex templates with modules

## Additional Resources

- [Bicep Syntax Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/file)
- [Bicep Best Practices](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/best-practices)
- [Resource Type Reference](https://learn.microsoft.com/en-us/azure/templates/)
- [Azure Bicep Playground](https://aka.ms/bicepdemo)

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
