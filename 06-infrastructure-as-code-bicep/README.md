# Module 06: Infrastructure as Code with Bicep

Infrastructure as Code (IaC) allows you to define Azure resources using code, enabling version control, repeatability, and automation. Bicep is Microsoft's domain-specific language (DSL) for Azure resource deployment.

## What is Bicep?

Bicep is a declarative language for deploying Azure resources. It's simpler and more readable than ARM (Azure Resource Manager) templates JSON while providing the same capabilities.

**Benefits:**
- Cleaner, easier-to-read syntax
- Better reusability with modules
- Version control for infrastructure
- Reproducible deployments
- Infrastructure consistency across environments

## Repository Structure

```
06-infrastructure-as-code-bicep/
├── README.md
├── lab01-bicep-basics/
│   ├── README.md
│   ├── main.bicep
│   ├── parameters.json
│   └── deploy.ps1
├── lab02-bicep-storage-vms/
│   ├── README.md
│   ├── main.bicep
│   ├── storage.bicep (module)
│   ├── vm.bicep (module)
│   ├── parameters.json
│   └── deploy.ps1
└── lab03-bicep-advanced/
    ├── README.md
    ├── main.bicep
    ├── modules/
    │   ├── storage.bicep
    │   ├── networking.bicep
    │   └── compute.bicep
    ├── parameters.json
    └── deploy.ps1
```

## Prerequisites for All Labs

### VS Code Setup

1. **Install VS Code Extensions:**
   - Open VS Code
   - Go to Extensions (Ctrl+Shift+X)
   - Search and install:
     - "Bicep" by Microsoft
     - "Azure Resource Manager (ARM) Tools" by Microsoft
     - "Azure Account" by Microsoft

2. **Install Azure CLI:**
   ```bash
   # Windows
   choco install azure-cli
   
   # Or download from: https://aka.ms/installazurecliwindows
   ```

3. **Install Bicep CLI:**
   ```bash
   az bicep install
   ```

4. **Sign in to Azure:**
   ```bash
   az login
   ```

## Lab Overview

### Lab 01: Bicep Basics
Learn Bicep syntax and create simple resource groups and storage accounts.
- Creating and deploying basic Bicep files
- Using parameters and variables
- Understanding outputs
- Deployment from CLI

### Lab 02: Storage Accounts and VMs
Build on basics with more complex resources using modules.
- Creating Bicep modules for reusability
- Deploying storage accounts with security settings
- Deploying virtual machines with networking
- Parameter files for different environments

### Lab 03: Advanced Bicep Patterns
Implement enterprise patterns and best practices.
- Complex module dependencies
- Conditional deployments
- Loops and arrays
- Secrets management with Key Vault
- Multi-environment deployment

## Quick Start

```bash
# Navigate to lab directory
cd lab01-bicep-basics

# Deploy resources
./deploy.ps1

# Verify deployment
az resource list --resource-group rg-bicep-demo

# Delete resources when done
az group delete --name rg-bicep-demo --yes
```

## Bicep vs ARM Templates

| Feature | Bicep | ARM Template |
|---------|-------|-------------|
| Syntax | Declarative, simple | JSON, verbose |
| Readability | Excellent | Poor |
| Modules | Native | Linked templates |
| Debugging | Better tooling | Complex |
| Learning curve | Gentle | Steep |
| Azure support | Full | Full (older) |

## Key Concepts

### Parameters
Define inputs for your deployment:
```bicep
param location string = 'eastus'
param environment string
param tags object = {
  env: 'dev'
  project: 'contoso'
}
```

### Variables
Create computed values:
```bicep
var storageAccountName = 'st${uniqueString(resourceGroup().id)}'
var vnetName = 'vnet-${environment}'
```

### Resources
Define Azure resources:
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}
```

### Outputs
Return values from deployment:
```bicep
output storageAccountId string = storageAccount.id
output storageAccountName string = storageAccount.name
```

### Modules
Reuse code across projects:
```bicep
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    environment: environment
  }
}
```

## Workflow

1. **Write Bicep file** in VS Code
2. **Create parameters file** for inputs
3. **Validate** with `az bicep build`
4. **Deploy** with `az deployment group create`
5. **Monitor** with Azure Portal or CLI
6. **Destroy** when finished

## Useful Commands

```bash
# Validate Bicep file
az bicep build --file main.bicep

# Validate deployment (no changes)
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters parameters.json

# Deploy
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters parameters.json

# View deployment details
az deployment group show \
  --resource-group myResourceGroup \
  --name myDeployment

# Delete resources
az group delete --name myResourceGroup --yes
```

## Best Practices

- ✅ Use meaningful names for parameters and variables
- ✅ Organize code with modules
- ✅ Use parameter files for different environments
- ✅ Add descriptions to parameters
- ✅ Use outputs to pass values between modules
- ✅ Store Bicep files in version control
- ✅ Validate before deploying
- ✅ Use symbolic names (not GUIDs)
- ✅ Document your modules
- ❌ Don't hardcode environment-specific values
- ❌ Don't skip validation
- ❌ Don't mix multiple concerns in one file

## Resources

- [Bicep Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Bicep Language](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/file)
- [Bicep Functions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions)
- [Bicep Modules](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules)
- [Azure Bicep Playground](https://aka.ms/bicepdemo)

## Next Steps

1. Start with Lab 01 to learn basic syntax
2. Progress to Lab 02 for practical deployments
3. Complete Lab 03 for enterprise patterns
4. Apply Bicep to your own Azure projects

---

Happy learning with Infrastructure as Code!
