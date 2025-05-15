# Configure and Manage Storage Accounts

### Objectives

- **Create and configure storage accounts**
    - **StorageV2 (General Purpose v2)**: Recommended, supports all storage services and latest features
    - **StorageV1 (General Purpose v1)**: Legacy, lacks some features like access tiers
    - **BlobStorage**: Specialized for block/append blobs only, supports tiering
    - **FileStorage**: Premium files only, requires Premium_LRS
    - **BlockBlobStorage**: Premium block blobs only, requires Premium_LRS/Premium_ZRS
    - **Standard Performance**: HDD-backed, cost-effective for most workloads
    - **Premium Performance**: SSD-backed, low latency, high IOPS
    - Choose replication during creation (can change some types later)
    - Select region, resource group, and naming (3-24 chars, globally unique)

- **Configure Azure Storage redundancy**
    - **LRS (Locally Redundant Storage)**: 3 copies in single datacenter, 11 9's durability
    - **ZRS (Zone Redundant Storage)**: 3 copies across availability zones, 12 9's durability
    - **GRS (Geo-Redundant Storage)**: LRS + async replication to paired region, 16 9's durability
    - **GZRS (Geo-Zone Redundant Storage)**: ZRS + async replication to paired region, 16 9's durability
    - **RA-GRS/RA-GZRS**: Read Access variants allow reading from secondary region
    - Failover capabilities: Manual (all GRS types) or no failover (LRS/ZRS)
    - Consider RPO (Recovery Point Objective) for geo-redundant options

- **Configure object replication**
    - Asynchronous copy of blobs between storage accounts
    - Source and destination can be in different regions
    - Requires versioning enabled on both accounts
    - Create replication policies with multiple rules
    - Filter by prefix, blob type, or creation time
    - Support for blob snapshots and versions
    - Useful for compliance, backup, and data distribution
    - Monitor replication status per blob

- **Configure storage account encryption**
    - **Storage Service Encryption (SSE)**: Always enabled, Microsoft-managed keys by default
    - **Customer-managed keys (CMK)**: Use Azure Key Vault or Managed HSM
    - **Infrastructure encryption**: Additional layer, 256-bit AES at infrastructure level
    - Encryption scopes for granular control within account
    - Key rotation support (automatic with CMK)
    - Table and Queue encryption with CMK requires special configuration
    - All data encrypted at rest, including metadata

- **Manage data by using Azure Storage Explorer and AzCopy**
    - **Storage Explorer**: Cross-platform GUI tool
        - Manage blobs, files, queues, tables
        - Upload/download with drag-and-drop
        - Manage access policies and SAS
        - Search and filter capabilities
        - Local emulator support
    - **AzCopy**: Command-line utility for high-performance transfers
        - Optimized for large-scale operations
        - Supports parallelism and resumable transfers
        - Copy between accounts, to/from local files
        - Sync functionality for incremental copies
        - Benchmark and optimize transfer settings
    - Both support SAS tokens, access keys, and Azure AD authentication

### Common Commands

```bash
# Azure CLI Commands

# Create General Purpose v2 Storage Account
az storage account create \
  --name mystorageaccount \
  --resource-group MyRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Create Premium Block Blob Storage Account
az storage account create \
  --name mypremiumblobs \
  --resource-group MyRG \
  --location eastus \
  --sku Premium_LRS \
  --kind BlockBlobStorage

# Change Redundancy Type
az storage account update \
  --name mystorageaccount \
  --resource-group MyRG \
  --sku Standard_GRS

# Enable Infrastructure Encryption
az storage account create \
  --name mysecurestorage \
  --resource-group MyRG \
  --location eastus \
  --sku Standard_LRS \
  --require-infrastructure-encryption true

# Configure Customer-Managed Keys
az storage account update \
  --name mystorageaccount \
  --resource-group MyRG \
  --encryption-key-source Microsoft.Keyvault \
  --encryption-key-vault https://mykeyvault.vault.azure.net \
  --encryption-key-name MyKey \
  --encryption-key-version 1234567890abcdef

# Enable Versioning (required for object replication)
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --enable-versioning true

# Create Object Replication Policy
az storage account or-policy create \
  --account-name destination-account \
  --resource-group MyRG \
  --source-account source-account \
  --destination-account destination-account \
  --policy-id policy001

# Add Replication Rule
az storage account or-policy rule add \
  --account-name destination-account \
  --resource-group MyRG \
  --policy-id policy001 \
  --rule-id rule001 \
  --source-container source \
  --destination-container destination \
  --prefix-match "logs/"

# AzCopy Examples
# Login with Azure AD
azcopy login

# Copy blob between storage accounts
azcopy copy \
  'https://source.blob.core.windows.net/container/blob?SAS' \
  'https://dest.blob.core.windows.net/container/blob?SAS'

# Sync directory to blob container
azcopy sync \
  '/local/path' \
  'https://account.blob.core.windows.net/container?SAS' \
  --recursive

# Copy with specific performance settings
azcopy copy \
  'https://source.blob.core.windows.net/container/*?SAS' \
  'https://dest.blob.core.windows.net/container/?SAS' \
  --block-size-mb 8 \
  --parallel-count 16
```

```powershell
# PowerShell Commands

# Create General Purpose v2 Storage Account
New-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -Location "eastus" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot"

# Create Premium File Storage Account
New-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mypremiumfiles" `
  -Location "eastus" `
  -SkuName "Premium_LRS" `
  -Kind "FileStorage"

# Change Storage Account Redundancy
Set-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -SkuName "Standard_GRS"

# Enable Infrastructure Encryption (during creation)
New-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mysecurestorage" `
  -Location "eastus" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -RequireInfrastructureEncryption

# Configure Customer-Managed Keys
Set-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -KeyVaultUri "https://mykeyvault.vault.azure.net" `
  -KeyName "MyKey" `
  -KeyVersion "1234567890abcdef" `
  -KeyVaultEncryption

# Enable Blob Versioning
Enable-AzStorageBlobVersioning `
  -ResourceGroupName "MyRG" `
  -StorageAccountName "mystorageaccount"

# Create Object Replication Policy
$rule = New-AzStorageObjectReplicationPolicyRule `
  -SourceContainer "source" `
  -DestinationContainer "destination" `
  -PrefixMatch "logs/"

New-AzStorageObjectReplicationPolicy `
  -ResourceGroupName "MyRG" `
  -StorageAccountName "destination-account" `
  -SourceAccount "source-account" `
  -Rule $rule

# Storage Explorer Operations (via PowerShell)
# Get Storage Context
$ctx = Get-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" | Get-AzStorageAccountKey | Select-Object -First 1 | Get-AzStorageContext

# List Containers
Get-AzStorageContainer -Context $ctx

# Upload Blob
Set-AzStorageBlobContent `
  -Container "mycontainer" `
  -File "C:\local\file.txt" `
  -Blob "uploaded-file.txt" `
  -Context $ctx

# Download Blob
Get-AzStorageBlobContent `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Destination "C:\local\" `
  -Context $ctx

# AzCopy with PowerShell
# Generate SAS for AzCopy
$sas = New-AzStorageAccountSASToken `
  -Service Blob `
  -ResourceType Container,Object `
  -Permission rwdl `
  -ExpiryTime (Get-Date).AddDays(7) `
  -Context $ctx

# Use with AzCopy (run in terminal)
$source = "https://sourceaccount.blob.core.windows.net/container" + $sas
$dest = "https://destaccount.blob.core.windows.net/container" + $sas
azcopy copy $source $dest --recursive
```

### Additional Relevant Information

#### Storage Account Naming Rules

- 3-24 characters long
- Lowercase letters and numbers only
- Must be globally unique across all Azure
- Cannot contain hyphens or special characters

#### Performance Tiers Comparison

|Feature|Standard|Premium|
|---|---|---|
|Storage Type|HDD|SSD|
|Latency|Higher|Sub-millisecond|
|IOPS|Up to 20K|Up to 100K+|
|Throughput|Up to 60 MiB/s per blob|Up to 5 GiB/s per account|
|Use Cases|General purpose, backup|High-performance apps, databases|

#### Redundancy Decision Matrix

- **LRS**: Development/test, data that can be reconstructed
- **ZRS**: High availability within region, compliance requirements
- **GRS**: Disaster recovery, cross-region compliance
- **GZRS**: Maximum availability and durability
- **RA-GRS/RA-GZRS**: When read access from secondary is needed

#### Object Replication Considerations

- Not real-time (asynchronous)
- Costs for egress, transactions, and storage
- Versioning must be enabled
- Doesn't replicate system metadata
- Consider for compliance, disaster recovery, content distribution

#### Encryption Scenarios

- **Default SSE**: Sufficient for most compliance requirements
- **CMK**: Required for regulatory compliance, key rotation control
- **Infrastructure Encryption**: Defense in depth, highly sensitive data
- **Encryption Scopes**: Department-level or project-level encryption

#### Storage Explorer vs AzCopy

- **Use Storage Explorer when**:
    - Ad-hoc operations
    - Visual management needed
    - Managing permissions/policies
    - Exploring storage structure
- **Use AzCopy when**:
    - Large-scale migrations
    - Automation/scripting
    - Performance is critical
    - Bandwidth throttling needed

#### Common Exam Topics

- Choosing appropriate storage account type and performance tier
- Understanding redundancy options and failover capabilities
- When to use object replication vs other backup methods
- Customer-managed keys requirements and limitations
- Selecting right tool for data management scenarios