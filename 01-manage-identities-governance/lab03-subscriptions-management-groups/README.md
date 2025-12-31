# Lab 03: Manage Subscriptions and Management Groups

## Objectives

By completing this lab, you will be able to:
- [ ] Understand subscription and management group hierarchy
- [ ] Create and organize management groups
- [ ] Apply policies at management group scope
- [ ] Manage multiple subscriptions
- [ ] Implement cost management and governance

## Prerequisites

- Active Azure subscription with multiple subscriptions (or ability to create new ones)
- Owner or Root Management Group Administrator role
- Entra ID tenant access
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation has multiple Azure subscriptions for different departments (IT, Marketing, Finance). They need to organize these subscriptions using management groups and apply consistent governance policies across all subscriptions.

## Architecture

```
Root Management Group (Contoso)
├── Management Group: Production
│   ├── Subscription: IT-Prod
│   ├── Subscription: Marketing-Prod
│   └── Subscription: Finance-Prod
├── Management Group: Development
│   ├── Subscription: IT-Dev
│   └── Subscription: Marketing-Dev
└── Management Group: Sandbox
    └── Subscription: Sandbox
```

## Task 1: Create Management Group Hierarchy

### Instructions

1. Navigate to **Management Groups** in Azure Portal
2. Note the root management group for your tenant
3. Create the following management groups:
   - **Production** (parent: root)
   - **Development** (parent: root)
   - **Sandbox** (parent: root)
   - **IT-Resources** (parent: Production)
   - **Marketing-Resources** (parent: Production)
   - **Finance-Resources** (parent: Production)

4. For each management group:
   - Set display name
   - Set ID (unique within tenant)
   - Assign to parent
   - Add description

### Verification

- [ ] All management groups are created
- [ ] Hierarchy is correct in the visualization
- [ ] Each group has a unique ID
- [ ] Parent-child relationships are established

### PowerShell Example

```powershell
# Get root management group
$root = Get-AzManagementGroup

# Create management group
New-AzManagementGroup -GroupName "Production" `
    -DisplayName "Production Resources" `
    -ParentId $root.Id

# Create child management group
New-AzManagementGroup -GroupName "IT-Resources" `
    -DisplayName "IT Production Resources" `
    -ParentId "/providers/Microsoft.Management/managementGroups/Production"

# View hierarchy
Get-AzManagementGroup -Recurse
```

## Task 2: Move Subscriptions to Management Groups

### Instructions

1. Navigate to **Management Groups**
2. Identify your subscriptions
3. Move subscriptions to appropriate management groups:
   - If you have multiple subscriptions, assign them to different groups
   - If you have only one, assign it to Production > IT-Resources (for example)

4. Verify subscription placement:
   - Each subscription should appear under its assigned management group
   - Subscriptions should be removed from default location

### Verification

- [ ] All subscriptions are assigned to management groups
- [ ] Subscriptions appear in correct hierarchy level
- [ ] Can navigate subscriptions through management group tree
- [ ] Azure Portal shows subscription under correct management group

### PowerShell Example

```powershell
# Get subscription ID
$subscription = Get-AzSubscription

# Move subscription to management group
New-AzManagementGroupSubscription -GroupName "IT-Resources" `
    -SubscriptionId $subscription.Id

# Verify subscription placement
Get-AzManagementGroup -GroupName "IT-Resources" -Expand
```

## Task 3: Apply Policies to Management Groups

### Instructions

1. Navigate to **Policy** in Azure Portal
2. Create an assignment at management group scope:
   - Go to **Definitions** and select a built-in policy
   - Example: "Allowed locations"

3. Create assignment:
   - Select policy: "Allowed locations"
   - Scope: Management Group "Production"
   - Parameters: Select 1-2 regions (e.g., East US, West US)
   - Effect: Deny
   - Enforcement: Enabled

4. Create another policy:
   - Policy: "Require tag and its value"
   - Tag name: "Environment"
   - Tag value: "Production"
   - Scope: Management Group "Production"

5. Test the policies:
   - Try creating a resource in non-allowed location
   - Verify policy enforcement

### Verification

- [ ] Policies are assigned to management groups
- [ ] Policies appear in Policy > Assignments
- [ ] Policy rules are correctly configured
- [ ] Resources are evaluated against policies
- [ ] Compliance report shows policy status

### PowerShell Example

```powershell
# Create policy assignment
$policy = Get-AzPolicyDefinition -Name "Allowed locations"
$scope = "/providers/Microsoft.Management/managementGroups/Production"

$assignment = New-AzPolicyAssignment -Name "AllowedLocations" `
    -PolicyDefinition $policy `
    -Scope $scope `
    -PolicyParameter @{ "listOfAllowedLocations"=@("eastus","westus") }

# Check policy compliance
Get-AzPolicyState -ManagementGroupName "Production" | Format-Table
```

## Task 4: Implement Cost Management

### Instructions

1. Navigate to **Cost Management + Billing**
2. Create budgets:
   - Budget Name: "Production-Monthly"
   - Amount: $5,000 USD
   - Scope: Management Group "Production"
   - Time period: Monthly
   - Start date: Current month

3. Set alert thresholds:
   - Alert at 50% of budget
   - Alert at 75% of budget
   - Alert at 100% of budget
   - Add email address for alerts

4. Create another budget for Development:
   - Amount: $1,000 USD
   - Scope: Management Group "Development"

5. Review cost analysis:
   - View costs by management group
   - View costs by subscription
   - View costs by resource group

### Verification

- [ ] Budgets are created for management groups
- [ ] Alert thresholds are configured
- [ ] Email notifications are set
- [ ] Cost Analysis shows spending by management group
- [ ] Can drill down from management group to subscription level

## Task 5: Assign Roles at Management Group Scope

### Instructions

1. Navigate to **Management Groups**
2. Select a management group (e.g., "Production")
3. Go to **Access Control (IAM)**
4. Assign roles:
   - **Management Group Contributor** to IT Department group
   - **Reader** to Finance Team group
   - **Cost Management Reader** to Finance Team
   - **Policy Contributor** to IT Department

5. Verify role inheritance:
   - Roles assigned at parent should apply to child subscriptions
   - Test access at different levels

### Verification

- [ ] Roles are assigned at management group scope
- [ ] Users inherit roles from management groups
- [ ] Role-based access works across subscriptions
- [ ] Finance team can view costs
- [ ] IT team can manage policies

### PowerShell Example

```powershell
# Assign role at management group scope
$scope = "/providers/Microsoft.Management/managementGroups/Production"
$group = Get-AzADGroup -DisplayName "IT Department"

New-AzRoleAssignment -ObjectId $group.Id `
    -RoleDefinitionName "Management Group Contributor" `
    -Scope $scope

# Verify assignment
Get-AzRoleAssignment -Scope $scope | Format-Table
```

## Cleanup

Remove management groups and assignments (note: subscriptions must be moved out first):

```powershell
# Remove management group (must be empty of subscriptions)
Remove-AzManagementGroup -GroupName "IT-Resources"

# Remove policy assignment
Remove-AzPolicyAssignment -Name "AllowedLocations" -Scope $scope

# Remove role assignment
Remove-AzRoleAssignment -InputObject $assignment
```

## Review

In this lab, you:
- Created a management group hierarchy for organizational structure
- Moved subscriptions to management groups
- Applied governance policies at management group scope
- Implemented cost management and budgeting
- Assigned roles across multiple subscriptions

This hierarchical approach enables consistent governance and cost management across large Azure deployments.

## Additional Resources

- [Management Groups Overview](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview)
- [Azure Policy Overview](https://learn.microsoft.com/en-us/azure/governance/policy/overview)
- [Cost Management + Billing](https://learn.microsoft.com/en-us/azure/cost-management-billing/)
- [Budgets and Alerts](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets)
- [Governance Best Practices](https://learn.microsoft.com/en-us/azure/governance/governance-in-azure)

## Notes for Instructors

- Management groups are free and built into all Azure subscriptions
- Root management group cannot be deleted
- Policy effects can be "Audit", "Deny", or "Append"
- Compliance data may take 30 minutes to populate
- Budget alerts are sent 1x per day at a specific time
- When moving subscriptions, there's no downtime
- Policy enforcement can be disabled for testing
- Consider creating test management groups that don't affect production

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
