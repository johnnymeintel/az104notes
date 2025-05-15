# Manage Azure Subscriptions and Governance

### Objectives

- **Implement and manage Azure Policy**
    - Azure Policy evaluates resources against defined rules
    - Policy definitions describe resource compliance conditions
    - Policy initiatives are groupings of policy definitions
    - Policies can audit, deny, deploy, modify, or remediate resources
    - Policy effects: Audit, Deny, Append, Deploy, Modify, AuditIfNotExists, DeployIfNotExists
    - Assignments apply policies to specific scopes (management group, subscription, resource group)

- **Configure resource locks**
    - Prevent accidental deletion or modification of resources
    - Lock types: CanNotDelete (Read/Modify, but no delete) and ReadOnly (Read only)
    - Locks apply regardless of RBAC permissions
    - Locks can be applied at subscription, resource group, or resource level
    - Inherited down the scope hierarchy

- **Apply and manage tags on resources**
    - Name-value pairs for organizing resources and managing costs
    - Use cases: Billing/cost management, operational context, security, governance
    - Can be applied at resource group and resource level (not subscription)
    - Maximum 50 tag name/value pairs per resource
    - Tag policy can enforce tagging standards

- **Manage resource groups**
    - Logical containers for resources that share same lifecycle
    - Resources can only exist in one resource group
    - Resource groups can't be nested
    - Operations on resource group affect all resources within
    - Resource group location only stores metadata

- **Manage subscriptions**
    - Container for Azure resources and billing boundary
    - Subscription types: Free, Pay-As-You-Go, Enterprise Agreement, CSP
    - Associate subscriptions with different Microsoft Entra tenants
    - Transfer subscription ownership
    - Control resource creation with subscription-level policies

- **Manage costs by using alerts, budgets, and Azure Advisor recommendations**
    - Cost alerts notify when spending reaches threshold
    - Budgets set spending limits for resources/subscriptions
    - Azure Advisor provides recommendations to optimize costs
    - Cost Management + Billing provides cost analysis tools
    - Reserved Instances and Azure Hybrid Benefit reduce costs

- **Configure management groups**
    - Organize subscriptions into hierarchical containers
    - Apply governance conditions to multiple subscriptions
    - Policy and RBAC inherit down the hierarchy
    - Maximum six levels of depth (root level + 5 levels of management groups)
    - 10,000 management groups supported in single directory

### Common Commands

```bash
# Create a policy assignment
az policy assignment create --policy "policy-definition-name" --name "assignment-name" --scope "/subscriptions/{subscriptionId}"

# Create a resource lock
az lock create --name "LockName" --resource-group "ResourceGroupName" --lock-type CanNotDelete

# Tag a resource
az resource tag --tags "Dept=Finance" "Environment=Production" --resource-group "ResourceGroupName" --name "ResourceName" --resource-type "Microsoft.Web/sites"

# Create a resource group
az group create --name "ResourceGroupName" --location "EastUS"

# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "SubscriptionName"

# Create a budget
az consumption budget create --budget-name "MonthlyBudget" --amount 1000 --time-grain "Monthly" --scope "subscriptions/{subscriptionId}"

# Create a management group
az account management-group create --name "ManagementGroupName" --display-name "Management Group Display Name"
```

```powershell
# Create a policy assignment
New-AzPolicyAssignment -Name "assignment-name" -PolicyDefinition (Get-AzPolicyDefinition -Name "policy-definition-name") -Scope "/subscriptions/{subscriptionId}"

# Create a resource lock
New-AzResourceLock -LockName "LockName" -LockLevel CanNotDelete -ResourceGroupName "ResourceGroupName"

# Tag a resource
$tags = @{"Dept"="Finance"; "Environment"="Production"}
Set-AzResource -ResourceGroupName "ResourceGroupName" -Name "ResourceName" -ResourceType "Microsoft.Web/sites" -Tag $tags -Force

# Create a resource group
New-AzResourceGroup -Name "ResourceGroupName" -Location "EastUS"

# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -Subscription "SubscriptionName"

# Create a budget
New-AzConsumptionBudget -Name "MonthlyBudget" -Amount 1000 -Category Cost -TimeGrain Monthly -StartDate (Get-Date) -EndDate (Get-Date).AddYears(1) -ResourceGroupName "ResourceGroupName"

# Create a management group
New-AzManagementGroup -GroupName "ManagementGroupName" -DisplayName "Management Group Display Name"
```