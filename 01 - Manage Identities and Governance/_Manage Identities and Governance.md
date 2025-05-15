# Manage Identities and Governance (20-25%)

### Objective Overview

Refer to individual objective pages for details:

- [[01 - Manage Microsoft Entra users and groups]]
- [[02 - Manage access to Azure resources]]
- [[03 - Manage Azure subscriptions and governance]]

---

### Key Concepts

- **Microsoft Entra ID (formerly Azure AD)**: Cloud-based identity and access management service
- **RBAC (Role-Based Access Control)**: Controls access to Azure resources based on assigned roles
- **Azure Policy**: Service for creating, assigning, and managing policies to enforce compliance
- **Management Groups**: Containers that help organize subscriptions into a hierarchy
- **Resource Hierarchy**: Management Groups > Subscriptions > Resource Groups > Resources

### Relevant Terms & Definitions

| Term                   | Definition                                                                                                 | Portal Location                                                             |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| RBAC                   | Role-Based Access Control system for managing resource access                                              | Any resource > Access control (IAM)                                         |
| Policy Definition      | Single rule definition for resource compliance                                                             | Policy > Definitions                                                        |
| Policy Initiative      | Collection of policy definitions                                                                           | Policy > Initiatives                                                        |
| Resource Lock          | Protection against accidental deletion or modification                                                     | Any resource > Locks                                                        |
| Management Group       | Container for organizing subscriptions                                                                     | Management Groups                                                           |
| Cost Management        | Tools for monitoring and controlling Azure costs                                                           | Cost Management + Billing                                                   |
| Entitlement Management | Identity governance solution for automating access request workflows, assignments, reviews, and expiration | Microsoft Entra admin center > Identity governance > Entitlement management |

### Service Comparison

|Feature|Microsoft Entra ID|Azure RBAC|Azure Policy|
|---|---|---|---|
|Primary Focus|Identity management|Resource access control|Resource compliance|
|Scope|Directory-wide|Management group to resource|Management group to resource|
|Inheritance|Group membership|Role assignments|Policy assignments|
|Default Options|Standard users/groups|100+ built-in roles|200+ built-in policies|
|Custom Options|Custom attributes|Custom roles|Custom policy definitions|

---

### Azure CLI Commands

```bash
# 1. Create a new user in Microsoft Entra ID
az ad user create --display-name "John Doe" --password "Password123!" --user-principal-name john@contoso.com

# 2. Assign a role to a user
az role assignment create --role "Contributor" --assignee john@contoso.com --resource-group "myResourceGroup"

# 3. Create an Azure policy assignment
az policy assignment create --policy "Require tag on resource groups" --name "require-rg-tag" --scope "/subscriptions/{subscriptionId}"

# 4. Apply a resource lock
az lock create --name "DoNotDelete" --resource-group "myResourceGroup" --lock-type CanNotDelete

# 5. Create a management group
az account management-group create --name "Finance" --display-name "Finance Department"

# 6. Apply tags to resources
az resource tag --tags "Environment=Production" "Department=Finance" --resource-group "myResourceGroup" --name "myResource" --resource-type "Microsoft.Web/sites"

# 7. Create a budget
az consumption budget create --name "Monthly Budget" --amount 1000 --time-grain monthly --scope "subscriptions/{subscriptionId}"
```

### PowerShell Commands

```powershell
# 1. Create a new user in Microsoft Entra ID
New-AzADUser -DisplayName "John Doe" -UserPrincipalName john@contoso.com -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -MailNickname "john"

# 2. Assign a role to a user
New-AzRoleAssignment -SignInName john@contoso.com -RoleDefinitionName "Contributor" -ResourceGroupName "myResourceGroup"

# 3. Create an Azure policy assignment
New-AzPolicyAssignment -Name "require-rg-tag" -PolicyDefinition (Get-AzPolicyDefinition -Name "Require tag on resource groups") -Scope "/subscriptions/{subscriptionId}"

# 4. Apply a resource lock
New-AzResourceLock -LockName "DoNotDelete" -LockLevel CanNotDelete -ResourceGroupName "myResourceGroup"

# 5. Create a management group
New-AzManagementGroup -GroupName "Finance" -DisplayName "Finance Department"

# 6. Apply tags to resources
$tags = @{"Environment"="Production"; "Department"="Finance"}
Set-AzResource -ResourceGroupName "myResourceGroup" -Name "myResource" -ResourceType "Microsoft.Web/sites" -Tag $tags -Force

# 7. Manage subscription
Set-AzContext -Subscription "MySubscriptionName"
```


---


### Cost Considerations

- Azure AD licensing costs (Free vs Premium P1 vs Premium P2)
- Cost implications of different role assignments (admin roles with broader permissions can increase risk)
- Cost management through proper resource organization and tagging
- Budget alert thresholds and notification strategies
- Cost optimization recommendations from Azure Advisor

### Security Best Practices

- Least privilege access model implementation
- Regular access reviews and cleanup
- Using Conditional Access policies
- Implementing Azure AD PIM for just-in-time access
- Enforcing MFA for all users, especially privileged accounts
- Proper external user management
- SSPR implementation to reduce admin overhead

### Business Use Cases

- Enterprise reorganization requiring new subscription structure
- Department-specific resource access control
- Contractor/vendor access management
- Compliance requirements enforcement through Azure Policy
- Cost allocation through tagging
- Resource protection through locks
- License management for fluctuating workforce

### Real-World Analogies

- RBAC is like a hotel key card system: different cards (roles) grant access to different rooms (resources)
- Management Groups are like departments in a company, organizing work units hierarchically
- Resource Locks are like childproofing cabinets - preventing accidental changes
- Azure Policy is like building code regulations that all structures must follow
- SSPR is like a self-service kiosk at the airport for checking in without staff assistance

### Common Pitfalls & Misconceptions

- Confusing Azure AD roles vs Azure RBAC roles
- Not understanding inheritance in management groups
- Applying too many/too few resource locks
- Not planning tag strategy before implementation
- Over-provisioning licenses
- Confusing scope levels for policy and role assignments
- Not implementing break-glass emergency access accounts

### Things to Look Out for on the Exam

- Questions about scope hierarchy and inheritance
- Scenarios requiring selection of the right built-in role
- Policy definitions vs initiatives vs assignments
- Resource lock types and implications
- Management group nesting limitations
- License requirements for specific Azure AD features
- Actions that require Global Administrator vs other roles


---

#### Practice Test Questions I Got Wrong

##### Question 1

**Question:**

**Options:**

- A.
- B.
- C.
- D.

**Correct Answer:**

**Why I Got It Wrong:**

**What to Remember:**


---

##### Question 2

**Question:**

**Options:**

- A.
- B.
- C.
- D.

**Correct Answer:**

**Why I Got It Wrong:**

**What to Remember:**


---

#### Practice Scenarios

##### Scenario 1:

Your global manufacturing company has recently moved to Azure and has set up a hub-and-spoke network architecture. You need to implement an identity and access management solution that meets the following requirements:

- Finance department needs Contributor access to finance-related resource groups but only Reader access to production VMs
- IT administrators need to temporarily elevate privileges for emergency changes
- External auditors need read-only access to specific resources for 30-day periods
- All privileged role assignments must be reviewed quarterly
- Regional teams should only manage resources in their geographic Azure regions

**Solution:**

1. Implement Azure AD Premium P2 for PIM and Access Reviews capabilities
2. Create the following Azure AD groups:
    - Finance-Contributors (assigned to finance resource groups with Contributor role)
    - Finance-VM-Readers (assigned to production VM resources with Reader role)
    - Regional-IT-EastUS (assigned to EastUS resources with Contributor role)
    - Regional-IT-WestEurope (assigned to WestEurope resources with Contributor role)
    - Auditor-Group (assigned temporary Reader role through PIM)
3. Configure Azure AD PIM:
    - Set up eligible role assignments for IT admins to request Owner privileges
    - Configure just-in-time access requiring approval and justification
    - Set maximum activation duration to 8 hours
4. Set up quarterly Access Reviews:
    - Target all privileged role assignments
    - Require attestation from resource owners
    - Auto-remove access if not approved
5. Use Conditional Access policies:
    - Require MFA for all privileged role activations
    - Restrict external auditor access to corporate-managed devices

##### Scenario 2:

Your retail company manages over 500 VMs across 30 resource groups and 3 subscriptions. The finance team needs better cost allocation reporting based on department, application, and environment. However, despite a tagging policy, only 60% of resources have proper tags. Management wants to enforce the following tagging standards:

- Required tags: Department, Application, Environment, Owner
- Environment must be one of: Production, Development, Testing, or Staging
- Department values should match the company's organizational structure
- Any resource without proper tags should be automatically flagged and remediated

**Solution:**

1. Create an Azure Policy initiative that combines multiple policy definitions:
    
    - Create policies for each required tag with "Modify" effect
    - Create an audit policy for invalid Environment values
    - Set default values where possible (e.g., Environment=Development for dev subscription resources)
2. Implement remediation tasks:
    
    - Create tasks that can apply missing tags to existing resources
    - Schedule monthly remediation runs
3. Set up tag inheritance where possible:
    
    - Configure resource group tags to be inherited by new resources
    - Use policy to enforce child resources inheriting parent tags
4. Apply the initiative at Management Group level to cover all subscriptions
    
5. Create cost reports and dashboards:
    
    - Set up Azure Cost Management views filtered by tags
    - Create Power BI reports for department chargebacks
    - Configure monthly exports to finance department
6. Implement tag governance process:
    
    - Create a custom RBAC role for tag management
    - Assign Tag Contributors to each department liaison

##### Scenario 3:

Your healthcare organization is expanding to multiple regions and needs to ensure regulatory compliance while maintaining operational efficiency. You need to implement a governance structure that satisfies:

- Different regulations apply to different geographic regions (HIPAA in US, GDPR in Europe)
- Development teams need freedom to create resources but within strict security guardrails
- Azure costs must be controlled and predicted, with approvals for any spend above 90% of budget
- No production system can be deleted without a 7-day waiting period
- All networking configuration changes must be logged for audit purposes

**Solution:**

1. Set up a Management Group hierarchy:
    
    - Top-level Management Group (Healthcare-Global)
    - Second-level geographic Management Groups (US-Regulated, Europe-Regulated, Non-Regulated)
    - Third-level environment Management Groups (Production, Non-Production)
2. Implement Azure Policy at appropriate levels:
    
    - Assign HIPAA policies at US-Regulated group
    - Assign GDPR policies at Europe-Regulated group
    - Assign common security baseline policies at Healthcare-Global level
    - Assign stricter policies at Production level vs. Non-Production
3. Configure Resource Locks:
    
    - Apply DeleteLock on all Production resource groups
    - Apply ReadOnly locks on critical networking components
4. Set up Azure Budgets:
    
    - Create subscription-level budgets with 90% threshold alerts
    - Configure action groups to notify finance approvers
    - Integrate with Logic Apps to pause non-critical dev/test resources when budget exceeds 95%
5. Implement Activity Log monitoring:
    
    - Create Log Analytics workspace for centralized logging
    - Configure diagnostic settings to capture all administrative operations
    - Set up alerts for networking configuration changes
    - Create custom dashboards for compliance officers
6. Configure Azure Blueprints:
    
    - Create compliant environment templates for each region
    - Include policy assignments, RBAC settings, and resource templates
    - Enable controlled roll-out of compliance changes

##### Scenario 4:

Your financial services company is restructuring its IT departments and needs to reorganize Azure resources without disruption. Current state includes 50+ resource groups spread across 5 subscriptions with inconsistent naming and organization. The new structure requires:

- Clear separation between production and non-production environments
- Resources grouped by application rather than department
- Consistent access patterns for support teams
- Ability to track costs by business unit, application, and environment
- Simplified license management for 2000+ users with frequent role changes

**Solution:**

1. Develop a resource organization plan:
    
    - Create naming convention for all new resources
    - Map existing resources to new structure
    - Identify dependencies between resources
2. Set up new Management Group structure:
    
    - Create Production and Non-Production top-level groups
    - Create Application-based groups beneath these
    - Place subscriptions appropriately in the hierarchy
3. Implement Azure AD group-based license assignment:
    
    - Create dynamic groups based on department attributes
    - Assign appropriate license SKUs to each group
    - Configure group membership rules to automatically update as employees change roles
4. Create a resource movement plan:
    
    - Use Azure Resource Mover to plan migrations
    - Script resource moves between resource groups
    - Schedule moves during maintenance windows
    - Verify network dependencies before migration
5. Establish new RBAC model:
    
    - Create Application Support groups with custom roles
    - Implement least-privilege access model
    - Use PIM for just-in-time access to production
    - Configure access reviews for all elevated permissions
6. Deploy standardized tagging:
    
    - BusinessUnit, Application, Environment, CostCenter
    - Use Azure Policy to enforce tagging during migration
    - Configure auto-tagging where possible
7. Set up new cost reporting structure:
    
    - Create Cost Management views by business unit
    - Configure budget alerts for application owners
    - Implement chargeback reporting by cost center

##### Scenario 5:

Your multinational corporation is adopting a multi-cloud strategy but wants to centralize identity management. You need to implement a solution that:

- Provides single sign-on experience across Azure, AWS, and GCP
- Enforces MFA for all administrative access
- Automates employee onboarding/offboarding across cloud platforms
- Ensures least privilege across all environments
- Meets SOC2 compliance requirements for access controls
- Provides comprehensive audit trail of all privileged operations

**Solution:**

1. Implement Azure AD as the central identity provider:
    
    - Configure federation with on-premises Active Directory
    - Set up SCIM provisioning with HR systems
    - Enable Azure AD Connect cloud sync for hybrid identity
2. Configure enterprise applications for cloud platforms:
    
    - Set up Azure AD application for AWS SSO
    - Configure GCP as an enterprise application
    - Implement SAML-based authentication for third-party services
3. Create administrative tier model:
    
    - Tier 0: Cloud platform administrators (highly restricted)
    - Tier 1: Service administrators (limited to specific services)
    - Tier 2: Application team members (day-to-day operations)
4. Implement Conditional Access policies:
    
    - Require MFA for all cloud administrative portals
    - Enforce device compliance for sensitive operations
    - Restrict administrative access to corporate networks
    - Block access from high-risk locations
5. Set up automated access lifecycle:
    
    - Use dynamic groups based on HR attributes
    - Configure access package for common role combinations
    - Implement automated reviews for privileged access
    - Create offboarding workflows triggered by HR status changes
6. Deploy centralized logging solution:
    
    - Configure Azure Sentinel for cross-cloud monitoring
    - Set up Azure Log Analytics as central repository
    - Create custom workbooks for SOC2 compliance reporting
    - Implement alerting for suspicious privilege escalation
7. Establish governance framework:
    
    - Monthly access reviews of cross-cloud privileges
    - Quarterly compliance assessment against SOC2
    - Risk-based approach to access exceptions



---

#### Project Ideas


