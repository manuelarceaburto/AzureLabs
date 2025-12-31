# Lab 02: Bicep with Modules - Storage Accounts and Virtual Machines

## Objectives

By completing this lab, you will be able to:
- [ ] Create and use Bicep modules
- [ ] Organize code for reusability
- [ ] Deploy storage accounts with advanced configuration
- [ ] Deploy virtual machines with Bicep
- [ ] Use modules to compose complex solutions
- [ ] Implement parameterized deployments

## Prerequisites

- Completed Lab 01 or Bicep basics knowledge
- VS Code with Bicep extension
- Azure CLI installed
- Active Azure subscription
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation wants to deploy a complete application infrastructure with storage and VMs. You'll use Bicep modules to organize code and deploy reusable components.

## Architecture

```
main.bicep
├── storage.bicep (module)
├── vm.bicep (module)
└── parameters.json

Deployed:
Resource Group
├── Storage Account (secure)
├── Virtual Network
├── Subnet
└── Virtual Machine
```

## Task 1: Create Storage Module

### Instructions

1. **Create storage.bicep module:**
   ```bicep
   // ============================
   // Storage Account Module
   // ============================
   
   param location string
   param environment string
   param projectName string
   param storageAccountType string = 'Standard_LRS'
   param allowBlobPublicAccess bool = false
   param httpsOnly bool = true
   
   // Generate unique storage name
   var storageAccountName = 'st${replace(projectName, '-', '')}${environment}${uniqueString(resourceGroup().id)}'
   
   var tags = {
     environment: environment
     project: projectName
     module: 'storage'
   }
   
   // Create storage account
   resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
     name: storageAccountName
     location: location
     kind: 'StorageV2'
     sku: {
       name: storageAccountType
     }
     properties: {
       accessTier: 'Hot'
       minimumTlsVersion: 'TLS1_2'
       supportsHttpsTrafficOnly: httpsOnly
       allowBlobPublicAccess: allowBlobPublicAccess
     }
     tags: tags
   }
   
   // Create blob container
   resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2021-06-01' = {
     parent: storageAccount
     name: 'default'
   }
   
   resource appContainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-06-01' = {
     parent: blobService
     name: 'app-data'
     properties: {
       publicAccess: 'None'
     }
   }
   
   // Outputs
   output storageAccountId string = storageAccount.id
   output storageAccountName string = storageAccount.name
   output storageAccountKey string = listKeys(storageAccount.id, storageAccount.apiVersion).keys[0].value
   output connectionString string = 'DefaultEndpointsProtocol=https;AccountName=${storageAccount.name};AccountKey=${listKeys(storageAccount.id, storageAccount.apiVersion).keys[0].value};EndpointSuffix=core.windows.net'
   ```

2. **Understand the module:**
   - Accepts parameters (location, environment, etc.)
   - Creates storage account with security settings
   - Creates blob service and container
   - Outputs connection string for use by other modules

### Verification

- [ ] `storage.bicep` file is created
- [ ] Syntax is valid (no errors in VS Code)
- [ ] Can see parameter definitions

## Task 2: Create Network Module

### Instructions

1. **Create network.bicep module:**
   ```bicep
   // ============================
   // Virtual Network Module
   // ============================
   
   param location string
   param environment string
   param projectName string
   param addressSpace string = '10.0.0.0/16'
   param subnetAddressPrefix string = '10.0.1.0/24'
   
   var vnetName = 'vnet-${projectName}-${environment}'
   var subnetName = 'subnet-${environment}'
   var nsgName = 'nsg-${projectName}-${environment}'
   
   var tags = {
     environment: environment
     project: projectName
     module: 'network'
   }
   
   // Create Network Security Group
   resource nsg 'Microsoft.Network/networkSecurityGroups@2021-05-01' = {
     name: nsgName
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
           name: 'AllowRDP'
           properties: {
             priority: 110
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
   
   // Create Virtual Network
   resource vnet 'Microsoft.Network/virtualNetworks@2021-05-01' = {
     name: vnetName
     location: location
     properties: {
       addressSpace: {
         addressPrefixes: [
           addressSpace
         ]
       }
       subnets: [
         {
           name: subnetName
           properties: {
             addressPrefix: subnetAddressPrefix
             networkSecurityGroup: {
               id: nsg.id
             }
           }
         }
       ]
     }
     tags: tags
   }
   
   // Outputs
   output vnetId string = vnet.id
   output vnetName string = vnet.name
   output subnetId string = vnet.properties.subnets[0].id
   output subnetName string = subnetName
   output nsgId string = nsg.id
   ```

### Verification

- [ ] `network.bicep` file is created
- [ ] NSG rules are defined
- [ ] VNet and subnet configuration is correct

## Task 3: Create VM Module

### Instructions

1. **Create vm.bicep module:**
   ```bicep
   // ============================
   // Virtual Machine Module
   // ============================
   
   param location string
   param environment string
   param projectName string
   param vmName string
   param vmSize string = 'Standard_B2s'
   param imagePublisher string = 'MicrosoftWindowsServer'
   param imageOffer string = 'WindowsServer'
   param imageSku string = '2022-datacenter'
   param adminUsername string = 'azureuser'
   @secure()
   param adminPassword string
   param subnetId string
   param storageAccountName string
   
   var nicName = 'nic-${vmName}'
   var osDiskName = 'disk-${vmName}-os'
   
   var tags = {
     environment: environment
     project: projectName
     module: 'compute'
   }
   
   // Create Network Interface
   resource nic 'Microsoft.Network/networkInterfaces@2021-05-01' = {
     name: nicName
     location: location
     properties: {
       ipConfigurations: [
         {
           name: 'ipconfig1'
           properties: {
             privateIPAllocationMethod: 'Dynamic'
             subnet: {
               id: subnetId
             }
             publicIPAddress: {
               id: publicIP.id
             }
           }
         }
       ]
     }
     tags: tags
   }
   
   // Create Public IP
   resource publicIP 'Microsoft.Network/publicIPAddresses@2021-05-01' = {
     name: 'pip-${vmName}'
     location: location
     properties: {
       publicIPAllocationMethod: 'Dynamic'
       dnsSettings: {
         domainNameLabel: toLower(vmName)
       }
     }
     tags: tags
   }
   
   // Create Virtual Machine
   resource vm 'Microsoft.Compute/virtualMachines@2021-07-01' = {
     name: vmName
     location: location
     properties: {
       hardwareProfile: {
         vmSize: vmSize
       }
       osProfile: {
         computerName: vmName
         adminUsername: adminUsername
         adminPassword: adminPassword
       }
       storageProfile: {
         imageReference: {
           publisher: imagePublisher
           offer: imageOffer
           sku: imageSku
           version: 'latest'
         }
         osDisk: {
           name: osDiskName
           caching: 'ReadWrite'
           createOption: 'FromImage'
         }
       }
       networkProfile: {
         networkInterfaces: [
           {
             id: nic.id
           }
         ]
       }
     }
     tags: tags
   }
   
   // Outputs
   output vmId string = vm.id
   output vmName string = vm.name
   output publicIpAddress string = publicIP.properties.ipAddress
   output fqdnAddress string = publicIP.properties.dnsSettings.fqdn
   ```

### Verification

- [ ] `vm.bicep` file is created
- [ ] Includes network interface and public IP
- [ ] Supports parameter-driven configuration

## Task 4: Create Main Template

### Instructions

1. **Create main.bicep that uses modules:**
   ```bicep
   // ============================
   // Main Infrastructure Template
   // ============================
   
   param location string = 'eastus'
   param environment string = 'dev'
   param projectName string = 'contoso'
   param vmAdminPassword string
   
   var deploymentSuffix = uniqueString(resourceGroup().id)
   
   // Deploy Storage Module
   module storageModule 'storage.bicep' = {
     name: 'storage-${deploymentSuffix}'
     params: {
       location: location
       environment: environment
       projectName: projectName
       storageAccountType: 'Standard_LRS'
     }
   }
   
   // Deploy Network Module
   module networkModule 'network.bicep' = {
     name: 'network-${deploymentSuffix}'
     params: {
       location: location
       environment: environment
       projectName: projectName
     }
   }
   
   // Deploy VM Module
   module vmModule 'vm.bicep' = {
     name: 'vm-${deploymentSuffix}'
     params: {
       location: location
       environment: environment
       projectName: projectName
       vmName: 'vm-${projectName}-${environment}'
       vmSize: 'Standard_B2s'
       adminPassword: vmAdminPassword
       subnetId: networkModule.outputs.subnetId
       storageAccountName: storageModule.outputs.storageAccountName
     }
   }
   
   // Outputs from all modules
   output storageConnectionString string = storageModule.outputs.connectionString
   output vmPublicIpAddress string = vmModule.outputs.publicIpAddress
   output vmFqdn string = vmModule.outputs.fqdnAddress
   output vnetId string = networkModule.outputs.vnetId
   ```

### Verification

- [ ] `main.bicep` references all modules correctly
- [ ] Module parameters are passed correctly
- [ ] Outputs are defined for all important values

## Task 5: Create Parameters File

### Instructions

1. **Create parameters.json:**
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
       "vmAdminPassword": {
         "value": "P@ssw0rd123!@#"
       }
     }
   }
   ```

2. **Important:** 
   - Never commit passwords to version control
   - Use Key Vault in production
   - Consider using `@secure()` parameter decorator

### Verification

- [ ] `parameters.json` is created
- [ ] All required parameters are included
- [ ] JSON is valid

## Task 6: Deploy Complete Infrastructure

### Instructions

1. **Validate the deployment:**
   ```powershell
   # Create resource group
   az group create --name rg-bicep-lab02 --location eastus
   
   # Validate
   az deployment group validate `
     --resource-group rg-bicep-lab02 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

2. **Deploy:**
   ```powershell
   az deployment group create `
     --name FullStackDeployment `
     --resource-group rg-bicep-lab02 `
     --template-file main.bicep `
     --parameters parameters.json
   ```

3. **View outputs:**
   ```powershell
   az deployment group show `
     --resource-group rg-bicep-lab02 `
     --name FullStackDeployment `
     --query 'properties.outputs' -o json
   ```

4. **Access deployed resources:**
   ```powershell
   # View all resources
   az resource list --resource-group rg-bicep-lab02 -o table
   
   # Get VM public IP
   az vm list-ip-addresses --resource-group rg-bicep-lab02
   ```

### Verification

- [ ] Deployment completes successfully
- [ ] All modules are deployed
- [ ] Can access VM public IP address
- [ ] Storage account is created
- [ ] Virtual network is created

## Task 7: Explore Module Composition

### Instructions

1. **Understand module dependencies:**
   - Storage module is independent
   - Network module is independent
   - VM module depends on network module
   - Main template orchestrates all

2. **Reuse modules for production:**
   - Copy storage.bicep, network.bicep, vm.bicep to new folder
   - Create new parameters.json for production
   - Deploy to new resource group with `environment: 'prod'`

3. **Test multi-environment deployment:**
   ```powershell
   # Create production version
   $params = Get-Content parameters.json | ConvertFrom-Json
   $params.parameters.environment.value = 'prod'
   $params | ConvertTo-Json | Set-Content parameters-prod.json
   
   # Deploy to production
   az group create --name rg-bicep-lab02-prod --location eastus
   az deployment group create `
     --name FullStackDeployment-Prod `
     --resource-group rg-bicep-lab02-prod `
     --template-file main.bicep `
     --parameters parameters-prod.json
   ```

## Cleanup

```powershell
# Delete resource groups
az group delete --name rg-bicep-lab02 --yes
az group delete --name rg-bicep-lab02-prod --yes
```

## Summary

In this lab, you:
- Created reusable Bicep modules for storage, networking, and compute
- Used modules to compose a complete solution
- Deployed a VM with networking and storage
- Retrieved deployment outputs
- Tested multi-environment deployments

## Key Takeaways

- Modules promote code reuse and organization
- Each module should have a single responsibility
- Outputs from one module can feed into another
- Parameters.json enables environment-specific configurations
- Module composition creates scalable solutions

## Next Steps

- Review the actual deployed resources in Azure Portal
- Experiment with different parameter values
- Move to Lab 03 for advanced patterns (loops, conditions)

## Additional Resources

- [Bicep Modules](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules)
- [Module Best Practices](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/best-practices)
- [Template Specs](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/template-specs)

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
