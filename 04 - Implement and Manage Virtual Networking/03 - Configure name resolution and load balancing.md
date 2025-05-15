# Configure Name Resolution and Load Balancing

### Objectives

- **Configure Azure DNS**
    - **Public DNS Zones**:
        - Host DNS records for internet-facing domains
        - Authoritative DNS service
        - Automatic failover with Traffic Manager
        - Support for DNSSEC (preview)
        - Global anycast network
    - **Private DNS Zones**:
        - Name resolution within virtual networks
        - No internet exposure
        - Auto-registration of VM records
        - Split-horizon DNS scenarios
        - Works with VNet peering
    - **DNS Record Types**:
        - A: IPv4 address
        - AAAA: IPv6 address
        - CNAME: Canonical name (alias)
        - MX: Mail exchange
        - NS: Name server
        - PTR: Reverse lookup
        - SOA: Start of authority
        - SRV: Service location
        - TXT: Text records
        - CAA: Certificate authority authorization
    - **DNS Zone Delegation**:
        - Parent zone points to Azure DNS name servers
        - NS records in parent domain
        - Azure provides 4 name servers per zone
        - TTL (Time to Live) configuration

- **Configure an internal or public load balancer**
    - **Load Balancer Types**:
        - Public: Internet-facing with public IP
        - Internal: Private IP for internal traffic
        - Basic SKU: Free, limited features
        - Standard SKU: SLA, availability zones, secure by default
    - **Frontend Configuration**:
        - Public or private IP address
        - Multiple frontends supported
        - Zone-redundant (Standard SKU)
        - Static IP allocation
    - **Backend Pools**:
        - Virtual machines or VMSS instances
        - IP-based backends (Standard SKU)
        - Availability set or zone requirements
        - Cross-region load balancing (preview)
    - **Health Probes**:
        - HTTP/HTTPS: Custom path and status codes
        - TCP: Port connectivity check
        - Probe interval and unhealthy threshold
        - Used to determine backend health
    - **Load Balancing Rules**:
        - Protocol: TCP or UDP
        - Frontend port and backend port
        - Session persistence (source IP affinity)
        - Idle timeout settings
        - Floating IP (Direct Server Return)
    - **Inbound NAT Rules**:
        - Port forwarding to specific VMs
        - RDP/SSH access through LB
        - Automatic rules for VMSS
    - **Outbound Rules** (Standard SKU):
        - SNAT port allocation
        - Multiple public IPs for scale
        - Idle timeout configuration

- **Troubleshoot load balancing**
    - **Common Issues**:
        - Health probe failures
        - Backend pool misconfiguration
        - NSG blocking probe/traffic
        - Asymmetric routing
        - SNAT port exhaustion
    - **Diagnostic Tools**:
        - Load balancer metrics
        - Health probe status
        - SNAT connection metrics
        - Azure Monitor insights
        - NSG flow logs
    - **Health Probe Troubleshooting**:
        - Verify probe port accessibility
        - Check application response
        - Validate NSG rules
        - Review probe configuration
        - Test from Load Balancer subnet
    - **Traffic Flow Analysis**:
        - Verify frontend IP configuration
        - Check backend pool membership
        - Validate load balancing rules
        - Review session persistence
        - Check NAT rule configuration
    - **SNAT Troubleshooting**:
        - Monitor SNAT port usage
        - Configure outbound rules
        - Add public IPs for more ports
        - Use instance-level public IPs
        - Implement connection pooling

### Common Commands

```bash
# Azure CLI Commands

# Create public DNS zone
az network dns zone create \
  --resource-group MyRG \
  --name contoso.com

# Create private DNS zone
az network private-dns zone create \
  --resource-group MyRG \
  --name contoso.internal

# Add DNS records
az network dns record-set a create \
  --resource-group MyRG \
  --zone-name contoso.com \
  --name www

az network dns record-set a add-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name www \
  --ipv4-address 20.30.40.50

# Create CNAME record
az network dns record-set cname create \
  --resource-group MyRG \
  --zone-name contoso.com \
  --name mail

az network dns record-set cname set-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name mail \
  --cname mail.contoso.com

# Link private DNS zone to VNet
az network private-dns link vnet create \
  --resource-group MyRG \
  --zone-name contoso.internal \
  --name MyVNetLink \
  --virtual-network MyVNet \
  --registration-enabled true

# Create public load balancer
az network lb create \
  --resource-group MyRG \
  --name MyPublicLB \
  --sku Standard \
  --location eastus \
  --public-ip-address MyPublicIP \
  --frontend-ip-name MyFrontend \
  --backend-pool-name MyBackendPool

# Create internal load balancer
az network lb create \
  --resource-group MyRG \
  --name MyInternalLB \
  --sku Standard \
  --location eastus \
  --vnet-name MyVNet \
  --subnet MySubnet \
  --frontend-ip-name MyFrontend \
  --backend-pool-name MyBackendPool \
  --private-ip-address 10.0.1.10

# Create health probe
az network lb probe create \
  --resource-group MyRG \
  --lb-name MyPublicLB \
  --name MyHealthProbe \
  --protocol Http \
  --port 80 \
  --path /health \
  --interval 15 \
  --threshold 2

# Create load balancing rule
az network lb rule create \
  --resource-group MyRG \
  --lb-name MyPublicLB \
  --name MyLBRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name MyFrontend \
  --backend-pool-name MyBackendPool \
  --probe-name MyHealthProbe \
  --idle-timeout 15 \
  --enable-tcp-reset true

# Create inbound NAT rule
az network lb inbound-nat-rule create \
  --resource-group MyRG \
  --lb-name MyPublicLB \
  --name MyNATRule \
  --protocol Tcp \
  --frontend-port 3389 \
  --backend-port 3389 \
  --frontend-ip-name MyFrontend

# Create outbound rule (Standard SKU)
az network lb outbound-rule create \
  --resource-group MyRG \
  --lb-name MyPublicLB \
  --name MyOutboundRule \
  --protocol All \
  --frontend-ip-configs MyFrontend \
  --address-pool MyBackendPool \
  --allocated-outbound-ports 10000 \
  --idle-timeout 15

# Add VM to backend pool
az network nic ip-config address-pool add \
  --resource-group MyRG \
  --nic-name MyVM-NIC \
  --ip-config-name ipconfig1 \
  --lb-name MyPublicLB \
  --address-pool MyBackendPool

# Show load balancer metrics
az monitor metrics list \
  --resource $(az network lb show --name MyPublicLB --resource-group MyRG --query id -o tsv) \
  --metric "HealthProbeStatus" \
  --interval PT1M

# Show health probe status
az network lb show \
  --resource-group MyRG \
  --name MyPublicLB \
  --query "probes[0].provisioningState"

# List backend pool members
az network lb address-pool show \
  --resource-group MyRG \
  --lb-name MyPublicLB \
  --name MyBackendPool \
  --query backendIpConfigurations[].id
```

```powershell
# PowerShell Commands

# Create public DNS zone
New-AzDnsZone `
  -Name contoso.com `
  -ResourceGroupName MyRG

# Create private DNS zone
New-AzPrivateDnsZone `
  -Name contoso.internal `
  -ResourceGroupName MyRG

# Add DNS A record
New-AzDnsRecordSet `
  -Name www `
  -RecordType A `
  -ZoneName contoso.com `
  -ResourceGroupName MyRG `
  -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -IPv4Address "20.30.40.50")

# Add CNAME record
New-AzDnsRecordSet `
  -Name mail `
  -RecordType CNAME `
  -ZoneName contoso.com `
  -ResourceGroupName MyRG `
  -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -Cname "mail.contoso.com")

# Link private DNS zone to VNet
$vnet = Get-AzVirtualNetwork -Name MyVNet -ResourceGroupName MyRG
New-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName MyRG `
  -ZoneName contoso.internal `
  -Name MyVNetLink `
  -VirtualNetwork $vnet `
  -EnableAutoRegistration

# Create public load balancer
$publicIP = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name MyFrontend `
  -PublicIpAddress $publicIP

$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name MyBackendPool

New-AzLoadBalancer `
  -ResourceGroupName MyRG `
  -Name MyPublicLB `
  -Location eastus `
  -Sku Standard `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool

# Create internal load balancer
$vnet = Get-AzVirtualNetwork -Name MyVNet -ResourceGroupName MyRG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name MySubnet -VirtualNetwork $vnet

$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name MyFrontend `
  -PrivateIpAddress 10.0.1.10 `
  -Subnet $subnet

New-AzLoadBalancer `
  -ResourceGroupName MyRG `
  -Name MyInternalLB `
  -Location eastus `
  -Sku Standard `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool

# Create health probe
$probe = New-AzLoadBalancerProbeConfig `
  -Name MyHealthProbe `
  -Protocol Http `
  -Port 80 `
  -RequestPath "/health" `
  -IntervalInSeconds 15 `
  -ProbeCount 2

# Create load balancing rule
$rule = New-AzLoadBalancerRuleConfig `
  -Name MyLBRule `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Probe $probe `
  -IdleTimeoutInMinutes 15 `
  -EnableTcpReset

# Add configurations to load balancer
$lb = Get-AzLoadBalancer -Name MyPublicLB -ResourceGroupName MyRG
$lb.Probes.Add($probe)
$lb.LoadBalancingRules.Add($rule)
Set-AzLoadBalancer -LoadBalancer $lb

# Create inbound NAT rule
$natRule = New-AzLoadBalancerInboundNatRuleConfig `
  -Name MyNATRule `
  -Protocol Tcp `
  -FrontendPort 3389 `
  -BackendPort 3389 `
  -FrontendIpConfiguration $frontendIP

# Create outbound rule
$outboundRule = New-AzLoadBalancerOutboundRuleConfig `
  -Name MyOutboundRule `
  -Protocol All `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -AllocatedOutboundPort 10000 `
  -IdleTimeoutInMinutes 15

# Add VM to backend pool
$nic = Get-AzNetworkInterface -Name MyVM-NIC -ResourceGroupName MyRG
$lb = Get-AzLoadBalancer -Name MyPublicLB -ResourceGroupName MyRG
$backendPool = Get-AzLoadBalancerBackendAddressPoolConfig -Name MyBackendPool -LoadBalancer $lb
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($backendPool)
Set-AzNetworkInterface -NetworkInterface $nic

# Get health probe status
$lb = Get-AzLoadBalancer -Name MyPublicLB -ResourceGroupName MyRG
$lb.Probes | Select-Object Name, ProvisioningState, Protocol, Port

# Monitor metrics
Get-AzMetric -ResourceId $lb.Id -MetricName "HealthProbeStatus" -TimeGrain 00:01:00
```

### Additional Information

#### DNS Zone Types Comparison

|Feature|Public DNS|Private DNS|
|---|---|---|
|Scope|Internet|VNet only|
|Registration|Manual|Auto-registration|
|DNSSEC|Supported|Not supported|
|Zone Transfer|Supported|Not supported|
|Access|Global|VNet + peered VNets|

#### Load Balancer SKU Comparison

|Feature|Basic|Standard|
|---|---|---|
|SLA|None|99.99%|
|Availability Zones|No|Yes|
|Backend Type|VMs in single AS/VMSS|Any VM/VMSS in VNet|
|Health Probes|TCP/HTTP|TCP/HTTP/HTTPS|
|Secure by Default|No|Yes|
|Outbound Rules|No|Yes|
|Price|Free|Charged|

#### Common DNS Record Configurations

```bash
# Wildcard record
az network dns record-set a create --name "*" --zone-name contoso.com --resource-group MyRG

# Root domain record
az network dns record-set a create --name "@" --zone-name contoso.com --resource-group MyRG

# MX record for email
az network dns record-set mx create --name "@" --zone-name contoso.com --resource-group MyRG
az network dns record-set mx add-record --exchange mail.contoso.com --preference 10
```

#### Load Balancer Traffic Distribution Methods

1. **Hash-based**: Default 5-tuple hash (source IP, source port, destination IP, destination port, protocol)
2. **Source IP affinity**: 2-tuple (source IP, destination IP) or 3-tuple (+ protocol)
3. **Floating IP**: For SQL AlwaysOn scenarios
4. **HA Ports**: All ports and protocols (Standard SKU)

#### Health Probe Best Practices

- Use HTTP probes with custom health endpoint
- Implement proper health check logic in applications
- Set reasonable intervals (15-30 seconds)
- Use TCP probes for non-HTTP services
- Monitor probe status in Azure Monitor

#### SNAT Port Allocation

|Backend Instances|Default Ports per Instance|
|---|---|
|1-50|1,024|
|51-100|512|
|101-200|256|
|201-500|128|
|501-1000|64|

#### Troubleshooting Checklist

1. Verify DNS resolution (nslookup/dig)
2. Check health probe status
3. Review NSG rules on backend VMs
4. Validate backend pool membership
5. Test connectivity from different sources
6. Monitor SNAT port exhaustion
7. Check Load Balancer metrics
8. Review application logs