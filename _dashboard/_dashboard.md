# AZ-104 Exam Dashboard

_Exam Date: May 30, 2025 | Current: May 11, 2025_

##### Currently Studying:
- Practice exams.

##### Immediate Actions:
- Complete all MeasureUp and WhizLabs practice questions, screenshot it.

---

### Service Interaction Matrix

|Primary Service|Impacts|Security Touch Points|Cost Implications|Complexity|
|---|---|---|---|---|
|Virtual Network|NSG, Bastion, Peering, Load Balancer|Service/Private Endpoints, Azure Firewall|Egress charges, Peering costs|🟡|
|Storage Account|VNet integration, Backup, CDN|Firewall rules, Private endpoints, SAS|Access tier charges, Transaction costs|🟢|
|Virtual Machines|VNet, Storage, Load Balancer|NSG, Managed Identity, Disk encryption|Compute hours, Storage, Network|🟢|
|Azure AD|All services via RBAC|Conditional Access, MFA, PIM|License costs per user|🟡|
|Policy & RBAC|All resource deployments|Compliance, Access control|Indirect via resource restrictions|🔴|

---

### Exam Objectives & Weight Distribution

#### 1. [[_Manage Identities and Governance]] (20-25%)

- Microsoft Entra ID (Azure AD) 🟡
- RBAC & Scope Hierarchy 🟢
- Azure Policy & Governance 🔴
- Management Groups & Subscriptions 🟡
- Cost Management 🟢

##### Pattern Triggers - Identities & Governance

- "Inheritance" → Think scope hierarchy (Management Groups → Subscriptions → RGs)
- "Least privilege" → RBAC custom roles
- "Compliance" → Azure Policy, not RBAC
- "Cost control" → Azure Policy for resource restrictions

#### 2. [[_Implement and Manage Storage]] (15-20%)

- Storage Account Configuration 🟢
- Access Management (SAS, Keys, RBAC) 🟡
- Azure Files & Blob Storage 🟢
- Storage Redundancy & Replication 🟢
- Lifecycle Management 🟡

##### Pattern Triggers - Storage

- "Secure access" → SAS tokens for temporary, Keys for permanent
- "High availability" → ZRS for zone, GRS for region
- "Cost optimization" → Lifecycle policies for tier transitions
- "Network security" → Service endpoints vs Private endpoints

#### 3. [[_Deploy and Manage Azure Compute Resources]] (20-25%)

- Virtual Machines & Scale Sets 🟢
- ARM Templates & Bicep 🔴
- Container Solutions (ACI, ACA, ACR) 🟡
- Azure App Service 🟢
- Availability Sets vs Zones 🟢

##### Pattern Triggers - Compute

- "High availability" → Zones (99.99%) over Sets (99.95%)
- "Auto-scaling" → VMSS or App Service
- "Quick deployment" → Container Instances
- "Infrastructure as Code" → ARM/Bicep templates

#### 4. [[_Implement and Manage Virtual Networking]] (15-20%)

- Virtual Networks & Subnets 🟢
- Network Security (NSGs, ASGs) 🟡
- Load Balancing & DNS 🟡
- VNet Peering & Connectivity 🟡
- Service/Private Endpoints 🔴

##### Pattern Triggers - Networking

- "Cross-VNet" → Peering (non-transitive)
- "Internet access" → NAT Gateway or Azure Firewall
- "Private access" → Private endpoints
- "Load distribution" → Standard LB for zones

#### 5. [[_Monitor and Maintain Azure Resources]] (10-15%)

- Azure Monitor (Metrics vs Logs) 🟡
- Backup & Recovery Solutions 🟢
- Site Recovery vs Backup 🟡
- Network Watcher 🟡
- Alerts & Action Groups 🟢

##### Pattern Triggers - Monitoring

- "Real-time data" → Metrics
- "Historical analysis" → Logs
- "Automated response" → Action Groups
- "Network issues" → Network Watcher

---

## Quick Decision Trees

### Minimize Cost Questions

```
Storage optimization?
├─ Frequent access? → Hot tier
├─ Monthly access? → Cool tier
├─ Yearly access? → Archive tier
└─ Cross-region backup? → Consider LRS + manual copy vs GRS

Compute optimization?
├─ Predictable workload? → Reserved Instances
├─ Interruptible? → Spot VMs
├─ Dev/Test? → Auto-shutdown policies
└─ Variable load? → Scale Sets with auto-scale

Network optimization?
├─ Inter-region traffic? → Consider VPN vs ExpressRoute
├─ Egress heavy? → Place resources in same region
└─ CDN candidate? → Static content offload
```

### Maximize Security Questions

```
Access control?
├─ Human users? → Azure AD + MFA + Conditional Access
├─ Service-to-service? → Managed Identity
├─ External apps? → Service Principal + Certificate
└─ Temporary access? → SAS tokens or Azure AD PIM

Network security?
├─ Internet exposure? → WAF + Front Door
├─ Private only? → Private Endpoints
├─ Hybrid connectivity? → VPN with forced tunneling
└─ Management plane? → Azure Bastion
```

### High Availability Questions

```
Compute HA?
├─ Region failure? → Traffic Manager + Multi-region
├─ Zone failure? → Availability Zones
├─ Rack failure? → Availability Sets
└─ Instance failure? → Load Balancer health probes

Storage HA?
├─ Region failure? → GRS/RA-GRS/GZRS
├─ Zone failure? → ZRS
├─ Datacenter failure? → LRS (11 9's)
└─ Deleted data? → Soft delete + versioning
```

---

## Quick Reference Cards

### Key SLAs & Durability

|Service Type|SLA|Notes|Complexity|
|---|---|---|---|
|Availability Zones|99.99%|Cross-datacenter|🟢|
|Availability Sets|99.95%|Within datacenter|🟢|
|LRS|11 9's|Single datacenter|🟢|
|ZRS|12 9's|Across zones|🟡|
|GRS/GZRS|16 9's|Cross-region|🟡|

### Storage Access Tiers

|Tier|Min Retention|Use Case|Early Deletion|Complexity|
|---|---|---|---|---|
|Hot|None|Frequent access|None|🟢|
|Cool|30 days|Infrequent access|Penalty|🟢|
|Archive|180 days|Rare access|Penalty|🟡|

### NSG Rule Evaluation

1. Inbound: Subnet NSG → NIC NSG
2. Outbound: NIC NSG → Subnet NSG
3. Priority: 100-4096 (lower = higher precedence)
4. Default rules: 65000-65500 (cannot delete)

### Quick Notes

- Bastion needs own subnet named AzureBastionSubnet. RDP access to VMs through web browser
- Boot diagnostics does not support a premium storage account
- Bicep file defines actionGroups and activityLogAlerts
- Action groups are response mechanisms, alerts are detection mechanisms

---

## Command Cheat Sheet

### Essential Azure CLI Commands

```bash
# Identity & Governance
# ⚡ Instant (< 30s)
az role assignment create --role "Contributor" --assignee user@example.com --scope "/subscriptions/id"
az policy assignment create --policy "policy-name" --name "assignment" --scope "/subscriptions/id"
az lock create --name "NoDelete" --resource-group "RG" --lock-type CanNotDelete

# ⏱️ Async (1-5 min)
az account management-group create --name "Finance" --display-name "Finance Dept"

# Storage
# ⚡ Instant
az storage account generate-sas --account-name storage123 --permissions rwdlac --expiry 2024-12-31
az storage blob set-tier --account-name storage123 --container-name container --name blob --tier Cool

# ⏱️ Async
az storage account create --name storage123 --resource-group RG --sku Standard_GRS

# Compute
# ⏳ Long-running (5+ min)
az vm create --resource-group RG --name VM1 --image UbuntuLTS --zone 1
az vmss create --resource-group RG --name ScaleSet --image UbuntuLTS --instance-count 2

# ⏱️ Async
az webapp create --name MyApp --resource-group RG --plan MyPlan --runtime "DOTNET|5.0"

# Networking
# ⏱️ Async
az network vnet create --name VNet1 --resource-group RG --address-prefixes 10.0.0.0/16
az network vnet peering create --name Peer1 --resource-group RG --vnet-name VNet1 --remote-vnet VNet2 --allow-vnet-access

# ⚡ Instant
az network nsg rule create --resource-group RG --nsg-name NSG1 --name Allow80 --priority 100 --access Allow --protocol Tcp --destination-port-ranges 80
```

---

## Failure Patterns & Solutions

|Symptom|Common Cause|Quick Fix|Prevention|
|---|---|---|---|
|"Unauthorized"|Scope mismatch|Check RBAC at resource level|Use --debug flag|
|"Conflict"|Naming collision|Use unique prefix strategy|Check name availability first|
|"InvalidTemplate"|ARM syntax error|Validate JSON structure|Use Bicep for cleaner syntax|
|"QuotaExceeded"|Subscription limits|Request quota increase|Monitor usage dashboard|
|"AllocationFailed"|Capacity issues|Try different region/size|Use availability zones|
|"NetworkSecurityGroupBlocked"|NSG rules|Check effective rules|Use Network Watcher|

---

## Common Exam Traps

### Watch Out For:

1. **VNet Peering** - Non-transitive (A→B→C doesn't work) 🟡
2. **NSG Rules** - Stateful (return traffic auto-allowed) 🟢
3. **Storage Naming** - 3-24 chars, lowercase only, globally unique 🟢
4. **Bastion Subnet** - Must be named 'AzureBastionSubnet', min /26 🟡
5. **Load Balancer** - Standard required for AZ support 🟡
6. **Management Groups** - Max 6 levels deep (root + 5) 🟡
7. **Availability** - Cannot combine Sets and Zones 🟢
8. **Archive Tier** - Requires rehydration before access 🟡

---

## Architecture Synthesis Challenges

### Challenge 1: Multi-Tier Application

**Scenario**: 3-tier app, multi-region, PCI compliance  
**Time Target**: < 60 seconds

Requirements:

- High availability within region
- Disaster recovery across regions
- PCI DSS compliance
- Cost optimization

Solution Pattern

- Front: App Service (Multi-zone) + WAF
- Middle: AKS with node pools across zones
- Data: SQL MI with auto-failover groups
- Security: Private endpoints, NSGs, Azure Policy
- DR: Traffic Manager, geo-replication
- Cost: Reserved capacity, auto-scaling

### Challenge 2: Hybrid Identity

**Scenario**: On-prem AD sync, MFA, privileged access  
**Time Target**: < 45 seconds

Requirements:

- Seamless SSO
- Risk-based access
- Just-in-time admin access
- Audit compliance

Solution Pattern

- Azure AD Connect with PHS/PTA
- Conditional Access policies
- Azure AD PIM for JIT
- Azure AD Identity Protection
- Log Analytics for audit


---

## Final Checklist with Pattern Recognition

### Core Concepts to Verify:

- [ ] Scope hierarchy and inheritance rules 🟢
- [ ] RBAC vs Azure AD roles distinction 🟡
- [ ] Policy effects (Audit, Deny, Deploy, etc.) 🔴
- [ ] Storage redundancy options and durability 🟢
- [ ] SAS token types and use cases 🟡
- [ ] VM sizing families and use cases 🟢
- [ ] Container service differences 🟡
- [ ] NSG rule evaluation order 🟡
- [ ] Load balancer SKU differences 🟡
- [ ] Monitoring metrics vs logs 🟡

### Study Protocol (20 days remaining)

- **Days 1-5**: Rapid traversal focusing on 🟡 and 🔴 items
- **Days 6-10**: Deep dive into complex patterns
- **Days 11-15**: Practice tests with pattern analysis
- **Days 16-19**: Architecture synthesis challenges
- **Day 20**: Pattern review and confidence calibration

---

## Exam Day Reminders

1. **Time Management**: ~150 seconds per question
2. **Flag & Review**: Mark difficult questions for later
3. **Read Carefully**: Look for keywords (minimize, maximize, ensure)
4. **Pattern Recognition**: Map to your neural patterns first
5. **Trust Preparation**: Your assumed mastery is real

**Remember**: You're not learning Azure from scratch - you're mapping Microsoft's terminology to your existing systems architecture knowledge. Each question is a pattern recognition exercise, not a memory test.