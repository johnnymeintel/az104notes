### Objectives


- **Describe Microsoft Entra ID.**
	- Cloud-based IAM service - managing identities, enforcing access policies, and securing applications and data.
	- PaaS
	- New Azure subscription gets Default Directory Free Tier
	- Default DNS domain name unique prefix + onmicrosoft.com
	- The Entra tenant serves as security boundary and container for Entra objects (users, groups, apps)

- **Compare Microsoft Entra ID to Active Directory Domain Services (AD DS).**
	- AD DS runs on Windows Server (domain controller)
	- Unlike AD DS - Entra ID:
		- Support for MFA
		- Identity protection
		- SSPR
		- Multi-tenant, flat structure by design
		- Schema contains fewer object types (device class, no computer class
		- No OU class (uses groups)
		- HTTP and HTTPS (REST API) communications
		- Instead of Kerberos, uses SAML, WS-Federation, and OpenID Connect for authentication, and OAuth for authorization
		- Federation services (Facebook)
	- Unlike Entra ID - AD DS:
		- Hierarchal x.500-based structure
		- AD DS uses DNS for locating domain controllers
		- LDAP
		- Kerberos
		- OUs and GPOs
		- Trusts between domains for delegated management
		- Separate data disk (don't use C drive)
		- Host Cache Preference set to 'None' 

- **Describe how Microsoft Entra ID is used as a directory for cloud apps.**
	- Application object - app blueprint
	- servicePrincipal object - actual instance of the app running (define app once, use servicePrincipal to run in multiple tenants)
	- Each cloud service that needs authentication will create its own Microsoft Entra tenant
	- You can enable Microsoft Entra authentication for the Web Apps feature of Azure App Service directly from the Authentication/Authorization blade in the Azure portal
	- It’s possible to apply different authentication settings to individual deployment slots

- **Describe Microsoft Entra ID P1 and P2.**
	- Unlike P1, P2 has:
		- Entra ID Protection - define user risk policies and sign-in policies, review user's behavior and flag users for risk
		- PIM - additional security levels for privileged users (permanent vs temporary admins), policy workflow. You need either Microsoft Entra ID Governance licenses or Microsoft Entra ID P2 licenses to use PIM and all of its settings

- **Describe Microsoft Entra Domain Services.**
	- P1 or P2
	- Group Policy management
	- Domain joining
	- Kerberos authentication
	- Compatible with locally deployed AD DS (but not necessary)
	- Admins don't need to manage, update, monitor domain controllers or deploy/manage AD replication
	- Only base computer AD object supported
	- Not possible to extend the schema
	- No nested OUs
	- Built-in GPO



---

### Common Commands

#### Scenario 1: User Lifecycle Management

```powershell
# Create user
New-AzureADUser -DisplayName "New Tactical Operator" -UserPrincipalName "tacticalop@yourtenant.onmicrosoft.com" -PasswordProfile $PasswordProfile -AccountEnabled $true -MailNickName "tacticalop"

# Disable user account
Set-AzureADUser -ObjectId "user-object-id" -AccountEnabled $false

# Delete user
Remove-AzureADUser -ObjectId "user-object-id"

# Get user properties
Get-AzureADUser -ObjectId "user-object-id" | Format-List

# List all users
Get-AzureADUser -All $true
```

#### Scenario 2: Group Management


```powershell
# Create dynamic group (requires P1 license)
New-AzureADMSGroup -DisplayName "All Contractors" -Description "Dynamic group for contractors" -MailEnabled $false -SecurityEnabled $true -MailNickname "contractors" -GroupTypes "DynamicMembership" -MembershipRule "(user.employeeType -eq ""Contractor"")" -MembershipRuleProcessingState "On"

# Add owner to group
Add-AzureADGroupOwner -ObjectId "group-object-id" -RefObjectId "user-object-id"

# Get group members
Get-AzureADGroupMember -ObjectId "group-object-id"
```

#### Scenario 3: Application Management


```powershell
# Create service principal (instantiates application in tenant)
New-AzureADServicePrincipal -AppId "application-id"

# Assign app to user
New-AzureADUserAppRoleAssignment -ObjectId "user-object-id" -PrincipalId "user-object-id" -ResourceId "service-principal-object-id" -Id "app-role-id"

# List all enterprise applications
Get-AzureADServicePrincipal -All $true | Where-Object {$_.Tags -contains "WindowsAzureActiveDirectoryIntegratedApp"}
```

#### Scenario 4: Role-Based Access Control


```powershell
# List directory roles
Get-AzureADDirectoryRole

# Assign user to directory role
Add-AzureADDirectoryRoleMember -ObjectId "role-object-id" -RefObjectId "user-object-id"

# List members of a role
Get-AzureADDirectoryRoleMember -ObjectId "role-object-id"

# View PIM-eligible role assignments
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "tenant-id"

# Make a user eligible for Global Administrator
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId "aadRoles" -ResourceId "tenant-id" -RoleDefinitionId "global-admin-role-id" -SubjectId "user-object-id" -AssignmentState "Eligible" -Schedule $schedule

```


---

### Practice Questions

1. **Scenario**: Your organization has implemented Microsoft Entra ID and has users with on-premises AD DS accounts. You need to ensure users can use the same credentials for both on-premises and cloud resources. Which solution should you implement?
    - A. Deploy Federation Services
    - **B. Implement Microsoft Entra Connect**
    - C. Upgrade to Microsoft Entra ID P1
    - D. Deploy Microsoft Entra Domain Services

2. **Scenario**: You have an Azure web application that needs to authenticate users from your organization's Microsoft Entra ID. Which protocol would you use to implement this authentication?
    - A. Kerberos
    - B. LDAP
    - **C. OpenID Connect**
    - D. NTLM

3. **Scenario**: Your organization has implemented Microsoft Entra ID P2. You need to ensure that the Global Administrator role is only assigned temporarily when needed. Which feature should you use?
    - A. Conditional Access
    - B. Just-In-Time Access
    - **C. Privileged Identity Management (PIM)**
    - D. Self-Service Password Reset

4. **Scenario**: You have users in Microsoft Entra ID and need to organize them by department. In Microsoft Entra ID, what should you use instead of Organizational Units?
    - A. Administrative Units
    - B. Management Groups
    - C. Resource Groups
    - **D. Security Groups**

5. **Scenario**: You are developing a multi-tenant application that will authenticate users from any Microsoft Entra ID tenant. When registering your application, which account types should you select?
    - A. Accounts in this organizational directory only
    - **B. Accounts in any organizational directory**
    - C. Accounts in any organizational directory and personal Microsoft accounts
    - D. Personal Microsoft accounts only
