# Lab 02: Implement RBAC and Custom Roles

## Objectives

By completing this lab, you will be able to:
- [ ] Understand Azure role-based access control (RBAC) fundamentals
- [ ] Assign built-in roles to users and groups
- [ ] Create custom roles with specific permissions
- [ ] Implement principle of least privilege
- [ ] Review and audit role assignments

## Prerequisites

- Active Azure subscription
- Owner or User Access Administrator role on the subscription
- Users or groups from Lab 01 (or create new ones)
- Resource group for testing
- Estimated time: 60 minutes

## Lab Scenario

Contoso Corporation needs to implement proper access control across their Azure resources. The IT Department should manage infrastructure, Marketing should manage specific resources, and Finance should have limited access for auditing. You'll configure RBAC and create a custom role for developers.

## Architecture

```
Subscription
├── Role Assignments
│   ├── Contributor (IT Department)
│   ├── Reader (Finance Team)
│   ├── Storage Account Contributor (Marketing)
│   └── Custom: Developer Role (Developers Group)
├── Resource Groups
│   ├── RG-IT
│   ├── RG-Marketing
│   └── RG-Finance
└── Resources
    ├── Storage Accounts
    ├── Virtual Machines
    └── App Services
```

## Task 1: Assign Built-in Roles

### Instructions

1. Navigate to **Subscriptions** in Azure Portal
2. Select your subscription
3. Go to **Access Control (IAM)** > **Role assignments**
4. Click **+ Add** > **Add role assignment**
5. Assign the following roles:
   - **Contributor** role to IT Department group (Resource Group: RG-IT)
   - **Reader** role to Finance Team group (Resource Group: RG-Finance)
   - **Storage Account Contributor** to Marketing Team (Scope: Storage Account)
   - **Virtual Machine Contributor** to John Smith (Resource Group: RG-IT)

6. For each assignment:
   - Select the role
   - Assign to user/group
   - Set the scope (subscription, resource group, or resource)
   - Review and assign

### Verification

- [ ] All role assignments are visible in IAM
- [ ] Each user/group has appropriate role at correct scope
- [ ] Verify using "Check Access" feature
- [ ] Test access by signing in as assigned users

### PowerShell Example

```powershell
# Get required IDs
$subscription = Get-AzSubscription
$resourceGroup = Get-AzResourceGroup -Name "RG-IT"
$user = Get-AzADUser -SearchString "john.smith"

# Assign Contributor role
New-AzRoleAssignment -SignInName "john.smith@yourdomain.onmicrosoft.com" `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/$($subscription.Id)/resourceGroups/$($resourceGroup.Name)"

# Assign role to group
$group = Get-AzADGroup -DisplayName "IT Department"
New-AzRoleAssignment -ObjectId $group.Id `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/$($subscription.Id)"
```

## Task 2: Create a Custom Role

### Instructions

1. Create a JSON file for the custom role definition:

```json
{
  "Name": "Developer",
  "IsCustom": true,
  "Description": "Custom role for developers with limited permissions",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Storage/storageAccounts/read",
    "Microsoft.Storage/storageAccounts/listKeys/action",
    "Microsoft.Web/sites/read",
    "Microsoft.Web/sites/write"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/delete",
    "Microsoft.Storage/storageAccounts/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

2. Navigate to **Subscriptions** > **Access Control (IAM)** > **Roles**
3. Click **Create custom role** or use PowerShell:

### PowerShell Example

```powershell
# Create custom role from JSON
$subscription = Get-AzSubscription
$role = New-AzRoleDefinition -InputFile "developer-role.json"

# Verify creation
Get-AzRoleDefinition -Custom | Where-Object Name -eq "Developer"
```

3. Assign the custom Developer role to a user or group
4. Review the permissions in the custom role

### Verification

- [ ] Custom Developer role is created
- [ ] Role is assignable
- [ ] Appropriate Actions and NotActions are configured
- [ ] Role appears in Access Control (IAM)
- [ ] Can be assigned to users/groups

## Task 3: Implement Principle of Least Privilege

### Instructions

1. Review current role assignments for excessive permissions
2. Create a more restrictive role for Readers:
   - Can read resources but cannot modify
   - Can view logs and monitoring data
   - Cannot create or delete resources

3. Audit Finance Team permissions:
   - They should have Reader role on subscription
   - No write permissions on critical resources

4. Create a custom "Auditor" role with:
   - Read-only access to all resources
   - Ability to view activity logs
   - Ability to read diagnostic settings

### Custom Auditor Role JSON

```json
{
  "Name": "Auditor",
  "IsCustom": true,
  "Description": "Read-only access for auditing and compliance",
  "Actions": [
    "*/read",
    "Microsoft.Insights/diagnosticSettings/read",
    "Microsoft.Insights/logs/read",
    "Microsoft.OperationalInsights/workspaces/*/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

### Verification

- [ ] Auditor role is created
- [ ] Finance Team is assigned Auditor role
- [ ] Users cannot modify resources with this role
- [ ] Activity logs are accessible

## Task 4: Review and Audit Role Assignments

### Instructions

1. Navigate to **Subscriptions** > **Access Control (IAM)**
2. Review all role assignments:
   - Filter by role type
   - Filter by assigned user/group
   - Check expiration dates if applicable
3. Export role assignments for audit:
   - Use PowerShell to get all assignments
   - Create a report

### PowerShell Example

```powershell
# Get all role assignments
$assignments = Get-AzRoleAssignment -Scope "/subscriptions/$((Get-AzSubscription).Id)"

# Export to CSV
$assignments | Select-Object DisplayName, RoleDefinitionName, Scope | Export-Csv -Path "role-assignments.csv"

# Find users with high-privilege roles
$assignments | Where-Object RoleDefinitionName -eq "Owner" | Format-Table DisplayName, Scope

# Find assignments expiring soon (if using PIM)
Get-AzRoleEligibleUserAssignment | Where-Object ExpirationDate -lt (Get-Date).AddDays(30)
```

4. Identify and remove unnecessary assignments
5. Ensure no users have Owner role at subscription level unnecessarily

### Verification

- [ ] All role assignments are documented
- [ ] No excessive permissions are assigned
- [ ] Audit report is generated
- [ ] Scope of each role is appropriate

## Cleanup

Remove test role assignments and custom roles:

```powershell
# Remove role assignments
$assignment = Get-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Developer"
Remove-AzRoleAssignment -InputObject $assignment

# Remove custom role
Remove-AzRoleDefinition -Id "custom-role-id" -Force
```

Or via Azure CLI:

```bash
# Remove role assignment
az role assignment delete --assignee "user@domain.com" \
    --role "Developer" \
    --scope "/subscriptions/{subscription-id}"

# Remove custom role
az role definition delete --name "Developer"
```

## Review

In this lab, you:
- Assigned built-in Azure roles at appropriate scopes
- Created custom roles with specific permissions
- Implemented principle of least privilege
- Audited and reviewed role assignments

Proper RBAC implementation is critical for security and compliance in Azure.

## Additional Resources

- [Azure RBAC Overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
- [Built-in Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)
- [Create Custom Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles)
- [Role Definitions](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-definitions)
- [Azure Privileged Identity Management (PIM)](https://learn.microsoft.com/en-us/entra/identity/privileged-identity-management/pim-configure)

## Notes for Instructors

- Ensure students understand role assignment scope
- Emphasize security implications of role assignments
- PIM (Privileged Identity Management) requires Entra ID P2 license
- Custom roles are immutable after creation; use versioning
- Document all role assignments for compliance auditing
- Consider time zone differences when setting assignment dates

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
