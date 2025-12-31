# Lab 01: Manage Entra ID Users and Groups

## Objectives

By completing this lab, you will be able to:
- [ ] Create and manage user accounts in Entra ID
- [ ] Create and configure security groups
- [ ] Configure group membership and dynamic groups
- [ ] Implement user and group naming conventions
- [ ] Manage user properties and profiles

## Prerequisites

- Active Azure subscription
- Entra ID P1 or higher (for some features) or free tier
- Global Administrator role or User Administrator role
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation is expanding its workforce and needs to implement a structured user and group management strategy in Entra ID. You need to create user accounts for new employees, organize them into groups, and configure group-based access policies.

## Architecture

```
Entra ID
├── Users
│   ├── User1 (user@contoso.com)
│   ├── User2 (user2@contoso.com)
│   └── User3 (user3@contoso.com)
├── Security Groups
│   ├── IT Department
│   ├── Marketing Team
│   └── Finance Team
└── Microsoft 365 Groups
    ├── Project-Alpha
    └── Sales-Team
```

## Task 1: Create User Accounts

### Instructions

1. Open the [Azure Portal](https://portal.azure.com)
2. Navigate to **Entra ID** > **Users** > **New user**
3. Create the following users:
   - **User 1**: John Smith (john.smith@yourdomain.onmicrosoft.com)
   - **User 2**: Sarah Johnson (sarah.johnson@yourdomain.onmicrosoft.com)
   - **User 3**: Mike Davis (mike.davis@yourdomain.onmicrosoft.com)
4. For each user, set:
   - Display name
   - User principal name
   - Initial password (or auto-generate)
   - Job title
   - Department
   - Office location
5. Note the temporary password for each user

### Verification

- [ ] All three users are created and visible in Entra ID Users list
- [ ] Each user has correct UPN format
- [ ] User properties are properly filled out
- [ ] Verify by navigating to **Entra ID** > **Users** and searching for each user

### PowerShell Example

```powershell
# Create a new user using PowerShell
$PasswordProfile = @{
    Password = "TempPassword123!@#"
}

New-MgUser -DisplayName "John Smith" `
    -MailNickname "john.smith" `
    -UserPrincipalName "john.smith@yourdomain.onmicrosoft.com" `
    -PasswordProfile $PasswordProfile `
    -AccountEnabled $true
```

## Task 2: Create and Configure Security Groups

### Instructions

1. Navigate to **Entra ID** > **Groups** > **New group**
2. Create the following security groups:
   - **IT Department**
   - **Marketing Team**
   - **Finance Team**
3. For each group, configure:
   - Group name
   - Group description
   - Group type: Security
   - Membership type: Assigned
4. Add users to groups:
   - IT Department: John Smith, Sarah Johnson
   - Marketing Team: Sarah Johnson
   - Finance Team: Mike Davis

### Verification

- [ ] All three groups are created
- [ ] Groups have appropriate descriptions
- [ ] Users are correctly assigned to groups
- [ ] Group membership is visible in each group's Members section

### Azure CLI Example

```bash
# Create a security group
az ad group create --display-name "IT Department" \
    --mail-nickname "itdepartment" \
    --description "IT Department Security Group"

# Add member to group (requires object IDs)
az ad group member add --group "IT Department" \
    --member-id "user-object-id"
```

## Task 3: Configure Dynamic Groups

### Instructions

1. Navigate to **Entra ID** > **Groups** > **New group**
2. Create a dynamic group named **Department-Based-Group**
3. Set group type to **Security** and membership type to **Dynamic User**
4. Configure membership rule using rule builder:
   - Add rule: `(user.department -eq "IT")`
5. Review the dynamic membership results
6. Create another dynamic group for users with job title containing "Manager":
   - Rule: `(user.jobTitle -contains "Manager")`

### Verification

- [ ] Dynamic groups are created successfully
- [ ] Users matching the criteria are automatically included
- [ ] Membership rule is valid and processing correctly
- [ ] Test by checking group membership in each dynamic group

## Task 4: Manage User Properties and Bulk Operations

### Instructions

1. Update user profiles:
   - Open each user in **Entra ID** > **Users**
   - Add profile photo (download sample images)
   - Update contact information
   - Set manager relationships
2. Test bulk operations:
   - Go to **Entra ID** > **Users** > **Bulk operations** > **Bulk invite**
   - Download the template CSV
   - Create a CSV with new user details
   - Upload and process bulk user creation

### Verification

- [ ] User profiles have photos and complete information
- [ ] Manager relationships are properly configured
- [ ] Bulk import processes without errors
- [ ] New users from bulk operation appear in user list

### PowerShell Example

```powershell
# Update user properties
$userId = "user-object-id"
$body = @{
    jobTitle = "Senior Developer"
    department = "IT"
    officeLocation = "Seattle"
}

Update-MgUser -UserId $userId -BodyParameter $body
```

## Cleanup

Delete the resources created in this lab to avoid unnecessary charges:

```powershell
# Remove users
Remove-MgUser -UserId "john.smith-object-id"
Remove-MgUser -UserId "sarah.johnson-object-id"
Remove-MgUser -UserId "mike.davis-object-id"

# Remove groups
Remove-MgGroup -GroupId "group-object-id"
```

Or via Azure CLI:

```bash
# Remove users
az ad user delete --id "john.smith@yourdomain.onmicrosoft.com"

# Remove groups
az ad group delete --group "IT Department"
```

## Review

In this lab, you:
- Created user accounts with proper UPN naming conventions
- Organized users into security groups
- Configured dynamic group membership based on attributes
- Managed user properties and bulk operations

These skills form the foundation for identity management in Azure and are essential for the AZ104 exam.

## Additional Resources

- [Entra ID User Management](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-user-accounts-azure-portal)
- [Create Groups in Entra ID](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-groups)
- [Dynamic Membership Rules](https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership)
- [Bulk User Management](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-bulk-upload-users)

## Notes for Instructors

- Users can use the Azure Portal, PowerShell, or Azure CLI
- Entra ID free tier supports basic user and group management
- Dynamic groups require Entra ID P1 or P2 license
- When deleting test users, note that they may be soft-deleted for 30 days
- For bulk operations, provide pre-formatted CSV templates to save time

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
