# Configure Azure Files and Azure Blob Storage

### Objectives

- **Create and configure a file share in Azure Storage**
    - **SMB 3.0 Protocol**: Encryption in transit, persistent handles, improved performance
    - **Standard File Shares**: Up to 100TB, HDD-backed, pay per GB used
    - **Premium File Shares**: SSD-backed, provisioned IOPS/throughput, FileStorage account only
    - **Quota Management**: Set share size from 1GB to 100TB (standard) or 100GB to 100TB (premium)
    - **Performance Tiers** (Premium only): Provisioned model based on share size
    - **Large File Share Support**: Enable for standard accounts to reach 100TB
    - **Access Tiers**: Transaction Optimized, Hot, Cool (standard shares only)
    - **Protocols**: SMB 2.1/3.0/3.1.1 for Windows/Linux, NFS 4.1 for Linux (premium only)

- **Create and configure a container in Blob Storage**
    - **Access Levels**:
        - Private: No anonymous access, authentication required
        - Blob: Anonymous read access to blobs only
        - Container: Anonymous read access to container and blobs
    - **Metadata**: Key-value pairs (up to 8KB total)
    - **System Properties**: Content-Type, Cache-Control, Content-Encoding
    - **Immutability Policies**: Legal hold or time-based retention
    - **Lease Management**: Exclusive write/delete access for concurrency control
    - **Container Naming**: 3-63 characters, lowercase letters, numbers, and hyphens
    - **Blob Types**: Block blobs, Append blobs, Page blobs

- **Configure storage tiers**
    - **Hot Tier**: Optimized for frequent access, highest storage cost, lowest access cost
    - **Cool Tier**: Infrequent access (30-day minimum), lower storage cost, higher access cost
    - **Archive Tier**: Rare access (180-day minimum), lowest storage cost, highest access cost
    - **Early Deletion Penalties**: Charges apply if moved/deleted before minimum retention
    - **Tier Changes**: Hot↔Cool immediate, Archive requires rehydration (Standard: 15hrs, High Priority: <1hr)
    - **Account-level vs Blob-level**: Set default at account, override per blob
    - **Supported Blob Types**: Block blobs and append blobs only

- **Configure soft delete for blobs and containers**
    - **Retention Period**: 1-365 days configurable
    - **Recovery Options**: Undelete through portal, PowerShell, CLI, or REST API
    - **Soft Delete for Containers**: Recover entire containers (preview in some regions)
    - **Soft Delete for Blobs**: Recover individual blobs and snapshots
    - **Integration**: Works with versioning, snapshots, and change feed
    - **Storage Cost**: Soft-deleted data counts toward storage capacity
    - **Permanent Deletion**: After retention period or manual permanent delete

- **Configure snapshots and soft delete for Azure Files**
    - **Share Snapshots**: Read-only point-in-time copies of entire share
    - **Snapshot Limits**: 200 snapshots per share maximum
    - **Differential Storage**: Only changed data consumes additional space
    - **Azure Backup Integration**: Automated snapshots with retention policies
    - **Soft Delete for Shares**: 1-365 days retention, recover deleted shares
    - **Snapshot Access**: Mount snapshots as read-only shares
    - **VSS Integration**: Application-consistent snapshots for Windows

- **Configure blob lifecycle management**
    - **Policy Rules**: If-then statements with filters and actions
    - **Supported Actions**: Tier transitions, delete blobs, delete snapshots/versions
    - **Filter Types**: Prefix match, blob type, blob age, last accessed time
    - **Rule Evaluation**: Once daily, changes may take 24-48 hours
    - **Action Types**:
        - tierToCool: Move to cool tier after X days
        - tierToArchive: Move to archive tier after X days
        - delete: Delete blob after X days
    - **Version/Snapshot Support**: Separate rules for current version, previous versions, snapshots

- **Configure blob versioning**
    - **Automatic Versioning**: New version on every write operation
    - **Version ID**: Unique identifier (timestamp-based)
    - **Current Version**: Latest version, mutable
    - **Previous Versions**: Historical versions, immutable
    - **Integration**: Works with soft delete and lifecycle management
    - **Storage Costs**: All versions count toward capacity and billing
    - **Access Methods**: Direct access via version ID or list versions
    - **Limitations**: Not supported with NFS 3.0 protocol, ADLS Gen2

### Common Commands

```bash
# Azure CLI Commands

# Create File Share
az storage share create \
  --name myshare \
  --account-name mystorageaccount \
  --quota 100

# Create Premium File Share
az storage share create \
  --name mypremiumshare \
  --account-name mypremiumfiles \
  --quota 500

# Update File Share Properties
az storage share update \
  --name myshare \
  --account-name mystorageaccount \
  --quota 200

# Create Blob Container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount \
  --public-access blob

# Set Container Metadata
az storage container metadata update \
  --name mycontainer \
  --account-name mystorageaccount \
  --metadata key1=value1 key2=value2

# Set Blob Tier
az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --tier Cool

# Configure Account Default Tier
az storage account update \
  --name mystorageaccount \
  --resource-group MyRG \
  --access-tier Cool

# Enable Blob Soft Delete
az storage blob service-properties delete-policy update \
  --account-name mystorageaccount \
  --enable true \
  --days-retained 30

# Enable Container Soft Delete
az storage blob service-properties update \
  --account-name mystorageaccount \
  --enable-container-delete-retention true \
  --container-delete-retention-days 30

# Create File Share Snapshot
az storage share snapshot \
  --name myshare \
  --account-name mystorageaccount

# Enable File Share Soft Delete
az storage account file-service-properties update \
  --account-name mystorageaccount \
  --enable-delete-retention true \
  --delete-retention-days 14

# Create Lifecycle Management Policy
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --policy @policy.json

# Sample policy.json content:
cat > policy.json << 'EOF'
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        }
      }
    }
  ]
}
EOF

# Enable Blob Versioning
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group MyRG \
  --enable-versioning true

# List Blob Versions
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include v \
  --prefix myblob.txt

# Restore Soft-Deleted Blob
az storage blob undelete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt

# Restore Blob from Version
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-blob myblob.txt \
  --destination-container mycontainer \
  --source-uri "https://mystorageaccount.blob.core.windows.net/mycontainer/myblob.txt?versionid=2023-01-01T00:00:00.0000000Z"
```

```powershell
# PowerShell Commands

# Create File Share
New-AzStorageShare `
  -Name "myshare" `
  -Context $ctx `
  -QuotaGiB 100

# Create Premium File Share (needs premium context)
$premiumCtx = (Get-AzStorageAccount -ResourceGroupName "MyRG" -Name "mypremiumfiles").Context
New-AzStorageShare `
  -Name "mypremiumshare" `
  -Context $premiumCtx `
  -QuotaGiB 500

# Update File Share Quota
Set-AzStorageShareQuota `
  -Share "myshare" `
  -Context $ctx `
  -Quota 200

# Create Blob Container
New-AzStorageContainer `
  -Name "mycontainer" `
  -Context $ctx `
  -Permission Blob

# Set Container Metadata
$container = Get-AzStorageContainer -Name "mycontainer" -Context $ctx
$container.CloudBlobContainer.Metadata["key1"] = "value1"
$container.CloudBlobContainer.Metadata["key2"] = "value2"
$container.CloudBlobContainer.SetMetadata()

# Set Blob Access Tier
Set-AzStorageBlobTier `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Tier Cool `
  -Context $ctx

# Configure Account Default Access Tier
Set-AzStorageAccount `
  -ResourceGroupName "MyRG" `
  -Name "mystorageaccount" `
  -AccessTier Cool

# Enable Blob Soft Delete
Enable-AzStorageBlobDeleteRetentionPolicy `
  -ResourceGroupName "MyRG" `
  -AccountName "mystorageaccount" `
  -RetentionDays 30

# Enable Container Soft Delete
Enable-AzStorageContainerDeleteRetentionPolicy `
  -ResourceGroupName "MyRG" `
  -AccountName "mystorageaccount" `
  -RetentionDays 30

# Create File Share Snapshot
$share = Get-AzStorageShare -Name "myshare" -Context $ctx
$snapshot = $share.CloudFileShare.Snapshot()

# Enable File Share Soft Delete
Update-AzStorageFileServiceProperty `
  -ResourceGroupName "MyRG" `
  -AccountName "mystorageaccount" `
  -EnableShareDeleteRetentionPolicy $true `
  -ShareRetentionDays 14

# Create Lifecycle Management Policy
$rule1 = New-AzStorageAccountManagementPolicyRule `
  -Name "rule1" `
  -Action (
    New-AzStorageAccountManagementPolicyAction -BaseBlobTierToCool 30 -BaseBlobTierToArchive 90 -BaseBlobDelete 365
  ) `
  -Filter (
    New-AzStorageAccountManagementPolicyFilter -PrefixMatch "logs/" -BlobType "blockBlob"
  )

Set-AzStorageAccountManagementPolicy `
  -ResourceGroupName "MyRG" `
  -AccountName "mystorageaccount" `
  -Rule $rule1

# Enable Blob Versioning
Update-AzStorageBlobServiceProperty `
  -ResourceGroupName "MyRG" `
  -AccountName "mystorageaccount" `
  -IsVersioningEnabled $true

# List Blob Versions
Get-AzStorageBlob `
  -Container "mycontainer" `
  -Context $ctx `
  -Prefix "myblob.txt" `
  -IncludeVersion

# Restore Soft-Deleted Blob
$blob = Get-AzStorageBlob `
  -Container "mycontainer" `
  -Context $ctx `
  -Blob "myblob.txt" `
  -IncludeDeleted

$blob | Restore-AzStorageBlob

# Restore Specific Version
$versions = Get-AzStorageBlob `
  -Container "mycontainer" `
  -Context $ctx `
  -Prefix "myblob.txt" `
  -IncludeVersion

$specificVersion = $versions | Where-Object { $_.VersionId -eq "2023-01-01T00:00:00.0000000Z" }
$specificVersion | Copy-AzStorageBlob -DestBlob "myblob.txt" -DestContainer "mycontainer"
```

### Additional Relevant Information

#### Azure Files vs Blob Storage Decision Matrix

|Scenario|Use Azure Files|Use Blob Storage|
|---|---|---|
|Lift-and-shift applications|✓||
|Shared application settings|✓||
|Dev/test file shares|✓||
|Unstructured data (images, videos)||✓|
|Static website hosting||✓|
|Data archival||✓|
|Big data analytics||✓|

#### Storage Tier Cost Comparison (per GB/month)

|Tier|Storage Cost|Read Cost|Write Cost|Early Deletion|
|---|---|---|---|---|
|Hot|Highest|Lowest|Lowest|None|
|Cool|Medium|Medium|Medium|30 days|
|Archive|Lowest|Highest|Highest|180 days|

#### Blob Versioning vs Snapshots

- **Versioning**: Automatic, captures every change, version-level access
- **Snapshots**: Manual, point-in-time copies, read-only access
- Use versioning for audit trails and automatic protection
- Use snapshots for manual checkpoints and backups

#### Lifecycle Management Best Practices

- Start with conservative rules (longer retention)
- Test policies on non-critical data first
- Consider last access tracking for tier transitions
- Separate rules for current versions vs old versions
- Monitor policy execution in activity logs

#### Soft Delete Recovery Scenarios

- Accidental deletion by users
- Application bugs causing data loss
- Ransomware attacks (limited protection)
- Compliance requirements for data recovery
- Development/testing environments

#### Common Exam Scenarios

- Choosing between file shares and blob storage
- Calculating costs for different tier transitions
- Understanding lifecycle policy execution timing
- Configuring appropriate soft delete retention
- Selecting correct blob type for scenarios