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

## Task 1: Set Up VS Code for Bicep

### Instructions

1. **Install VS Code Extensions:**
   - Open VS Code
   - Press `Ctrl+Shift+X` to open Extensions
   - Search for "Bicep" 
   - Install "Bicep" by Microsoft
   - Search for "Azure Resource Manager"
   - Install "Azure Resource Manager (ARM) Tools"
   - Search for "Azure Account"
   - Install "Azure Account" by Microsoft

2. **Install Bicep CLI:**
   ```powershell
   # Open PowerShell as Administrator
   az bicep install
   ```

3. **Sign in to Azure:**
   ```powershell
   az login
   ```
   This opens browser to authenticate

4. **Verify Installation:**
   ```powershell
   # Check Bicep version
   az bicep version
   
   # Should output something like: Bicep CLI version 0.x.x
   ```

5. **Create project folder:**
   ```powershell
   mkdir C:\MyBicepProject
   cd C:\MyBicepProject
   ```

6. **Open in VS Code:**
   ```powershell
   code .
   ```

### Verification

- [ ] All extensions installed in VS Code
- [ ] Bicep CLI is installed
- [ ] Can run `az bicep version`
- [ ] Folder is open in VS Code

## Task 2: Create Your First Bicep File

### Instructions

1. **Create main.bicep file:**
   - In VS Code, right-click in Explorer
   - Select "New File"
   - Name it `main.bicep`

2. **Add basic structure:**
   ```bicep
   // ============================
   // Bicep Template for Storage Account
   // ============================
   
   // Parameters - inputs to the template
   param location string = 'eastus'
   param environment string = 'dev'
   param projectName string = 'contoso'
   
   // Variables - computed values
   var storageAccountName = 'st${projectName}${environment}${uniqueString(resourceGroup().id)}'
   var tags = {
     environment: environment
     project: projectName
     createdDate: utcNow('yyyy-MM-dd')
   }
   
   // Resources
   resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
     name: storageAccountName
     location: location
     kind: 'StorageV2'
     sku: {
       name: 'Standard_LRS'
     }
     properties: {
       accessTier: 'Hot'
       minimumTlsVersion: 'TLS1_2'
       supportsHttpsTrafficOnly: true
     }
     tags: tags
   }
   
   // Outputs
   output storageAccountId string = storageAccount.id
   output storageAccountName string = storageAccount.name
   output storageAccountUri string = storageAccount.properties.primaryEndpoints.web
   ```

3. **Understand the structure:**
   - **Parameters**: Inputs to template (location, environment, etc.)
   - **Variables**: Computed values (storage account name, tags)
   - **Resources**: Azure resources to create
   - **Outputs**: Values returned after deployment

### Verification

- [ ] File is created as `main.bicep`
- [ ] Syntax is highlighted (Bicep extension working)
- [ ] No squiggly error lines
- [ ] Can see resource snippets and autocomplete

## Task 3: Create Parameters File

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
