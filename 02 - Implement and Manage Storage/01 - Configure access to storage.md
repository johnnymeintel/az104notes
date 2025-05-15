# Configure Access to Storage

### Objectives

- **Configure Azure Storage firewalls and virtual networks**
    - Set IP-based firewall rules to allow specific public IP addresses or ranges
    - Enable virtual network rules to restrict access to specific VNets/subnets
    - Configure service endpoints on VNets to route traffic through Azure backbone
    - Implement private endpoints for completely private connectivity
    - Set up trusted Azure service exceptions (Backup, Site Recovery, etc.)
    - Choose between "Allow" and "Deny" as default network action
    - Understand metrics and logging for network rule evaluation

- **Create and use shared access signature (SAS) tokens**
    - **Service SAS**: Grants access to specific service (Blob, Queue, Table, or File)
    - **Account SAS**: Can grant access to multiple services in an account
    - **User Delegation SAS**: Uses Azure AD credentials (Blob only)
    - Define granular permissions (read, write, delete, list, create, etc.)
    - Set start/expiry times and allowed IP ranges
    - Configure allowed protocols (HTTPS only vs HTTP/HTTPS)
    - Specify allowed resource types and services
    - Include signature version and signing key information

- **Configure stored access policies**
    - Create up to 5 stored access policies per container/queue/table/share
    - Define reusable permission sets that can be referenced by SAS tokens
    - Modify or revoke permissions without regenerating SAS tokens
    - Set identifier name, permissions, and expiry time
    - Link SAS tokens to policies for centralized management
    - Best practice: Use stored access policies for long-term SAS tokens

- **Manage access keys**
    - Two access keys per storage account (primary and secondary)
    - Full administrative access to storage account data
    - Rotate keys regularly (recommended: every 90 days)
    - Update applications to use secondary key before regenerating primary
    - Monitor key usage in Azure Monitor logs
    - Consider using Azure Key Vault for automated key rotation
    - Access keys work across all storage services in the account

- **Configure identity-based access for Azure Files**
    - **Azure AD Domain Services (Azure AD DS)**: Managed domain services in Azure
    - **On-premises AD DS**: Requires AD Connect and VPN/ExpressRoute
    - **Azure AD Kerberos**: For hybrid identities (preview)
    - Assign share-level permissions using RBAC
    - Configure NTFS permissions on files/directories
    - Mount drives using user credentials instead of storage keys
    - Supports SMB 3.0 with encryption and secure channel

### Common Commands

```bash
# Azure CLI Commands

# Configure Storage Firewall - Allow specific IP
az storage account update \
  --name mystorageaccount \
  --resource-group MyRG \
  --default-action Deny

az storage account network-rule add \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --ip-address 13.65.24.96

# Configure Virtual Network Access
az storage account network-rule add \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --subnet MySubnet

# Generate Service SAS for Blob
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob \
  --permissions rw \
  --expiry 2024-12-31T23:59:59Z

# Generate Account SAS
az storage account generate-sas \
  --account-name mystorageaccount \
  --permissions rwdlacup \
  --resource-types sco \
  --services bfqt \
  --expiry 2024-12-31T23:59:59Z

# Create Stored Access Policy
az storage container policy create \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name mypolicy \
  --permissions rwl \
  --expiry 2024-12-31T23:59:59Z

# List Storage Account Keys
az storage account keys list \
  --account-name mystorageaccount \
  --resource-group MyRG

# Regenerate Storage Key
az storage account keys renew \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --key primary

# Enable Azure AD Auth for Files
az storage account update \
  --name mystorageaccount \
  --resource-group MyRG \
  --enable-files-aadds true
```

```powershell
# PowerShell Commands

# Configure Storage Firewall
Update-AzStorageAccountNetworkRuleSet `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -DefaultAction Deny

Add-AzStorageAccountNetworkRule `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -IPAddressOrRange "13.65.24.96"

# Add Virtual Network Rule
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName "MyRG" -Name "MyVNet") `
  -Name "MySubnet"

Add-AzStorageAccountNetworkRule `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -VirtualNetworkResourceId $subnet.Id

# Create SAS Token for Blob
$ctx = Get-AzStorageAccount -ResourceGroupName "MyRG" -Name "mystorageaccount" | Get-AzStorageAccountKey | Select-Object -First 1 | Get-AzStorageContext

New-AzStorageBlobSASToken `
  -Container "mycontainer" `
  -Blob "myblob" `
  -Permission rw `
  -ExpiryTime (Get-Date).AddDays(30) `
  -Context $ctx

# Create Account SAS
New-AzStorageAccountSASToken `
  -Service Blob,File,Table,Queue `
  -ResourceType Service,Container,Object `
  -Permission rwdlacup `
  -ExpiryTime (Get-Date).AddDays(365) `
  -Context $ctx

# Create Stored Access Policy
New-AzStorageContainerStoredAccessPolicy `
  -Container "mycontainer" `
  -Policy "mypolicy" `
  -Permission rwl `
  -ExpiryTime (Get-Date).AddDays(90) `
  -Context $ctx

# Get Storage Account Keys
Get-AzStorageAccountKey `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount"

# Regenerate Storage Key
New-AzStorageAccountKey `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -KeyName key1

# Enable Azure AD for Files
Set-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -EnableAzureActiveDirectoryDomainServicesForFile $true
```

### Additional Relevant Information

#### Network Security Architecture

- **Service Endpoints**: Keep traffic on Azure backbone, source IP appears as private
- **Private Endpoints**: Assign private IP to storage, accessible only within VNet
- **Trusted Services**: Azure Backup, Azure Site Recovery, Azure Monitor, etc.
- **Firewall Rules**: Evaluated in order - specific rules override default action

#### SAS Token Best Practices

- Use HTTPS-only SAS tokens in production
- Implement User Delegation SAS when possible (Azure AD-based)
- Short expiration times for ad-hoc access
- Stored Access Policies for long-term scenarios
- Never expose SAS tokens in client-side code
- Monitor SAS token usage in Storage Analytics logs

#### Access Key Security

- Never commit access keys to source control
- Use Azure Key Vault for key management
- Implement key rotation automation
- Use managed identities instead of keys when possible
- Monitor for unauthorized key usage

#### Identity-Based Access Levels

1. **Share-level permissions (RBAC)**:
    - Storage File Data SMB Share Reader
    - Storage File Data SMB Share Contributor
    - Storage File Data SMB Share Elevated Contributor
2. **File/Directory permissions (NTFS)**:
    - Read, Write, Execute
    - Full Control, Modify, etc.

#### Common Exam Scenarios

- Choosing between Service SAS vs Account SAS vs User Delegation SAS
- Understanding when to use private endpoints vs service endpoints
- Configuring least-privilege access with SAS parameters
- Implementing key rotation without service disruption
- Troubleshooting network access issues with firewall rules