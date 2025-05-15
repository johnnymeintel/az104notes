# Configure Secure Access to Virtual Networks

### Objectives

- **Create and configure network security groups (NSGs) and application security groups**
    - **NSG Basics**:
        - Filter network traffic to/from Azure resources
        - Applied at subnet or NIC level
        - Contains security rules with priorities (100-4096)
        - Lower priority numbers = higher precedence
        - Stateful rules (return traffic automatically allowed)
    - **NSG Rule Components**:
        - Name: Unique identifier within NSG
        - Priority: 100-4096 (lower = higher priority)
        - Source/Destination: IP, Service Tag, ASG
        - Protocol: TCP, UDP, ICMP, ESP, AH, or Any
        - Port ranges: Single, range, or comma-separated
        - Action: Allow or Deny
    - **Default Rules** (cannot be deleted):
        - AllowVNetInBound (priority 65000)
        - AllowAzureLoadBalancerInBound (priority 65001)
        - DenyAllInBound (priority 65500)
        - AllowVNetOutBound (priority 65000)
        - AllowInternetOutBound (priority 65001)
        - DenyAllOutBound (priority 65500)
    - **Application Security Groups (ASGs)**:
        - Logical grouping of VMs
        - Simplify complex security rules
        - No IP address management
        - Can only be used within same VNet
        - VMs can belong to multiple ASGs

- **Evaluate effective security rules in NSGs**
    - **Rule Evaluation Order**:
        - Inbound traffic: Subnet NSG → NIC NSG
        - Outbound traffic: NIC NSG → Subnet NSG
        - First matching rule is applied
        - Deny rules stop evaluation
    - **Service Tags**:
        - Predefined groups of IP addresses
        - Managed by Microsoft
        - Examples: Internet, VirtualNetwork, AzureLoadBalancer, Storage
        - Regional service tags: Storage.EastUS
    - **Augmented Security Rules**:
        - Multiple IPs, ports, and service tags in single rule
        - Reduces number of security rules needed
        - Simplifies rule management
    - **NSG Flow Logs**:
        - Record information about IP traffic
        - Stored in Azure Storage account
        - Used for troubleshooting and compliance
        - Integration with Traffic Analytics

- **Implement Azure Bastion**
    - **Service Overview**:
        - Managed PaaS service
        - Provides secure RDP/SSH access
        - No public IP needed on VMs
        - SSL-based connectivity through browser
        - Protection against port scanning
    - **Requirements**:
        - Dedicated subnet named 'AzureBastionSubnet'
        - Minimum subnet size: /26 (64 addresses)
        - Cannot contain other resources
        - Standard public IP required
    - **SKU Tiers**:
        - Basic: Standard RDP/SSH
        - Standard: Additional features (IP-based connection, shareable links)
    - **Security Benefits**:
        - No exposed RDP/SSH ports
        - Protection against zero-day exploits
        - Centralized access management
        - Azure AD integration
        - NSG not required on AzureBastionSubnet

- **Configure service endpoints for Azure platform as a service (PaaS)**
    - **Concept**:
        - Direct connection to PaaS services
        - Traffic remains on Azure backbone
        - Service appears to have private IP
        - Configured per subnet
    - **Supported Services**:
        - Storage (Blob, Table, Queue, File)
        - SQL Database
        - Azure Cosmos DB
        - Key Vault
        - Service Bus
        - Event Hubs
        - App Service
    - **Configuration Steps**:
        - Enable on subnet
        - Configure service to accept VNet traffic
        - Update firewall rules on PaaS service
    - **Limitations**:
        - Service still has public IP
        - Not truly private connection
        - Regional service endpoints only
        - No transitive routing support

- **Configure private endpoints for Azure PaaS**
    - **Key Features**:
        - Private IP address in your VNet
        - Service accessed via private IP
        - Eliminates public endpoint (optional)
        - Uses Azure Private Link
        - Global connectivity (cross-region)
    - **Components**:
        - Private endpoint resource
        - Network interface with private IP
        - Private DNS zone (recommended)
        - Connection to PaaS service
    - **DNS Configuration**:
        - Private DNS zones for name resolution
        - Auto-registration of DNS records
        - Custom DNS server integration
        - Conditional forwarders for hybrid
    - **Benefits over Service Endpoints**:
        - True private connectivity
        - Global reach (cross-region)
        - No data exfiltration risk
        - Granular access control
        - Works with ExpressRoute/VPN

### Common Commands

```bash
# Azure CLI Commands

# Create Network Security Group
az network nsg create \
  --resource-group MyRG \
  --name MyNSG

# Create NSG rule
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name AllowHTTPS \
  --priority 100 \
  --source-address-prefixes Internet \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443 \
  --protocol Tcp \
  --access Allow \
  --direction Inbound

# Create rule with service tag
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name AllowStorage \
  --priority 110 \
  --source-address-prefixes VirtualNetwork \
  --destination-address-prefixes Storage.EastUS \
  --destination-port-ranges '*' \
  --protocol '*' \
  --access Allow \
  --direction Outbound

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name MySubnet \
  --network-security-group MyNSG

# Create Application Security Group
az network asg create \
  --resource-group MyRG \
  --name WebServers

# Create NSG rule with ASG
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name AllowWebToDb \
  --priority 120 \
  --source-asgs WebServers \
  --destination-asgs DatabaseServers \
  --destination-port-ranges 1433 \
  --protocol Tcp \
  --access Allow

# Associate VM NIC with ASG
az network nic update \
  --resource-group MyRG \
  --name MyVM-NIC \
  --application-security-groups WebServers

# Show effective security rules
az network nic list-effective-nsg \
  --resource-group MyRG \
  --name MyVM-NIC

# Create Azure Bastion subnet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name AzureBastionSubnet \
  --address-prefixes 10.0.254.0/24

# Create public IP for Bastion
az network public-ip create \
  --resource-group MyRG \
  --name MyBastionIP \
  --sku Standard \
  --location eastus

# Create Azure Bastion
az network bastion create \
  --resource-group MyRG \
  --name MyBastion \
  --public-ip-address MyBastionIP \
  --vnet-name MyVNet \
  --location eastus

# Enable service endpoint on subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name MySubnet \
  --service-endpoints Microsoft.Storage Microsoft.Sql

# Configure storage account for service endpoints
az storage account network-rule add \
  --resource-group MyRG \
  --account-name mystorageaccount \
  --vnet-name MyVNet \
  --subnet MySubnet

# Create private endpoint for storage
az network private-endpoint create \
  --resource-group MyRG \
  --name MyStoragePrivateEndpoint \
  --vnet-name MyVNet \
  --subnet MySubnet \
  --private-connection-resource-id $(az storage account show --name mystorageaccount --query id -o tsv) \
  --group-id blob \
  --connection-name MyConnection

# Create private DNS zone
az network private-dns zone create \
  --resource-group MyRG \
  --name privatelink.blob.core.windows.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group MyRG \
  --zone-name privatelink.blob.core.windows.net \
  --name MyDNSLink \
  --virtual-network MyVNet \
  --registration-enabled false

# Create DNS record for private endpoint
az network private-endpoint dns-zone-group create \
  --resource-group MyRG \
  --endpoint-name MyStoragePrivateEndpoint \
  --name MyDnsZoneGroup \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name default
```

```powershell
# PowerShell Commands

# Create Network Security Group
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName MyRG `
  -Name MyNSG `
  -Location eastus

# Create NSG rule
$rule = New-AzNetworkSecurityRuleConfig `
  -Name AllowHTTPS `
  -Priority 100 `
  -Direction Inbound `
  -Access Allow `
  -Protocol Tcp `
  -SourceAddressPrefix Internet `
  -SourcePortRange '*' `
  -DestinationAddressPrefix '*' `
  -DestinationPortRange 443

$nsg.SecurityRules.Add($rule)
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# Create rule with service tag
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name AllowStorage `
  -Priority 110 `
  -Direction Outbound `
  -Access Allow `
  -Protocol '*' `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange '*' `
  -DestinationAddressPrefix Storage.EastUS `
  -DestinationPortRange '*'

# Associate NSG with subnet
$vnet = Get-AzVirtualNetwork -Name MyVNet -ResourceGroupName MyRG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name MySubnet -VirtualNetwork $vnet
$subnet.NetworkSecurityGroup = $nsg
$vnet | Set-AzVirtualNetwork

# Create Application Security Group
$asg = New-AzApplicationSecurityGroup `
  -ResourceGroupName MyRG `
  -Name WebServers `
  -Location eastus

# Create NSG rule with ASG
$webAsg = Get-AzApplicationSecurityGroup -Name WebServers -ResourceGroupName MyRG
$dbAsg = Get-AzApplicationSecurityGroup -Name DatabaseServers -ResourceGroupName MyRG

$asgRule = New-AzNetworkSecurityRuleConfig `
  -Name AllowWebToDb `
  -Priority 120 `
  -Direction Inbound `
  -Access Allow `
  -Protocol Tcp `
  -SourceApplicationSecurityGroup $webAsg `
  -SourcePortRange '*' `
  -DestinationApplicationSecurityGroup $dbAsg `
  -DestinationPortRange 1433

# Get effective security rules
Get-AzEffectiveNetworkSecurityGroup `
  -ResourceGroupName MyRG `
  -NetworkInterfaceName MyVM-NIC

# Create Bastion subnet
$bastionSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name AzureBastionSubnet `
  -AddressPrefix 10.0.254.0/24

$vnet.Subnets.Add($bastionSubnet)
$vnet | Set-AzVirtualNetwork

# Create Bastion
$publicip = New-AzPublicIpAddress `
  -ResourceGroupName MyRG `
  -Name MyBastionIP `
  -Location eastus `
  -Sku Standard `
  -AllocationMethod Static

New-AzBastion `
  -ResourceGroupName MyRG `
  -Name MyBastion `
  -PublicIpAddress $publicip `
  -VirtualNetworkName MyVNet

# Enable service endpoints
$subnet = Get-AzVirtualNetworkSubnetConfig -Name MySubnet -VirtualNetwork $vnet
$subnet.ServiceEndpoints = @("Microsoft.Storage", "Microsoft.Sql")
$vnet | Set-AzVirtualNetwork

# Configure storage for service endpoints
$storageAccount = Get-AzStorageAccount -ResourceGroupName MyRG -Name mystorageaccount
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName MyRG `
  -Name mystorageaccount `
  -VirtualNetworkResourceId $subnet.Id

# Create private endpoint
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
  -Name MyConnection `
  -PrivateLinkServiceId $storageAccount.Id `
  -GroupId blob

$privateEndpoint = New-AzPrivateEndpoint `
  -ResourceGroupName MyRG `
  -Name MyStoragePrivateEndpoint `
  -Location eastus `
  -Subnet $subnet `
  -PrivateLinkServiceConnection $privateEndpointConnection

# Create private DNS zone
$zone = New-AzPrivateDnsZone `
  -ResourceGroupName MyRG `
  -Name privatelink.blob.core.windows.net

# Link DNS zone to VNet
$link = New-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName MyRG `
  -ZoneName privatelink.blob.core.windows.net `
  -Name MyDNSLink `
  -VirtualNetworkId $vnet.Id
```

### Additional Information

#### NSG vs ASG Comparison

|Feature|Network Security Group|Application Security Group|
|---|---|---|
|Purpose|Filter network traffic|Group VMs for security rules|
|Scope|Subnet or NIC level|Logical grouping|
|IP Management|Required|Not required|
|Cross-VNet|Yes|No|
|Rule Complexity|Can be complex|Simplifies rules|

#### Service Endpoints vs Private Endpoints

|Feature|Service Endpoints|Private Endpoints|
|---|---|---|
|IP Address|Public IP|Private IP in VNet|
|Network Path|Azure backbone|Private connection|
|DNS|Public DNS|Private DNS zones|
|Cost|Free|Per hour + data|
|Region|Same region only|Cross-region|
|Data Exfiltration|Possible|Not possible|

#### Azure Bastion Deployment Checklist

1. Create dedicated subnet (/26 or larger)
2. Name subnet exactly 'AzureBastionSubnet'
3. Create Standard SKU public IP
4. Deploy Bastion service
5. Configure NSGs on target VMs (if needed)
6. Test connectivity through Azure Portal

#### Common Security Patterns

1. **DMZ Pattern**: Web servers in DMZ subnet with restricted backend access
2. **Hub-Spoke Security**: Centralized NVA with spoke NSGs
3. **Micro-segmentation**: ASGs for granular application tiers
4. **Zero Trust**: Private endpoints with no public access
5. **Jump Box Alternative**: Bastion replacing traditional jump boxes

#### Private Endpoint DNS Zones

|Service|DNS Zone Name|
|---|---|
|Blob Storage|privatelink.blob.core.windows.net|
|File Storage|privatelink.file.core.windows.net|
|Queue Storage|privatelink.queue.core.windows.net|
|Table Storage|privatelink.table.core.windows.net|
|SQL Database|privatelink.database.windows.net|
|Cosmos DB|privatelink.documents.azure.com|
|Key Vault|privatelink.vaultcore.azure.net|

#### Troubleshooting Security Access

1. Check effective NSG rules
2. Verify ASG membership
3. Validate service endpoint configuration
4. Test private endpoint DNS resolution
5. Review Bastion connectivity logs
6. Use Network Watcher for detailed analysis