# Manage Microsoft Entra ID Users and Groups

### Objectives

- **Create users and groups**
    - Entra ID is Microsoft's cloud-based identity service (formerly Azure AD)
    - Users can be cloud-only or synchronized from on-premises AD
    - Groups provide access management for multiple users simultaneously
    - Two group types: Security groups and Microsoft 365 groups

- **Manage user and group properties**
    - User properties include name, contact info, job info, usage location
    - Group properties include name, description, membership type, owners
    - Membership types: Assigned, Dynamic user, Dynamic device

- **Manage licenses in Microsoft Entra ID**
    - Licenses can be assigned directly to users or via group-based licensing
    - Group-based licensing simplifies license management at scale
    - License assignment requires setting user's usage location

- **Manage external users**
    - Guest users from other directories or email addresses
    - B2B collaboration allows secure sharing with external partners
    - Guest user permissions are more restricted than member users

- **Configure self-service password reset (SSPR)**
    - Allows users to reset passwords without admin intervention
    - Authentication methods: Email, phone, security questions
    - Can be enabled for all users or specific groups
    - Registration policy determines how users register authentication methods

### Common Commands

```bash
# Create a new user
az ad user create --display-name "John Doe" --password "Password123!" --user-principal-name john@contoso.com

# Create a new group
az ad group create --display-name "Marketing Department" --mail-nickname marketing

# Add user to group
az ad group member add --group "Marketing Department" --member-id user-object-id

# List users
az ad user list

# Get user details
az ad user show --id john@contoso.com

# Invite external user
az ad invitation create --email-address guest@external.com --redirect-url https://portal.azure.com
```

```powershell
# Create a new user
New-AzADUser -DisplayName "John Doe" -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -UserPrincipalName john@contoso.com -MailNickname "john"

# Create a new group
New-AzADGroup -DisplayName "Marketing Department" -MailNickname "marketing"

# Add user to group
Add-AzADGroupMember -TargetGroupObjectId group-object-id -MemberObjectId user-object-id

# Get user details
Get-AzADUser -UserPrincipalName john@contoso.com

# Configure SSPR (requires Microsoft Graph PowerShell)
Update-MgIdentityAuthenticationMethodPolicy -AuthenticationMethodPolicyId PasswordReset -IsEnabled $true

# Assign license (requires Microsoft Graph PowerShell)
Set-MgUserLicense -UserId user-id -AddLicenses @{SkuId = "license-sku-id"} -RemoveLicenses @()
```