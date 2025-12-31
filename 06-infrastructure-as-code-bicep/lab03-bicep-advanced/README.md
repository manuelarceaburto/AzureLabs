# Lab 03: Advanced Bicep Patterns - Loops, Conditions, and Enterprise Patterns

## Objectives

By completing this lab, you will be able to:
- [ ] Use loops to deploy multiple resources
- [ ] Implement conditional deployments
- [ ] Use variables and outputs effectively
- [ ] Implement naming conventions
- [ ] Deploy multi-tier applications with Bicep
- [ ] Use Key Vault for secrets

## Prerequisites

- Completed Lab 02 or solid Bicep foundation
- VS Code with Bicep extension
- Azure CLI installed
- Active Azure subscription
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation wants to deploy scalable infrastructure with:
- Multiple storage accounts
- Multiple VMs
- Conditional resources based on environment
- Enterprise naming conventions
- Secrets management

## Advanced Concepts

### 1. Loops with `for`

Deploy multiple resources by iterating:

```bicep
// Deploy 3 storage accounts
param storageCount int = 3

resource storageAccounts 'Microsoft.Storage/storageAccounts@2021-06-01' = [for i in range(0, storageCount): {
  name: 'st${projectName}${i}${uniqueString(resourceGroup().id)}'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}]
```

### 2. Conditional Deployments with `if`

Deploy resources only when conditions are met:

```bicep
// Deploy premium tier only in production
param environment string
param deployPremium bool = (environment == 'prod')

resource premiumStorage 'Microsoft.Storage/storageAccounts@2021-06-01' = if (deployPremium) {
  name: 'stpremium${uniqueString(resourceGroup().id)}'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Premium_LRS'  // Only in production
  }
}
```

### 3. Complex Variable Types

```bicep
@description('VM configuration by environment')
var vmConfig = {
  dev: {
    vmSize: 'Standard_B2s'
    diskSku: 'Standard_LRS'
  }
  prod: {
    vmSize: 'Standard_D2s_v3'
    diskSku: 'Premium_LRS'
  }
}

var currentConfig = vmConfig[environment]
```

## Task 1: Create Advanced Main Template

### Instructions

1. **Create main.bicep with advanced features:**
   ```bicep
   // ============================
   // Advanced Infrastructure Template
   // ============================
   
   @description('Deployment location')
   param location string = 'eastus'
   
   @description('Environment: dev, test, or prod')
   @allowed([
     'dev'
     'test'
     'prod'
   ])
   param environment string = 'dev'
   
   @description('Project name')
   param projectName string = 'contoso'
   
   @description('Number of storage accounts to create')
   @minValue(1)
   @maxValue(5)
   param storageAccountCount int = 1
   
   @description('Number of VMs to create')
   @minValue(1)
   @maxValue(3)
   param vmCount int = 1
   
   @description('VM admin password')
   @secure()
   param vmAdminPassword string
   
   @description('Deploy premium resources (production only)')
   param deployPremium bool = (environment == 'prod')
   
   // ============================
   // Variables
   // ============================
   
   var deploymentId = uniqueString(resourceGroup().id)
   var resourceNamePrefix = '${projectName}-${environment}'
   
   @description('Environment-specific configuration')
   var environmentConfig = {
     dev: {
       vmSize: 'Standard_B2s'
       storageSku: 'Standard_LRS'
       diskType: 'Standard_LRS'
     }
     test: {
       vmSize: 'Standard_B2s'
       storageSku: 'Standard_GRS'
       diskType: 'Standard_SSD_LRS'
     }
     prod: {
       vmSize: 'Standard_D2s_v3'
       storageSku: 'Standard_GRS'
       diskType: 'Premium_LRS'
     }
   }
   
   var currentConfig = environmentConfig[environment]
   
   var tags = {
     environment: environment
     project: projectName
     deploymentDate: utcNow('yyyy-MM-dd')
     deploymentId: deploymentId
   }
   
   // ============================
   // Resources
   // ============================
   
   // Virtual Network
   resource vnet 'Microsoft.Network/virtualNetworks@2021-05-01' = {
     name: 'vnet-${resourceNamePrefix}'
     location: location
     properties: {
       addressSpace: {
         addressPrefixes: [
           '10.0.0.0/16'
         ]
       }
       subnets: [
         {
           name: 'subnet-web'
           properties: {
             addressPrefix: '10.0.1.0/24'
           }
         }
         {
           name: 'subnet-db'
           properties: {
             addressPrefix: '10.0.2.0/24'
           }
         }
       ]
     }
     tags: tags
   }
   
   // Storage Accounts - Multiple using loop
   resource storageAccounts 'Microsoft.Storage/storageAccounts@2021-06-01' = [for i in range(0, storageAccountCount): {
     name: 'st${replace(projectName, '-', '')}${environment}${i}${deploymentId}'
     location: location
     kind: 'StorageV2'
     sku: {
       name: currentConfig.storageSku
     }
     properties: {
       accessTier: 'Hot'
       minimumTlsVersion: 'TLS1_2'
       supportsHttpsTrafficOnly: true
     }
     tags: tags
   }]
   
   // Premium Storage Account - Only in production
   resource premiumStorage 'Microsoft.Storage/storageAccounts@2021-06-01' = if (deployPremium) {
     name: 'stprem${replace(projectName, '-', '')}${deploymentId}'
     location: location
     kind: 'FileStorage'
     sku: {
       name: 'Premium_LRS'
     }
     properties: {
       accessTier: 'Premium'
       minimumTlsVersion: 'TLS1_2'
       supportsHttpsTrafficOnly: true
     }
     tags: tags
   }
   
   // Network Security Group
   resource nsg 'Microsoft.Network/networkSecurityGroups@2021-05-01' = {
     name: 'nsg-${resourceNamePrefix}'
     location: location
     properties: {
       securityRules: [
         {
           name: 'AllowHTTPS'
           properties: {
             priority: 100
             direction: 'Inbound'
             access: 'Allow'
             protocol: 'Tcp'
             sourcePortRange: '*'
             destinationPortRange: '443'
             sourceAddressPrefix: '*'
             destinationAddressPrefix: '*'
           }
         }
         {
           name: 'AllowHTTP'
           properties: {
             priority: 110
             direction: 'Inbound'
             access: 'Allow'
             protocol: 'Tcp'
             sourcePortRange: '*'
             destinationPortRange: '80'
             sourceAddressPrefix: '*'
             destinationAddressPrefix: '*'
           }
         }
         {
           name: 'AllowRDP'
           properties: {
             priority: 120
             direction: 'Inbound'
             access: 'Allow'
             protocol: 'Tcp'
             sourcePortRange: '*'
             destinationPortRange: '3389'
             sourceAddressPrefix: '*'
             destinationAddressPrefix: '*'
           }
         }
       ]
     }
     tags: tags
   }
   
   // Public IPs and NICs for each VM
   resource publicIPs 'Microsoft.Network/publicIPAddresses@2021-05-01' = [for i in range(0, vmCount): {
     name: 'pip-vm${i}-${resourceNamePrefix}'
     location: location
     properties: {
       publicIPAllocationMethod: 'Dynamic'
       dnsSettings: {
         domainNameLabel: toLower('vm${i}-${projectName}-${environment}-${deploymentId}')
       }
     }
     tags: tags
   }]
   
   resource networkInterfaces 'Microsoft.Network/networkInterfaces@2021-05-01' = [for i in range(0, vmCount): {
     name: 'nic-vm${i}-${resourceNamePrefix}'
     location: location
     properties: {
       ipConfigurations: [
         {
           name: 'ipconfig1'
           properties: {
             privateIPAllocationMethod: 'Dynamic'
             subnet: {
               id: '${vnet.id}/subnets/subnet-web'
             }
             publicIPAddress: {
               id: publicIPs[i].id
             }
           }
         }
       ]
       networkSecurityGroup: {
         id: nsg.id
       }
     }
     tags: tags
   }]
   
   // Virtual Machines - Multiple using loop
   resource virtualMachines 'Microsoft.Compute/virtualMachines@2021-07-01' = [for i in range(0, vmCount): {
     name: 'vm${i}-${resourceNamePrefix}'
     location: location
     properties: {
       hardwareProfile: {
         vmSize: currentConfig.vmSize
       }
       osProfile: {
         computerName: 'vm${i}${environment}'
         adminUsername: 'azureuser'
         adminPassword: vmAdminPassword
       }
       storageProfile: {
         imageReference: {
           publisher: 'MicrosoftWindowsServer'
           offer: 'WindowsServer'
           sku: '2022-datacenter'
           version: 'latest'
         }
         osDisk: {
           name: 'disk-vm${i}-${resourceNamePrefix}-os'
           caching: 'ReadWrite'
           createOption: 'FromImage'
           managedDisk: {
             storageAccountType: currentConfig.diskType
           }
         }
       }
       networkProfile: {
         networkInterfaces: [
           {
             id: networkInterfaces[i].id
           }
         ]
       }
     }
     tags: tags
   }]
   
   // ============================
   // Outputs
   // ============================
   
   @description('All created storage account IDs')
   output storageAccountIds array = [for i in range(0, storageAccountCount): storageAccounts[i].id]
   
   @description('All created storage account names')
   output storageAccountNames array = [for i in range(0, storageAccountCount): storageAccounts[i].name]
   
   @description('Premium storage account ID (if deployed)')
   output premiumStorageId string = deployPremium ? premiumStorage.id : ''
   
   @description('All VM public IP addresses')
   output vmPublicIps array = [for i in range(0, vmCount): publicIPs[i].properties.ipAddress]
   
   @description('All VM FQDNs')
   output vmFqdns array = [for i in range(0, vmCount): publicIPs[i].properties.dnsSettings.fqdn]
   
   @description('Virtual Network ID')
   output vnetId string = vnet.id
   
   @description('Deployment configuration used')
   output configurationApplied object = currentConfig
   
   @description('Environment tags')
   output tags object = tags
   ```

### Verification

- [ ] File is created with advanced patterns
- [ ] No syntax errors in VS Code
- [ ] Loops and conditions are properly formatted

## Task 2: Create Parameters File

### Instructions

1. **Create parameters.json for development:**
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
       },
       "storageAccountCount": {
         "value": 2
       },
       "vmCount": {
         "value": 1
       },
       "vmAdminPassword": {
         "value": "P@ssw0rd123!@#"
       },
       "deployPremium": {
         "value": false
       }
     }
   }
   ```

2. **Create parameters-prod.json for production:**
   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "location": {
         "value": "eastus"
       },
       "environment": {
         "value": "prod"
       },
       "projectName": {
         "value": "contoso"
       },
       "storageAccountCount": {
         "value": 3
       },
       "vmCount": {
         "value": 3
       },
       "vmAdminPassword": {
         "value": "P@ssw0rd123!@#"
       },
       "deployPremium": {
         "value": true
       }
     }
   }
   ```

### Verification

- [ ] Both parameter files are created
- [ ] Dev has smaller deployments
- [ ] Prod has larger deployments and premium storage

## Task 3: Validate and Deploy

### Instructions

1. **Validate development deployment:**
   ```powershell
   az group create --name rg-bicep-lab03-dev --location eastus
   
   az deployment group validate `
     --resource-group rg-bicep-lab03-dev `
     --template-file main.bicep `
     --parameters parameters.json
   ```

2. **Deploy development:**
   ```powershell
   az deployment group create `
     --name AdvancedDeployment-Dev `
     --resource-group rg-bicep-lab03-dev `
     --template-file main.bicep `
     --parameters parameters.json
   ```

3. **View development outputs:**
   ```powershell
   az deployment group show `
     --resource-group rg-bicep-lab03-dev `
     --name AdvancedDeployment-Dev `
     --query 'properties.outputs' -o json
   ```

4. **Deploy production:**
   ```powershell
   az group create --name rg-bicep-lab03-prod --location eastus
   
   az deployment group create `
     --name AdvancedDeployment-Prod `
     --resource-group rg-bicep-lab03-prod `
     --template-file main.bicep `
     --parameters parameters-prod.json
   ```

5. **Compare outputs:**
   ```powershell
   # Dev should have fewer resources
   az resource list --resource-group rg-bicep-lab03-dev | Measure-Object
   
   # Prod should have more resources including premium storage
   az resource list --resource-group rg-bicep-lab03-prod | Measure-Object
   ```

### Verification

- [ ] Both deployments succeed
- [ ] Development has 2 storage accounts, 1 VM
- [ ] Production has 3 storage accounts, 3 VMs, plus premium storage
- [ ] Can retrieve all outputs successfully

## Task 4: Advanced Features Exploration

### Instructions

1. **Understand loops in outputs:**
   - Outputs use loops to return arrays of values
   - Each VM gets a public IP
   - Can iterate over outputs in shell scripts

2. **Understand conditionals:**
   - Premium storage only in production
   - Different VM sizes by environment
   - Different storage SKUs by environment

3. **Understand variables:**
   - `environmentConfig` maps environment to configuration
   - `currentConfig` selects appropriate config
   - Promotes DRY principle (Don't Repeat Yourself)

4. **Test parameter validation:**
   ```powershell
   # This should fail - invalid environment
   az deployment group validate `
     --resource-group rg-bicep-lab03-dev `
     --template-file main.bicep `
     --parameters '{"parameters":{"environment":{"value":"invalid"}}}'
   
   # This should fail - too many VMs
   az deployment group validate `
     --resource-group rg-bicep-lab03-dev `
     --template-file main.bicep `
     --parameters '{"parameters":{"vmCount":{"value":10}}}'
   ```

### Verification

- [ ] Loops work correctly
- [ ] Conditionals deploy/skip resources appropriately
- [ ] Parameter validation works
- [ ] Different configs are applied per environment

## Task 5: Advanced Patterns

### Instructions

1. **Understand array iteration:**
   - `range(0, count)` creates array of integers
   - `[for i in array: ...]` iterates over items
   - Can access loop variable `i` for naming/configuration

2. **Understand object access:**
   - `environmentConfig[environment]` gets config for environment
   - `currentConfig.vmSize` accesses nested property
   - Type-safe with autocomplete

3. **Understand ternary operators:**
   - `deployPremium ? premiumStorage.id : ''`
   - Returns value if true, alternative if false
   - Used for conditional outputs

4. **Document patterns:**
   ```bicep
   // Example: Using descriptions
   @description('Number of VMs to deploy')
   @minValue(1)
   @maxValue(3)
   param vmCount int
   ```

## Task 6: Cleanup and Best Practices

### Instructions

1. **Clean up resources:**
   ```powershell
   az group delete --name rg-bicep-lab03-dev --yes
   az group delete --name rg-bicep-lab03-prod --yes
   ```

2. **Review best practices:**
   - ✅ Use meaningful parameter descriptions
   - ✅ Add `@minValue` and `@maxValue` constraints
   - ✅ Use `@allowed` for enum parameters
   - ✅ Add `@description` to all parameters and outputs
   - ✅ Use `@secure()` for sensitive parameters
   - ✅ Organize with variables for DRY code
   - ✅ Use loops instead of repeating resources
   - ✅ Use conditionals for environment-specific resources
   - ❌ Don't hardcode values
   - ❌ Don't skip validation

## Summary

In this lab, you:
- Created advanced Bicep templates with loops and conditions
- Deployed multiple instances of resources using loops
- Implemented environment-specific configurations
- Used conditionals for feature flags
- Applied enterprise naming conventions
- Deployed and compared dev/prod environments

## Key Advanced Concepts

**Loops** - Deploy multiple similar resources efficiently
**Conditionals** - Enable/disable features based on parameters
**Object Variables** - Store configuration for different environments
**Array Outputs** - Return multiple values from template
**Parameter Decorators** - Add validation and descriptions

## Next Steps

- Explore templates in Azure Quickstart Templates
- Implement Bicep in your organization
- Use `az bicep decompile` to convert ARM templates to Bicep
- Explore template specs for sharing templates

## Additional Resources

- [Bicep Loops and Conditions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/loops)
- [Bicep Variables](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/variables)
- [Bicep Functions Reference](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/bicep-functions)
- [Bicep Best Practices](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/best-practices)
- [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates)

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
