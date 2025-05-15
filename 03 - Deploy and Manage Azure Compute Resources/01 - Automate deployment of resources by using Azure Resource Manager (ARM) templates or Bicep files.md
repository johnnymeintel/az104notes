# Automate deployment of resources by using Azure Resource Manager (ARM) templates or Bicep files

### Objectives

- **Interpret an Azure Resource Manager template or a Bicep file**
    - **ARM Template Structure**:
        - schema: Version of template language
        - contentVersion: Version of your template
        - parameters: Input values for deployment
        - variables: Values constructed from parameters
        - resources: Azure resources to deploy
        - outputs: Values returned after deployment
    - **Bicep File Structure**:
        - param: Parameter declarations
        - var: Variable declarations
        - resource: Resource declarations
        - output: Output declarations
        - module: Reference to other Bicep files
    - **Understanding Dependencies**:
        - Implicit (Bicep): Automatic when referencing properties
        - Explicit (ARM): Using dependsOn array
    - **Resource Types**: Microsoft.Compute/virtualMachines, Microsoft.Storage/storageAccounts, etc.
    - **API Versions**: Each resource type has specific API versions

- **Modify an existing Azure Resource Manager template**
    - **Adding Parameters**: Define inputs for reusability
        - type: string, int, bool, object, array, secureString
        - defaultValue: Optional default if not provided
        - allowedValues: Restrict input to specific values
        - description: Documentation for parameter
    - **Adding Variables**: Compute values from parameters
        - Concatenation using concat() function
        - Conditional logic using if() function
    - **Modifying Resources**:
        - Change properties like size, SKU, location
        - Add tags for organization
        - Update dependencies
    - **Adding Outputs**: Return deployment information
        - Resource IDs, connection strings, endpoints

- **Modify an existing Bicep file**
    - **Adding Parameters**: Cleaner syntax than ARM
    - **Using Variables**: Simplified syntax
    - **Resource Modifications**:
        - Direct property access with dot notation
        - Simplified dependency management
        - String interpolation for dynamic values
    - **Adding Modules**: Reusable Bicep components

- **Deploy resources by using an Azure Resource Manager template or a Bicep file**
    - **Deployment Scopes**:
        - Resource Group (most common)
        - Subscription
        - Management Group
        - Tenant
    - **Deployment Modes**:
        - Incremental (default): Adds/updates resources
        - Complete: Deletes resources not in template
    - **Parameter Files**: Separate JSON files for different environments
    - **What-If Operations**: Preview changes before deployment
    - **Validation**: Check template syntax before deployment
    - **Deployment History**: Track all deployments in resource group

- **Export a deployment as an Azure Resource Manager template or convert an Azure Resource Manager template to a Bicep file**
    - **Export Options**:
        - From deployment history
        - From resource group
        - From individual resources
    - **Export Limitations**:
        - Some properties not exported
        - Passwords/secrets excluded
        - May need manual cleanup
    - **ARM to Bicep Conversion**:
        - Use bicep decompile command
        - Review and fix warnings
        - Optimize generated code
    - **Bicep to ARM**:
        - Use bicep build command
        - Automatic during deployment

### Common Commands

```bash
# Azure CLI Commands

# Validate ARM template
az deployment group validate \
  --resource-group MyRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# Deploy ARM template
az deployment group create \
  --resource-group MyRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# Deploy Bicep file
az deployment group create \
  --resource-group MyRG \
  --template-file main.bicep \
  --parameters location=eastus vmSize=Standard_DS2_v2

# What-if deployment
az deployment group what-if \
  --resource-group MyRG \
  --template-file main.bicep \
  --parameters @parameters.json

# Export deployment template
az deployment group export \
  --resource-group MyRG \
  --name MyDeployment > exported-template.json

# Export resource group template
az group export \
  --resource-group MyRG > exported-rg-template.json

# Convert ARM to Bicep
az bicep decompile --file azuredeploy.json

# Convert Bicep to ARM  
az bicep build --file main.bicep

# Install/Upgrade Bicep
az bicep install
az bicep upgrade

# List deployments
az deployment group list --resource-group MyRG

# Show deployment details
az deployment group show \
  --resource-group MyRG \
  --name MyDeployment

# Delete deployment history (not resources)
az deployment group delete \
  --resource-group MyRG \
  --name MyDeployment

# Deploy at subscription scope
az deployment sub create \
  --location eastus \
  --template-file subscription-template.json

# Deploy with inline parameters
az deployment group create \
  --resource-group MyRG \
  --template-file main.bicep \
  --parameters vmName=MyVM adminUsername=azureuser
```

```powershell
# PowerShell Commands

# Validate ARM template
Test-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile azuredeploy.json `
  -TemplateParameterFile azuredeploy.parameters.json

# Deploy ARM template  
New-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile azuredeploy.json `
  -TemplateParameterFile azuredeploy.parameters.json

# Deploy Bicep file
New-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile main.bicep `
  -location eastus `
  -vmSize Standard_DS2_v2

# What-if deployment
New-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile main.bicep `
  -TemplateParameterFile parameters.json `
  -WhatIf

# Export deployment template
Export-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -DeploymentName MyDeployment `
  -Path exported-template.json

# Export resource group template
Export-AzResourceGroup `
  -ResourceGroupName MyRG `
  -Path exported-rg-template.json

# Convert ARM to Bicep (requires Bicep CLI)
bicep decompile azuredeploy.json

# Convert Bicep to ARM (requires Bicep CLI)
bicep build main.bicep

# List deployments
Get-AzResourceGroupDeployment -ResourceGroupName MyRG

# Show deployment details
Get-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -DeploymentName MyDeployment

# Remove deployment from history
Remove-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -DeploymentName MyDeployment

# Deploy at subscription scope
New-AzDeployment `
  -Location eastus `
  -TemplateFile subscription-template.json

# Deploy with hashtable parameters
$parameters = @{
  vmName = "MyVM"
  adminUsername = "azureuser"
}

New-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile main.bicep `
  -TemplateParameterObject $parameters

# Complete mode deployment (deletes extra resources)
New-AzResourceGroupDeployment `
  -ResourceGroupName MyRG `
  -TemplateFile main.bicep `
  -Mode Complete
```

### Additional Information

#### ARM Template Key Functions

- `concat()`: Combine strings
- `resourceGroup()`: Get current resource group info
- `subscription()`: Get current subscription info
- `parameters()`: Reference parameter values
- `variables()`: Reference variable values
- `reference()`: Get resource properties
- `resourceId()`: Construct resource identifier
- `uniqueString()`: Generate deterministic hash

#### Bicep Key Features

- Simpler syntax than ARM JSON
- Direct property access
- String interpolation: `'prefix-${vmName}-suffix'`
- Conditional deployment: `if (condition) { ... }`
- Loops: `for i in range(0, count): { ... }`
- Modules for reusability
- Better IntelliSense support

#### Common Template Patterns

- **Linked Templates**: Reference external templates
- **Nested Templates**: Embed templates within templates
- **Conditional Resources**: Deploy based on parameters
- **Copy Element**: Create multiple instances
- **Dependencies**: Control deployment order

#### Export Considerations

- Clean up auto-generated names
- Remove system-assigned properties
- Add parameters for environment differences
- Fix hardcoded values
- Update API versions if needed