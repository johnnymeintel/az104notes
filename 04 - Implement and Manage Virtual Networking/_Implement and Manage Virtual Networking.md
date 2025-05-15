# Implement and manage virtual networking (15–20%)

### Objective Overview

Refer to individual objective pages for details:

- [[01 - Configure and manage virtual networks in Azure]]
- [[02 - Configure secure access to virtual networks]]
- [[03 - Configure name resolution and load balancing]]

---

### Key Concepts

- Network segmentation using VNets and subnets for isolation and security
- Hub-spoke network topology for centralized management and shared services
- Zero Trust security model with NSGs and private endpoints
- High availability through load balancing and multiple availability zones
- Hybrid connectivity options for on-premises integration

### Relevant Terms & Definitions

|Term|Definition|Portal Location|
|---|---|---|
|Virtual Network (VNet)|Isolated network in Azure cloud|Home > Virtual networks|
|Subnet|IP address range within a VNet|VNet > Subnets|
|Network Security Group (NSG)|Firewall rules for network traffic|Home > Network security groups|
|Application Security Group (ASG)|Logical grouping of VMs for security rules|Home > Application security groups|
|VNet Peering|Direct connection between VNets|VNet > Peerings|
|Service Endpoint|Direct route to Azure PaaS services|Subnet > Service endpoints|
|Private Endpoint|Private IP for Azure PaaS service|Home > Private endpoints|
|Azure Bastion|Secure RDP/SSH without public IPs|Home > Bastions|
|Load Balancer|Distributes traffic across VMs|Home > Load balancers|
|Azure DNS|Name resolution service|Home > DNS zones|

### Service Comparison

|Feature|Service Endpoint|Private Endpoint|NAT Gateway|
|---|---|---|---|
|Purpose|Secure PaaS access|Private PaaS access|Outbound connectivity|
|IP Type|Public service IP|Private IP in VNet|Public IP for outbound|
|Cost|Free|Per hour + data|Per hour + data|
|Security|Network-level|Application-level|Outbound only|
|Use Case|Simple security|Compliance needs|SNAT prevention|

|Feature|Basic Load Balancer|Standard Load Balancer|Application Gateway|
|---|---|---|---|
|Layer|4 (Transport)|4 (Transport)|7 (Application)|
|Availability Zones|No|Yes|Yes|
|SLA|No SLA|99.99%|99.95%|
|Backend Pool Size|300|1000|100|
|Health Probes|Basic|Advanced|HTTP/HTTPS|
|Cost|Free|Per rule + data|Per hour + data|

---

### Azure CLI Commands

```bash
# Virtual Network and Subnet
az network vnet create --resource-group MyRG --name MyVNet --address-prefixes 10.0.0.0/16
az network vnet subnet create --resource-group MyRG --vnet-name MyVNet --name MySubnet --address-prefixes 10.0.1.0/24

# VNet Peering
az network vnet peering create --resource-group MyRG --name VNet1ToVNet2 --vnet-name VNet1 --remote-vnet VNet2 --allow-vnet-access

# Network Security Group
az network nsg create --resource-group MyRG --name MyNSG
az network nsg rule create --resource-group MyRG --nsg-name MyNSG --name AllowSSH --priority 100 --source-address-prefixes Internet --destination-port-ranges 22 --access Allow --protocol Tcp

# Azure Bastion
az network bastion create --resource-group MyRG --name MyBastion --vnet-name MyVNet --public-ip-address MyBastionIP

# Load Balancer
az network lb create --resource-group MyRG --name MyLoadBalancer --sku Standard --public-ip-address MyPublicIP
```

### PowerShell Commands

```powershell
# Virtual Network and Subnet
$vnet = New-AzVirtualNetwork -ResourceGroupName MyRG -Name MyVNet -AddressPrefix 10.0.0.0/16 -Location eastus
Add-AzVirtualNetworkSubnetConfig -Name MySubnet -AddressPrefix 10.0.1.0/24 -VirtualNetwork $vnet | Set-AzVirtualNetwork

# VNet Peering
Add-AzVirtualNetworkPeering -Name VNet1ToVNet2 -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id

# Network Security Group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName MyRG -Name MyNSG -Location eastus
$nsg | Add-AzNetworkSecurityRuleConfig -Name AllowRDP -Priority 100 -Access Allow -Protocol Tcp -Direction Inbound -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 | Set-AzNetworkSecurityGroup

# Load Balancer
$lb = New-AzLoadBalancer -ResourceGroupName MyRG -Name MyLoadBalancer -Location eastus -Sku Standard
```

---

### Cost Considerations

- VNet peering charges for data transfer between VNets (ingress free, egress charged)
- Azure Bastion pricing includes hourly charge plus data processing fees
- Standard Load Balancer has rule charges plus data processing costs
- Private endpoints incur hourly charges plus data processing fees
- NAT Gateway charges include hourly rate and data processing

### Security Best Practices

- Implement defense in depth with NSGs at both subnet and NIC levels
- Use Application Security Groups to simplify complex security rules
- Deploy Azure Bastion instead of public IPs for management access
- Prefer Private Endpoints over Service Endpoints for sensitive data
- Regular security rule audits and cleanup of unused rules

### Business Use Cases

- **Hub-spoke topology**: Centralized services with departmental VNets
- **Multi-tier applications**: Web, app, and database tiers in separate subnets
- **Hybrid connectivity**: Secure connection to on-premises networks
- **PaaS security**: Private connectivity to storage, SQL, and other services
- **High availability**: Load balanced applications across availability zones

### Real-World Analogies

- **VNet**: Private office building with controlled access
- **Subnet**: Different floors or departments within the building
- **NSG**: Security guards with specific access lists
- **Load Balancer**: Reception desk directing visitors to available staff
- **VNet Peering**: Private hallway between two office buildings

### Common Pitfalls & Misconceptions

- Forgetting that NSG rules are stateful (return traffic automatically allowed)
- Not understanding that VNet peering is non-transitive
- Assuming Basic Load Balancer supports availability zones (it doesn't)
- Overlooking that some resources require dedicated subnets (Bastion, Gateway)
- Confusing Service Endpoints (network path) with Private Endpoints (private IP)

### Things to Look Out for on the Exam

- Understand the difference between NSG priority numbers (lower = higher priority)
- Know which Azure services support Service Endpoints vs Private Endpoints
- Remember that Azure Bastion requires a specific subnet named 'AzureBastionSubnet'
- Standard Load Balancer is required for availability zones and cross-zone scenarios
- VNet peering must be configured in both directions for bidirectional communication
- Application Security Groups can only be used within the same VNet
- Default NSG rules cannot be deleted but can be overridden
- Know the address ranges reserved by Azure in each subnet (first 4 and last 1)

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