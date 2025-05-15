# Create and Configure Virtual Machines

### Objectives

- **Create a virtual machine**
    - **Basic Requirements**:
        - Resource group and location
        - VM name (1-64 characters, region-unique)
        - Image (OS): Windows Server, Ubuntu, RHEL, etc.
        - Size: Defines vCPUs, memory, storage capacity
        - Administrator account: Username/password or SSH key
    - **Networking Components**:
        - Virtual network and subnet (created automatically if not specified)
        - Public IP address (optional)
        - Network security group (NSG)
        - Network interface card (NIC)
    - **Storage Options**:
        - OS disk: Required, premium or standard SSD/HDD
        - Data disks: Optional, attach during or after creation
        - Temporary disk: Local SSD, data lost on maintenance
    - **Additional Settings**:
        - Boot diagnostics
        - Guest OS diagnostics
        - Extensions (e.g., antimalware, monitoring)

- **Configure Azure Disk Encryption**
    - **Prerequisites**:
        - Azure Key Vault in same region
        - Key Vault access policies configured
        - VM must be running and healthy
    - **Supported Scenarios**:
        - Windows VMs: BitLocker encryption
        - Linux VMs: DM-Crypt encryption
        - Both OS and data disks can be encrypted
    - **Encryption Types**:
        - Platform-managed keys (default)
        - Customer-managed keys (CMK)
        - Double encryption at rest
    - **Limitations**:
        - Basic tier VMs not supported
        - Some VM sizes not supported (e.g., A-series)
        - Cannot disable after enabling

- **Move a virtual machine to another resource group, subscription, or region**
    - **Move to Different Resource Group/Subscription**:
        - Same region only
        - Moves VM and associated resources
        - Downtime not required
        - Validate move before executing
        - Update RBAC permissions after move
    - **Move to Different Region**:
        - Requires Azure Resource Mover or manual recreation
        - Plan for downtime
        - Consider data egress costs
        - Update dependent resources (Load Balancers, etc.)
    - **Resources That Move Together**:
        - Network interfaces
        - Public IP addresses
        - NSGs associated with VM
        - Managed disks

- **Manage virtual machine sizes**
    - **Size Families**:
        - General purpose: B, D, DS, Dv3, Dsv3
        - Compute optimized: F, FS, Fsv2
        - Memory optimized: E, ES, M, GS
        - Storage optimized: Ls, Lsv2
        - GPU: NC, NV, ND
        - High performance: H, HB, HC
    - **Resizing Considerations**:
        - Some sizes require VM deallocation
        - Size must be available in current region
        - Cannot change between certain families
        - May impact temporary disk data
    - **Resize Options**:
        - Hot resize: No downtime for compatible sizes
        - Cold resize: Requires VM stop/deallocate

- **Manage virtual machine disks**
    - **Disk Types**:
        - OS Disk: Required, contains operating system
        - Data Disk: Optional, for application data
        - Temporary Disk: Ephemeral local storage
    - **Managed Disk Features**:
        - Standard HDD: Cost-effective, sequential workloads
        - Standard SSD: Better reliability than HDD
        - Premium SSD: Production workloads
        - Ultra Disk: Highest performance
    - **Disk Operations**:
        - Attach/detach data disks
        - Expand disk size (cannot shrink)
        - Change performance tier
        - Create snapshots
        - Enable disk encryption
    - **Disk Caching Options**:
        - None: No caching
        - ReadOnly: Good for read-heavy workloads
        - ReadWrite: Default for OS disks

- **Deploy virtual machines to availability zones and availability sets**
    - **Availability Zones**:
        - Physically separate locations within region
        - Independent power, cooling, networking
        - Minimum 3 zones in enabled regions
        - 99.99% VM uptime SLA
        - Cross-zone data transfer charges apply
    - **Availability Sets**:
        - Logical grouping within datacenter
        - Fault domains: Separate physical hardware
        - Update domains: Groups for maintenance
        - 99.95% VM uptime SLA
        - No additional cost
    - **Key Differences**:
        - Zones protect against datacenter failures
        - Sets protect against hardware failures
        - Cannot combine zones and sets
        - Zones have higher SLA

- **Deploy and configure an Azure Virtual Machine Scale Sets**
    - **Core Features**:
        - Identical VMs from same image
        - Automatic scaling based on metrics
        - Load balancer or Application Gateway integration
        - Managed or unmanaged disks
        - Rolling upgrades support
    - **Scaling Options**:
        - Manual: Set instance count directly
        - Autoscale: CPU, memory, custom metrics
        - Time-based: Schedule scaling
    - **Update Policies**:
        - Automatic: Immediate updates
        - Rolling: Gradual updates with control
        - Manual: User-triggered updates
    - **Health Monitoring**:
        - Load balancer health probes
        - Application health extension
        - Automatic instance repair

### Common Commands

```bash
# Azure CLI Commands

# Create a basic VM
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --admin-username azureuser \
  --generate-ssh-keys

# Create Windows VM with password
az vm create \
  --resource-group MyRG \
  --name MyWindowsVM \
  --image Win2019Datacenter \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --admin-password 'ComplexPassword123!'

# Enable Azure Disk Encryption
# First, create Key Vault if needed
az keyvault create \
  --name MyKeyVault \
  --resource-group MyRG \
  --location eastus \
  --enabled-for-disk-encryption

# Enable encryption on VM
az vm encryption enable \
  --resource-group MyRG \
  --name MyVM \
  --disk-encryption-keyvault MyKeyVault

# Check encryption status
az vm encryption show \
  --resource-group MyRG \
  --name MyVM

# Move VM to different resource group
az resource move \
  --destination-group NewRG \
  --ids $(az vm show --resource-group MyRG --name MyVM --query id --output tsv)

# Resize VM
az vm resize \
  --resource-group MyRG \
  --name MyVM \
  --size Standard_DS3_v2

# List available VM sizes
az vm list-sizes --location eastus

# Attach data disk
az vm disk attach \
  --resource-group MyRG \
  --vm-name MyVM \
  --name MyDataDisk \
  --size-gb 128 \
  --sku Premium_LRS \
  --new

# Create VM in availability zone
az vm create \
  --resource-group MyRG \
  --name MyZoneVM \
  --image UbuntuLTS \
  --zone 1

# Create availability set
az vm availability-set create \
  --resource-group MyRG \
  --name MyAvailSet \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5

# Create VM in availability set
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image UbuntuLTS \
  --availability-set MyAvailSet

# Create VM Scale Set
az vmss create \
  --resource-group MyRG \
  --name MyScaleSet \
  --image UbuntuLTS \
  --instance-count 2 \
  --vm-sku Standard_DS2_v2 \
  --admin-username azureuser \
  --generate-ssh-keys

# Configure autoscale
az monitor autoscale create \
  --resource-group MyRG \
  --resource MyScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add autoscale rule
az monitor autoscale rule create \
  --resource-group MyRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 3

# Update VMSS instances
az vmss update-instances \
  --resource-group MyRG \
  --name MyScaleSet \
  --instance-ids "*"
```

```powershell
# PowerShell Commands

# Create a basic VM
New-AzVM `
  -ResourceGroupName "MyRG" `
  -Name "MyVM" `
  -Location "eastus" `
  -Image "UbuntuLTS" `
  -Size "Standard_DS2_v2" `
  -Credential (Get-Credential)

# Create Windows VM
$cred = Get-Credential -Message "Enter admin credentials"
New-AzVM `
  -ResourceGroupName "MyRG" `
  -Name "MyWindowsVM" `
  -Location "eastus" `
  -Image "Win2019Datacenter" `
  -Size "Standard_D2s_v3" `
  -Credential $cred

# Enable Azure Disk Encryption
# Create Key Vault
New-AzKeyVault `
  -Name "MyKeyVault" `
  -ResourceGroupName "MyRG" `
  -Location "eastus" `
  -EnabledForDiskEncryption

# Enable encryption
Set-AzVMDiskEncryptionExtension `
  -ResourceGroupName "MyRG" `
  -VMName "MyVM" `
  -DiskEncryptionKeyVaultUrl (Get-AzKeyVault -Name "MyKeyVault").VaultUri `
  -DiskEncryptionKeyVaultId (Get-AzKeyVault -Name "MyKeyVault").ResourceId

# Get encryption status
Get-AzVMDiskEncryptionStatus `
  -ResourceGroupName "MyRG" `
  -VMName "MyVM"

# Move VM to different resource group
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
Move-AzResource `
  -DestinationResourceGroupName "NewRG" `
  -ResourceId $vm.Id

# Resize VM
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM -ResourceGroupName "MyRG" -VM $vm

# List available VM sizes
Get-AzVMSize -Location "eastus"

# Attach data disk
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
Add-AzVMDataDisk `
  -VM $vm `
  -Name "MyDataDisk" `
  -CreateOption Empty `
  -DiskSizeInGB 128 `
  -StorageAccountType Premium_LRS `
  -Lun 0

Update-AzVM -ResourceGroupName "MyRG" -VM $vm

# Create VM in availability zone
New-AzVM `
  -ResourceGroupName "MyRG" `
  -Name "MyZoneVM" `
  -Location "eastus" `
  -Image "UbuntuLTS" `
  -Zone 1

# Create availability set
New-AzAvailabilitySet `
  -ResourceGroupName "MyRG" `
  -Name "MyAvailSet" `
  -Location "eastus" `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 5 `
  -Sku Aligned

# Create VM in availability set
$availSet = Get-AzAvailabilitySet -ResourceGroupName "MyRG" -Name "MyAvailSet"
New-AzVM `
  -ResourceGroupName "MyRG" `
  -Name "MyVM" `
  -Location "eastus" `
  -Image "UbuntuLTS" `
  -AvailabilitySetId $availSet.Id

# Create VM Scale Set
New-AzVmss `
  -ResourceGroupName "MyRG" `
  -VMScaleSetName "MyScaleSet" `
  -Location "eastus" `
  -ImageName "UbuntuLTS" `
  -InstanceCount 2 `
  -VmSize "Standard_DS2_v2" `
  -Credential (Get-Credential)

# Configure autoscale
$rule = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId "/subscriptions/.../MyScaleSet" `
  -TimeGrain 00:01:00 `
  -MetricStatistic Average `
  -TimeWindow 00:05:00 `
  -Operator GreaterThan `
  -Threshold 70 `
  -ScaleActionDirection Increase `
  -ScaleActionScaleType ChangeCount `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

$profile = New-AzAutoscaleProfile `
  -DefaultCapacity 2 `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $rule `
  -Name "autoscale"

Add-AzAutoscaleSetting `
  -ResourceGroupName "MyRG" `
  -Name "autoscale" `
  -TargetResourceId "/subscriptions/.../MyScaleSet" `
  -AutoscaleProfile $profile
```

### Additional Information

#### VM Size Categories and Use Cases

- **B-series**: Burstable, good for variable workloads
- **D-series**: General purpose, balanced CPU/memory
- **E-series**: Memory optimized, in-memory applications
- **F-series**: Compute optimized, high CPU-to-memory ratio
- **M-series**: Memory intensive, large databases
- **N-series**: GPU-enabled, AI/ML workloads

#### Disk Performance Tiers

|Disk Type|IOPS|Throughput|Latency|
|---|---|---|---|
|Standard HDD|Up to 500|Up to 60 MB/s|~10ms|
|Standard SSD|Up to 6,000|Up to 750 MB/s|<10ms|
|Premium SSD|Up to 20,000|Up to 900 MB/s|<1ms|
|Ultra Disk|Up to 160,000|Up to 2,000 MB/s|<1ms|

#### Availability Options Comparison

|Feature|Availability Set|Availability Zone|
|---|---|---|
|Protection Level|Rack/hardware|Datacenter|
|SLA|99.95%|99.99%|
|Cost|No additional|Cross-zone transfer|
|Region Support|All regions|Select regions|

#### Common Exam Scenarios

- Choosing between availability sets and zones
- Understanding disk encryption requirements
- Selecting appropriate VM sizes for workloads
- VMSS autoscaling configuration
- Moving VMs between subscriptions