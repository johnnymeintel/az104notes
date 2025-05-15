# Configure and Manage Virtual Networks in Azure

### Objectives

- **Create and configure virtual networks and subnets**
    - **VNet Planning**:
        - Address space planning (CIDR notation)
        - Non-overlapping ranges for connectivity
        - RFC 1918 private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
        - Consider future growth and peering requirements
    - **Subnet Configuration**:
        - Subnet size calculation (Azure reserves 5 IPs per subnet)
        - Reserved addresses: Network, Default gateway, Azure DNS (2), Broadcast
        - Special purpose subnets: GatewaySubnet, AzureBastionSubnet, AzureFirewallSubnet
        - Subnet delegation for specific Azure services
    - **VNet Features**:
        - DNS settings (Azure-provided or custom)
        - DDoS protection (Basic or Standard)
        - VM protection (prevent accidental deletion)
        - Encryption (VNet encryption for VM-to-VM traffic)
    - **Regional Considerations**:
        - VNets are regional resources
        - Cannot span multiple regions
        - Global VNet peering for cross-region connectivity

- **Create and configure virtual network peering**
    - **Peering Types**:
        - Regional VNet peering (same region)
        - Global VNet peering (cross-region)
    - **Peering Properties**:
        - Allow forwarded traffic
        - Allow gateway transit
        - Use remote gateways
        - Allow virtual network access
    - **Requirements and Limitations**:
        - Non-overlapping address spaces
        - Peering is non-transitive
        - Must be configured bidirectionally
        - Cannot be created if VNets have overlapping address space
    - **Common Scenarios**:
        - Hub-spoke topology
        - Cross-subscription connectivity
        - Cross-tenant peering (requires permissions)
        - Service chaining with UDRs

- **Configure public IP addresses**
    - **SKU Types**:
        - Basic: Dynamic or static, open by default
        - Standard: Always static, secure by default, zone-redundant
    - **Allocation Methods**:
        - Dynamic: Assigned when resource starts
        - Static: Reserved immediately, never changes
    - **IP Versions**:
        - IPv4: Standard addressing
        - IPv6: Dual-stack scenarios
        - IPv4+IPv6: Both protocols on same resource
    - **Associated Resources**:
        - Virtual machine NICs
        - Load balancers
        - VPN gateways
        - Application gateways
        - Bastion hosts
    - **DNS and Domains**:
        - DNS name label (optional)
        - FQDN: label.region.cloudapp.azure.com
        - Reverse DNS support

- **Configure user-defined network routes**
    - **Route Types**:
        - System routes (automatic)
        - User-defined routes (UDR)
        - BGP routes (from on-premises)
    - **Next Hop Types**:
        - Virtual network gateway
        - Virtual network (VNet)
        - Internet
        - Virtual appliance (NVA)
        - None (drop traffic)
    - **Route Priority**:
        - Most specific route wins
        - User routes override system routes
        - BGP routes have lowest priority
    - **Common Use Cases**:
        - Force tunneling to on-premises
        - Route through network virtual appliance
        - Isolate subnet traffic
        - Service chaining
    - **Route Table Management**:
        - Associate with subnets
        - Multiple subnets can share route table
        - Each subnet can have only one route table

- **Troubleshoot network connectivity**
    - **Diagnostic Tools**:
        - Network Watcher
        - Connection Monitor
        - IP flow verify
        - Next hop
        - VPN diagnostics
        - NSG flow logs
    - **Common Issues**:
        - NSG rules blocking traffic
        - Missing or incorrect routes
        - Peering misconfiguration
        - DNS resolution problems
        - Asymmetric routing
    - **Troubleshooting Steps**:
        - Verify NSG rules (inbound and outbound)
        - Check effective routes
        - Test connectivity between VMs
        - Validate DNS resolution
        - Review VNet peering status
    - **Network Watcher Features**:
        - Packet capture
        - Connection troubleshoot
        - Topology view
        - Security group view
        - NSG diagnostics

### Common Commands

```bash
# Azure CLI Commands

# Create Virtual Network
az network vnet create \
  --name MyVNet \
  --resource-group MyRG \
  --location eastus \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name MySubnet \
  --subnet-prefixes 10.0.1.0/24

# Add additional subnet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --address-prefixes 10.0.2.0/24

# Create subnet with delegation
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name AppServiceSubnet \
  --address-prefixes 10.0.3.0/24 \
  --delegations Microsoft.Web/serverFarms

# Update VNet DNS servers
az network vnet update \
  --resource-group MyRG \
  --name MyVNet \
  --dns-servers 10.0.1.4 10.0.1.5

# Create VNet peering (local)
az network vnet peering create \
  --resource-group MyRG \
  --name VNet1ToVNet2 \
  --vnet-name VNet1 \
  --remote-vnet VNet2 \
  --allow-vnet-access true \
  --allow-forwarded-traffic true

# Create global VNet peering
az network vnet peering create \
  --resource-group MyRG \
  --name VNet1ToVNet3 \
  --vnet-name VNet1 \
  --remote-vnet /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/VNet3 \
  --allow-vnet-access true

# Create public IP address
az network public-ip create \
  --resource-group MyRG \
  --name MyPublicIP \
  --sku Standard \
  --allocation-method Static \
  --zone 1 2 3 \
  --dns-name myapp

# Create public IP with IPv6
az network public-ip create \
  --resource-group MyRG \
  --name MyPublicIPv6 \
  --sku Standard \
  --allocation-method Static \
  --version IPv6

# Create route table
az network route-table create \
  --resource-group MyRG \
  --name MyRouteTable

# Add route to route table
az network route-table route create \
  --resource-group MyRG \
  --route-table-name MyRouteTable \
  --name ToOnPrem \
  --address-prefix 10.1.0.0/16 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.2.4

# Associate route table with subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name MySubnet \
  --route-table MyRouteTable

# Enable Network Watcher
az network watcher configure \
  --resource-group MyRG \
  --locations eastus \
  --enabled

# Run connectivity check
az network watcher test-connectivity \
  --resource-group MyRG \
  --source-resource MyVM1 \
  --dest-resource MyVM2 \
  --protocol TCP \
  --dest-port 80

# Check effective routes
az network nic show-effective-route-table \
  --resource-group MyRG \
  --name MyVM-NIC

# IP flow verify
az network watcher test-ip-flow \
  --resource-group MyRG \
  --vm MyVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:80 \
  --remote 10.0.2.4:5000

# Show next hop
az network watcher show-next-hop \
  --resource-group MyRG \
  --vm MyVM \
  --source-ip 10.0.1.4 \
  --dest-ip 20.0.1.4
```

```powershell
# PowerShell Commands

# Create Virtual Network
$vnet = New-AzVirtualNetwork `
  -Name MyVNet `
  -ResourceGroupName MyRG `
  -Location eastus `
  -AddressPrefix 10.0.0.0/16

# Add subnet to VNet
Add-AzVirtualNetworkSubnetConfig `
  -Name MySubnet `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork

# Create subnet with delegation
$subnet = New-AzVirtualNetworkSubnetConfig `
  -Name AppServiceSubnet `
  -AddressPrefix 10.0.3.0/24 `
  -Delegation (New-AzDelegation `
    -Name "myDelegation" `
    -ServiceName "Microsoft.Web/serverFarms")

# Update DNS servers
$vnet.DhcpOptions.DnsServers = @("10.0.1.4", "10.0.1.5")
$vnet | Set-AzVirtualNetwork

# Create VNet peering
Add-AzVirtualNetworkPeering `
  -Name VNet1ToVNet2 `
  -VirtualNetwork $vnet1 `
  -RemoteVirtualNetworkId $vnet2.Id `
  -AllowForwardedTraffic `
  -AllowVirtualNetworkAccess

# Create global peering
Add-AzVirtualNetworkPeering `
  -Name VNet1ToVNet3 `
  -VirtualNetwork $vnet1 `
  -RemoteVirtualNetworkId "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/VNet3"

# Create public IP
$publicIP = New-AzPublicIpAddress `
  -Name MyPublicIP `
  -ResourceGroupName MyRG `
  -Location eastus `
  -Sku Standard `
  -AllocationMethod Static `
  -DomainNameLabel myapp `
  -Zone 1,2,3

# Create route table
$routeTable = New-AzRouteTable `
  -Name MyRouteTable `
  -ResourceGroupName MyRG `
  -Location eastus

# Add route
Add-AzRouteConfig `
  -Name ToOnPrem `
  -RouteTable $routeTable `
  -AddressPrefix 10.1.0.0/16 `
  -NextHopType VirtualAppliance `
  -NextHopIpAddress 10.0.2.4 | Set-AzRouteTable

# Associate route table with subnet
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name MySubnet `
  -VirtualNetwork $vnet
$subnet.RouteTable = $routeTable
$vnet | Set-AzVirtualNetwork

# Enable Network Watcher
New-AzNetworkWatcher `
  -Name NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG `
  -Location eastus

# Test connectivity
Test-AzNetworkWatcherConnectivity `
  -ResourceGroupName MyRG `
  -NetworkWatcherName NetworkWatcher_eastus `
  -SourceId $vm1.Id `
  -DestinationId $vm2.Id `
  -Protocol TCP `
  -DestinationPort 80

# Get effective routes
Get-AzEffectiveRouteTable `
  -ResourceGroupName MyRG `
  -NetworkInterfaceName MyVM-NIC

# Test IP flow
Test-AzNetworkWatcherIPFlow `
  -ResourceGroupName MyRG `
  -NetworkWatcherName NetworkWatcher_eastus `
  -VMName MyVM `
  -Direction Inbound `
  -Protocol TCP `
  -LocalIPAddress 10.0.1.4 `
  -LocalPort 80 `
  -RemoteIPAddress 10.0.2.4 `
  -RemotePort 5000

# Get next hop
Get-AzNetworkWatcherNextHop `
  -ResourceGroupName MyRG `
  -NetworkWatcherName NetworkWatcher_eastus `
  -VMName MyVM `
  -SourceIPAddress 10.0.1.4 `
  -DestinationIPAddress 20.0.1.4
```

### Additional Information

#### Address Space Best Practices

- Use large address spaces for VNets (e.g., /16)
- Plan for growth and future peering
- Avoid overlapping with on-premises networks
- Document IP address allocation
- Consider using separate ranges for different environments

#### VNet Peering Decision Matrix

|Scenario|Same Region|Cross Region|Cross Subscription|Cross Tenant|
|---|---|---|---|---|
|Latency|Lowest|Higher|Same as region|Same as region|
|Cost|Data transfer|Higher charges|Same pricing|Same pricing|
|Setup|Simple|Simple|Needs permissions|Needs app registration|

#### Public IP SKU Comparison

|Feature|Basic SKU|Standard SKU|
|---|---|---|
|Allocation|Dynamic/Static|Static only|
|Security|Open by default|Secure by default|
|Availability Zones|Not supported|Zone-redundant|
|Load Balancer|Basic LB only|Standard LB|
|NAT Gateway|Not supported|Supported|

#### Common Route Scenarios

1. **Force Tunneling**: Route all internet traffic through on-premises
2. **Hub-Spoke**: Route spoke traffic through hub NVA
3. **Service Chaining**: Route through multiple NVAs
4. **Internet Breakout**: Direct internet access from specific subnets
5. **Subnet Isolation**: Prevent communication between subnets

#### Troubleshooting Checklist

1. Check NSG rules at both NIC and subnet level
2. Verify effective routes on source and destination
3. Validate VNet peering status and configuration
4. Test DNS resolution
5. Use Network Watcher tools for diagnosis
6. Check for asymmetric routing issues
7. Verify service endpoints/private endpoints configuration