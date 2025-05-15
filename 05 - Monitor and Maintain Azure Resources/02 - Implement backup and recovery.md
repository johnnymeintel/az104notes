# Implement Backup and Recovery

### Objectives

- **Create a Recovery Services vault**
    - **Vault Basics**:
        - Container for backup data and recovery points
        - Region-specific (must be in same region as resources)
        - Storage redundancy options: LRS, GRS, RA-GRS
        - Soft delete enabled by default (14-day retention)
    - **Supported Workloads**:
        - Azure VMs (Windows and Linux)
        - SQL Server in Azure VMs
        - SAP HANA databases
        - Azure File shares
        - On-premises workloads (via MARS agent)
    - **Storage Replication Types**:
        - LRS: 3 copies in single datacenter
        - ZRS: 3 copies across availability zones
        - GRS: 6 copies (3 local + 3 in paired region)
        - RA-GRS: GRS with read access to secondary
    - **Security Features**:
        - Encryption at rest (Microsoft or customer keys)
        - Role-based access control
        - Multi-factor authentication for critical operations
        - Immutable vaults for compliance

- **Create an Azure Backup vault**
    - **New Backup Solution**:
        - Designed for newer Azure services
        - Simplified management experience
        - Built on Recovery Services vault technology
    - **Supported Services**:
        - Azure Blobs (operational backup)
        - Azure Disks
        - Azure Database for PostgreSQL
        - Future Azure data services
    - **Key Differences from RSV**:
        - Operational vs snapshot-based backups
        - Continuous backup for supported services
        - Retention up to 10 years
        - Zone-redundant storage by default
    - **Management Features**:
        - Backup center integration
        - Azure Policy support
        - Cost estimation tools
        - Cross-subscription management

- **Create and configure a backup policy**
    - **Policy Components**:
        - Backup schedule (daily/weekly)
        - Retention settings (daily/weekly/monthly/yearly)
        - Instant restore snapshots (1-5 days)
        - Backup time and time zone
    - **Default Policies**:
        - DefaultPolicy: Daily backup at 6:00 PM, 30-day retention
        - EnhancedPolicy: Includes instant restore
        - HourlyPolicy: Multiple backups per day
    - **Custom Policy Options**:
        - Multiple backups per day (Enhanced tier)
        - Application-consistent snapshots
        - File-level restore capability
        - Cross-region restore enablement
    - **Retention Rules**:
        - Daily: 7-9999 days
        - Weekly: 1-5163 weeks
        - Monthly: 1-1188 months
        - Yearly: 1-99 years

- **Perform backup and restore operations by using Azure Backup**
    - **Backup Types**:
        - Full backup: Complete copy of data
        - Incremental: Only changed blocks
        - Differential: Changes since last full
        - Application-consistent: With VSS/quiescing
    - **VM Backup Process**:
        - Initial backup: Full copy
        - Subsequent: Incremental only
        - Instant restore tier (1-5 days)
        - Standard tier (up to retention limit)
    - **Restore Options**:
        - Create new VM
        - Replace existing VM
        - Restore disks only
        - File-level recovery
        - Cross-region restore (if GRS)
    - **File Recovery Process**:
        - Mount recovery point as drive
        - Browse and copy files
        - Unmount when complete
        - Available for Windows and Linux VMs

- **Configure Azure Site Recovery for Azure resources**
    - **Disaster Recovery Concepts**:
        - RPO (Recovery Point Objective): Data loss tolerance
        - RTO (Recovery Time Objective): Downtime tolerance
        - Replication frequency: Continuous for Azure VMs
        - Crash-consistent vs app-consistent points
    - **Supported Scenarios**:
        - Azure to Azure (cross-region)
        - On-premises to Azure
        - VMware/Hyper-V to Azure
        - Physical servers to Azure
    - **Replication Settings**:
        - Source and target regions
        - Cache storage account
        - Target resource group
        - Virtual network mapping
        - Replication policy (24-hour retention default)
    - **Network Considerations**:
        - Preserve IP addresses option
        - Network mapping configuration
        - NSG and load balancer replication
        - Public IP assignment options

- **Perform a failover to a secondary region by using Site Recovery**
    - **Failover Types**:
        - Test failover: Non-disruptive validation
        - Planned failover: Zero data loss migration
        - Unplanned failover: Emergency DR activation
    - **Failover Process**:
        - Select recovery point
        - Choose failover type
        - Shut down source (if accessible)
        - Create resources in target region
        - Boot VMs in correct order
    - **Recovery Points**:
        - Latest (lowest RPO)
        - Latest processed
        - Latest app-consistent
        - Custom time-based
    - **Post-Failover Tasks**:
        - Commit or cancel failover
        - Reprotect for failback
        - Update DNS/traffic routing
        - Validate application functionality

- **Configure and interpret reports and alerts for backups**
    - **Backup Reports**:
        - Backup items summary
        - Job success/failure trends
        - Storage consumption
        - Protected vs unprotected resources
        - Cost analysis and optimization
    - **Alert Types**:
        - Backup failures
        - Restore failures
        - Security alerts
        - Configuration issues
        - Deletion attempts
    - **Monitoring Options**:
        - Azure Monitor integration
        - Log Analytics workspace
        - Backup Explorer dashboard
        - Email notifications
        - ITSM connector integration
    - **Compliance Reporting**:
        - Backup compliance dashboard
        - Policy adherence reports
        - Retention compliance
        - Encryption status
        - Audit logs

### Common Commands

```bash
# Azure CLI Commands

# Create Recovery Services vault
az backup vault create \
  --resource-group MyRG \
  --name MyRecoveryVault \
  --location eastus \
  --storage-redundancy LocallyRedundant

# Create Azure Backup vault
az dataprotection backup-vault create \
  --resource-group MyRG \
  --vault-name MyBackupVault \
  --location eastus \
  --storage-setting datastore-type="VaultStore" type="LocallyRedundant"

# Set vault storage redundancy
az backup vault backup-properties set \
  --resource-group MyRG \
  --name MyRecoveryVault \
  --backup-storage-redundancy GeoRedundant

# Create backup policy
az backup policy create \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --policy @policy.json \
  --name MyCustomPolicy \
  --backup-management-type AzureIaasVM

# Sample policy.json
cat > policy.json << 'EOF'
{
  "schedulePolicy": {
    "schedulePolicyType": "SimpleSchedulePolicy",
    "scheduleRunFrequency": "Daily",
    "scheduleRunTimes": ["2021-01-01T02:00:00Z"]
  },
  "retentionPolicy": {
    "retentionPolicyType": "LongTermRetentionPolicy",
    "dailySchedule": {
      "retentionTimes": ["2021-01-01T02:00:00Z"],
      "retentionDuration": {
        "count": 30,
        "durationType": "Days"
      }
    }
  }
}
EOF

# Enable VM backup
az backup protection enable-for-vm \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --vm MyVM \
  --policy-name DefaultPolicy

# Start ad-hoc backup
az backup protection backup-now \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name MyVM \
  --item-name MyVM \
  --retain-until 01-02-2024

# List recovery points
az backup recoverypoint list \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name MyVM \
  --item-name MyVM

# Restore VM
az backup restore restore-disks \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --container-name MyVM \
  --item-name MyVM \
  --rp-name <recovery-point> \
  --storage-account mystorageaccount \
  --restore-mode AlternateLocation \
  --target-resource-group RestoreRG

# Enable Site Recovery for VM
az resource create \
  --resource-group MyRG \
  --resource-type "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems" \
  --name "MyRecoveryVault/Azure-EastUS/cloud_123/VM1" \
  --api-version "2018-07-10" \
  --properties @asr-properties.json

# Initiate test failover
az resource invoke-action \
  --resource-group MyRG \
  --resource-type "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationProtectedItems" \
  --name "MyRecoveryVault/Azure-EastUS/cloud_123/VM1" \
  --api-version "2018-07-10" \
  --action testFailover \
  --request-body @test-failover.json

# Create backup alert rule
az monitor metrics alert create \
  --resource-group MyRG \
  --name BackupFailureAlert \
  --scopes /subscriptions/{subId}/resourceGroups/MyRG/providers/Microsoft.RecoveryServices/vaults/MyRecoveryVault \
  --condition "count BackupFailureCount > 0" \
  --action-group MyActionGroup \
  --description "Alert when backup fails"

# Configure backup reports
az backup report configure \
  --resource-group MyRG \
  --vault-name MyRecoveryVault \
  --log-analytics-workspace MyWorkspace
```

```powershell
# PowerShell Commands

# Create Recovery Services vault
New-AzRecoveryServicesVault `
  -Name MyRecoveryVault `
  -ResourceGroupName MyRG `
  -Location eastus

# Create Azure Backup vault
New-AzDataProtectionBackupVault `
  -ResourceGroupName MyRG `
  -VaultName MyBackupVault `
  -Location eastus `
  -StorageSettingType LocallyRedundant

# Set vault context
$vault = Get-AzRecoveryServicesVault -Name MyRecoveryVault -ResourceGroupName MyRG
Set-AzRecoveryServicesVaultContext -Vault $vault

# Set storage redundancy
Set-AzRecoveryServicesBackupProperty `
  -Vault $vault `
  -BackupStorageRedundancy GeoRedundant

# Create backup policy
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType AzureVM
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType AzureVM

$schPol.ScheduleRunFrequency = "Daily"
$schPol.ScheduleRunTimes[0] = "2021-01-01T02:00:00Z"

$retPol.DailySchedule.DurationCountInDays = 30
$retPol.IsDailyScheduleEnabled = $true

New-AzRecoveryServicesBackupProtectionPolicy `
  -Name MyCustomPolicy `
  -WorkloadType AzureVM `
  -RetentionPolicy $retPol `
  -SchedulePolicy $schPol

# Enable VM backup
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name DefaultPolicy
$vm = Get-AzVM -ResourceGroupName MyRG -Name MyVM

Enable-AzRecoveryServicesBackupProtection `
  -ResourceGroupName MyRG `
  -Policy $policy `
  -Name $vm.Name `
  -ResourceType AzureVM

# Start backup job
$container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Name MyVM
$item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType AzureVM

Backup-AzRecoveryServicesBackupItem `
  -Item $item `
  -ExpiryDateTimeUTC (Get-Date).AddDays(30)

# List recovery points
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item

# Restore VM
Restore-AzRecoveryServicesBackupItem `
  -RecoveryPoint $rp[0] `
  -StorageAccountName mystorageaccount `
  -StorageAccountResourceGroupName MyRG `
  -TargetResourceGroupName RestoreRG `
  -RestoreAsUnmanagedDisks

# Configure Site Recovery
$vault = Get-AzRecoveryServicesVault -Name MyRecoveryVault
Set-AzRecoveryServicesAsrVaultContext -Vault $vault

# Create replication policy
New-AzRecoveryServicesAsrPolicy `
  -Name MyReplicationPolicy `
  -ReplicationProvider HyperVReplicaAzure `
  -ReplicationFrequencyInSeconds 300 `
  -RecoveryPoints 24 `
  -ApplicationConsistentSnapshotFrequencyInHours 4 `
  -RecoveryAzureStorageAccountId "/subscriptions/{subId}/resourceGroups/MyRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount"

# Enable replication for VM
$vm = Get-AzVM -ResourceGroupName MyRG -Name MyVM
$protectionContainer = Get-AzRecoveryServicesAsrProtectionContainer
$policy = Get-AzRecoveryServicesAsrPolicy -Name MyReplicationPolicy

New-AzRecoveryServicesAsrProtectableItem `
  -ProtectionContainer $protectionContainer `
  -FriendlyName $vm.Name `
  -ProtectionContainerId $protectionContainer.ID `
  -AzureVmId $vm.Id

# Start test failover
$protectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer
Start-AzRecoveryServicesAsrTestFailoverJob `
  -ReplicationProtectedItem $protectedItem `
  -Direction PrimaryToRecovery `
  -AzureVMNetworkId "/subscriptions/{subId}/resourceGroups/MyRG/providers/Microsoft.Network/virtualNetworks/TestVNet"

# Configure backup alerts
$actionEmail = New-AzActionGroupReceiver -Name 'email' -EmailReceiver -EmailAddress 'admin@contoso.com'
$actionGroup = Set-AzActionGroup -ResourceGroupName MyRG -Name MyActionGroup -ShortName MyAG -Receiver $actionEmail

# Get backup metrics for alerts
Get-AzMetric `
  -ResourceId $vault.ID `
  -MetricName "BackupFailureCount" `
  -TimeGrain 01:00:00 `
  -StartTime (Get-Date).AddDays(-1) `
  -EndTime (Get-Date)
```

### Additional Information

#### Backup vs Site Recovery Comparison

|Feature|Azure Backup|Azure Site Recovery|
|---|---|---|
|Purpose|Data protection|Disaster recovery|
|RPO|Up to 24 hours|Near real-time|
|RTO|Hours to days|Minutes to hours|
|Granularity|File/folder level|VM/server level|
|Use Case|Accidental deletion|Region failure|

#### Recovery Services Vault Storage Types

|Type|Copies|Distribution|Use Case|
|---|---|---|---|
|LRS|3|Single datacenter|Dev/test, low-cost|
|ZRS|3|Across zones|High availability|
|GRS|6|Cross-region|Disaster recovery|
|RA-GRS|6|Cross-region + read|Compliance/reporting|

#### Backup Policy Best Practices

1. Use default policies for standard workloads
2. Create custom policies for specific requirements
3. Enable instant restore for quick recovery
4. Configure appropriate retention periods
5. Consider compliance requirements
6. Test restore procedures regularly

#### Site Recovery Failover Types

|Type|Data Loss|Downtime|Use Case|
|---|---|---|---|
|Test Failover|None|None|DR drills|
|Planned Failover|None|Minimal|Maintenance|
|Unplanned Failover|Possible|Variable|Disasters|

#### Common Monitoring Queries

```kql
// Backup job failures
AzureBackupJobsV2
| where JobStatus == "Failed"
| project TimeGenerated, BackupItemName, ErrorCode, ErrorMessage
| order by TimeGenerated desc

// Storage consumption trend
AzureBackupStorageV2
| summarize StorageGB = sum(StorageConsumedInMBs)/1024 by bin(TimeGenerated, 1d)
| render timechart

// Recovery point health
AzureBackupRecoveryPointsV2
| summarize LatestRP = max(RecoveryPointTime) by BackupItemName
| where LatestRP < ago(1d)
```