# Manage Access to Azure Resources

### Objectives

- **Manage built-in Azure roles**
    - Azure uses Role-Based Access Control (RBAC) to manage resource access
    - Built-in roles include Owner, Contributor, Reader, and many specialized roles
    - Owner: Full access to resources, including assigning access to others
    - Contributor: Full access to resources, but cannot assign access
    - Reader: View-only access to resources
    - Custom roles can be created when built-in roles don't meet requirements

- **Assign roles at different scopes**
    - Management Group: Highest level, applies to multiple subscriptions
    - Subscription: Applies to all resource groups and resources within
    - Resource Group: Applies to all resources within the group
    - Resource: Most granular level, applies to specific resource only
    - Roles inherit down the scope hierarchy (management group → subscription → resource group → resource)

- **Interpret access assignments**
    - Effective permissions are the combination of all role assignments
    - Deny assignments override allow permissions
    - Most privileged role takes precedence when multiple roles assigned
    - Azure provides "Access control (IAM)" blade to view and manage assignments
    - "Check access" feature helps verify effective permissions for a user

### Common Commands

```bash
# List all built-in roles
az role definition list --output table

# Get details of a specific built-in role
az role definition list --name "Contributor"

# Assign a role at subscription scope
az role assignment create --role "Reader" --assignee user@example.com --subscription subscription-id

# Assign a role at resource group scope
az role assignment create --role "Contributor" --assignee user@example.com --resource-group resource-group-name

# Assign a role at resource scope
az role assignment create --role "Virtual Machine Contributor" --assignee user@example.com --resource-group resource-group-name --resource-name vm-name --resource-type Microsoft.Compute/virtualMachines

# List role assignments for a user
az role assignment list --assignee user@example.com

# Delete a role assignment
az role assignment delete --assignee user@example.com --role "Contributor" --resource-group resource-group-name
```

```powershell
# List all built-in roles
Get-AzRoleDefinition | Select-Object Name, Description | Sort-Object Name

# Get details of a specific built-in role
Get-AzRoleDefinition -Name "Contributor"

# Assign a role at subscription scope
New-AzRoleAssignment -SignInName user@example.com -RoleDefinitionName "Reader" -Scope "/subscriptions/subscription-id"

# Assign a role at resource group scope
New-AzRoleAssignment -SignInName user@example.com -RoleDefinitionName "Contributor" -ResourceGroupName resource-group-name

# Assign a role at resource scope
$resource = Get-AzResource -ResourceGroupName resource-group-name -Name resource-name
New-AzRoleAssignment -SignInName user@example.com -RoleDefinitionName "Virtual Machine Contributor" -ResourceId $resource.Id

# List role assignments for a user
Get-AzRoleAssignment -SignInName user@example.com

# Delete a role assignment
Remove-AzRoleAssignment -SignInName user@example.com -RoleDefinitionName "Contributor" -ResourceGroupName resource-group-name
```