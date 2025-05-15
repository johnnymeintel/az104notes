# Deploy and manage Azure compute resources (20–25%)

### Objective Overview

Refer to individual objective pages for details:

- [[01 - Automate deployment of resources by using Azure Resource Manager (ARM) templates or Bicep files]]
- [[02 - Create and configure virtual machines]]
- [[03 - Provision and manage containers in the Azure portal]]
- [[04 - Create and configure Azure App Service]]

---

### Key Concepts

- Infrastructure as Code (IaC) - Managing infrastructure through declarative configuration files (ARM/Bicep)
- Idempotency - Templates produce same result regardless of how many times they're deployed
- High Availability - Using availability sets/zones to protect against failures
- Containerization - Packaging applications with dependencies for consistent deployment
- Platform as a Service (PaaS) - Managed application hosting without infrastructure management

### Relevant Terms & Definitions

|Term|Definition|Portal Location|
|---|---|---|
|ARM Template|JSON files that define Azure resources declaratively|Home > All Resources > Create a resource > Template deployment|
|Bicep|Domain-specific language for deploying Azure resources, compiles to ARM|Azure CLI/Cloud Shell for deployment|
|Availability Set|Logical grouping ensuring VMs are distributed across fault/update domains|VM creation > Availability options|
|Availability Zone|Physically separate locations within an Azure region|VM creation > Availability options|
|VM Scale Set|Automatically scaling group of identical VMs|Home > Create a resource > Virtual machine scale set|
|Container Registry|Private Docker registry for container images|Home > Create a resource > Container Registry|
|App Service Plan|Defines compute resources for App Service|Home > Create a resource > App Service Plan|
|Deployment Slot|Separate instance of App Service for staging|App Service > Deployment slots|

### Service Comparison

|Feature|Virtual Machines|Container Instances|Container Apps|App Service|
|---|---|---|---|---|
|Type|IaaS|CaaS|CaaS|PaaS|
|Control Level|Full OS control|Container level|Container orchestration|Application level|
|Scaling|Manual/VMSS|Manual|Automatic|Automatic|
|Pricing Model|Per VM hour|Per second|Per request/resource|Per plan hour|
|Use Case|Full control needed|Quick container deployment|Microservices|Web apps|
|Startup Time|Minutes|Seconds|Seconds|Seconds|

---

### Azure CLI Commands

```bash
# ARM/Bicep Deployment
az deployment group create --resource-group MyRG --template-file template.json
az deployment group create --resource-group MyRG --template-file main.bicep

# Virtual Machine
az vm create --resource-group MyRG --name MyVM --image UbuntuLTS
az vm resize --resource-group MyRG --name MyVM --size Standard_DS3_v2

# Container Registry
az acr create --resource-group MyRG --name myregistry --sku Basic
az acr build --registry myregistry --image myapp:v1 .

# App Service
az appservice plan create --name MyPlan --resource-group MyRG --sku B1
az webapp create --name MyApp --resource-group MyRG --plan MyPlan
```

### PowerShell Commands

```powershell
# ARM/Bicep Deployment
New-AzResourceGroupDeployment -ResourceGroupName MyRG -TemplateFile template.json
New-AzResourceGroupDeployment -ResourceGroupName MyRG -TemplateFile main.bicep

# Virtual Machine
New-AzVM -ResourceGroupName MyRG -Name MyVM -Image UbuntuLTS
Update-AzVM -ResourceGroupName MyRG -VM $vm

# Container Registry
New-AzContainerRegistry -ResourceGroupName MyRG -Name myregistry -Sku Basic

# App Service
New-AzAppServicePlan -ResourceGroupName MyRG -Name MyPlan -Tier Basic
New-AzWebApp -ResourceGroupName MyRG -Name MyApp -AppServicePlan MyPlan
```

---

### Cost Considerations

- VM costs include compute, storage, networking, and licensing
- Container Instances charge per second of execution
- App Service plans have fixed monthly costs regardless of apps deployed
- VM Scale Sets can reduce costs through automatic scaling
- Reserved instances provide up to 72% savings for predictable workloads

### Security Best Practices

- Enable Azure Disk Encryption for VMs at rest
- Use managed identities instead of storing credentials
- Implement network security groups and application security groups
- Store container images in private registries
- Enable HTTPS and use managed certificates for App Services

### Business Use Cases

- **VMs**: Legacy applications, custom software requiring specific OS
- **Container Instances**: Batch jobs, build agents, short-lived tasks
- **Container Apps**: Microservices, API backends, event-driven apps
- **App Service**: Corporate websites, web APIs, mobile backends
- **VMSS**: Auto-scaling web tiers, big data processing, batch computing

### Real-World Analogies

- **ARM Templates**: Building blueprints that contractors follow exactly
- **Availability Sets**: Eggs in different baskets within same room
- **Availability Zones**: Eggs in different buildings within same city
- **Container Registry**: Private warehouse for shipping containers
- **Deployment Slots**: Dress rehearsal stage before main performance

### Common Pitfalls & Misconceptions

- Confusing availability sets (rack-level) with availability zones (datacenter-level)
- Not understanding that Container Instances are for short-lived workloads
- Assuming App Service is only for web apps (also supports APIs, mobile backends)
- Forgetting that VM disks need separate backup strategy from VM itself
- Not realizing Bicep is Azure-specific while ARM works with other tools

### Things to Look Out for on the Exam

- Difference between fault domains and update domains in availability sets
- When to use availability sets vs availability zones
- Understanding App Service plan tiers and their limitations
- Container registry SKUs and their features (Basic, Standard, Premium)
- Know that moving VMs between regions requires recreation
- Bicep is more readable than ARM but compiles to ARM
- Deployment slots are only available in Standard tier and above
- VM Scale Sets require load balancer or application gateway

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





---

#### Project Ideas

- 
