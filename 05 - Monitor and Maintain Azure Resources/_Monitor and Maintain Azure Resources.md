# Monitor and maintain Azure resources (10–15%)

### Objective Overview

Refer to individual objective pages for details:

- [[01 - Monitor resources in Azure]]
- [[02 - Implement backup and recovery]]

---

#### Key Concepts

- Azure Monitor as the central platform for metrics, logs, and alerts
- Difference between metrics (numerical time-series data) and logs (detailed event data)
- Recovery Services vault vs Azure Backup vault usage scenarios
- Site Recovery for disaster recovery vs Backup for data protection
- KQL (Kusto Query Language) for log analysis

#### Relevant Terms & Definitions

|Term|Definition|Portal Location|
|---|---|---|
|Azure Monitor|Comprehensive monitoring service for collecting, analyzing, and acting on telemetry|Home > Monitor|
|Log Analytics Workspace|Repository for log data collected by Azure Monitor|Monitor > Log Analytics workspaces|
|Application Insights|Application performance monitoring service|Monitor > Application Insights|
|Recovery Services Vault|Container for backup data and Site Recovery configuration|Home > Recovery Services vaults|
|Azure Backup Vault|Modern backup solution for specific Azure services|Home > Backup vaults|
|Action Group|Collection of notification and action preferences for alerts|Monitor > Alerts > Action groups|
|Network Watcher|Network monitoring and diagnostic service|Home > Network Watcher|
|Connection Monitor|Tool for monitoring network connectivity between resources|Network Watcher > Connection monitor|

#### Service Comparison

|Feature|Recovery Services Vault|Azure Backup Vault|Azure Site Recovery|
|---|---|---|---|
|Purpose|Traditional backup container|Modern backup solution|Disaster recovery|
|Supported Resources|VMs, SQL, Files|Blobs, Disks, PostgreSQL|VMs, Physical servers|
|Storage Type|LRS/GRS/RA-GRS|LRS/ZRS/GRS|Based on source|
|Pricing Model|Per instance + storage|Per protected instance|Per protected instance|
|Cross-region|Yes|Limited|Yes (by design)|

|Monitoring Feature|Metrics|Logs|Alerts|
|---|---|---|---|
|Data Type|Numerical time-series|Text/structured events|Threshold/condition-based|
|Retention|93 days|31 days (customizable)|Alert history: 30 days|
|Query Language|Simple aggregations|KQL|Metric/Log queries|
|Real-time|Yes|Near real-time|Yes|
|Use Case|Performance tracking|Troubleshooting|Proactive notification|

---

#### Azure CLI Commands

```bash
# Azure Monitor - Create Log Analytics Workspace
az monitor log-analytics workspace create --resource-group MyRG --workspace-name MyWorkspace

# Create metric alert
az monitor metrics alert create --name MyAlert --resource-group MyRG --scopes /subscriptions/{SubID}/resourceGroups/{RG}/providers/Microsoft.Compute/virtualMachines/{VM} --condition "avg Percentage CPU > 80" --window-size 5m --evaluation-frequency 1m

# Query logs
az monitor log-analytics query --workspace MyWorkspace --analytics-query "AzureActivity | summarize count() by Category" --time-span P1D

# Create Recovery Services vault
az backup vault create --resource-group MyRG --name MyVault --location eastus

# Enable VM backup
az backup protection enable-for-vm --resource-group MyRG --vault-name MyVault --vm MyVM --policy-name DefaultPolicy

# Create Site Recovery vault
az site-recovery vault create --name MySRVault --resource-group MyRG --location eastus

# Network Watcher - Test connectivity
az network watcher test-connectivity --resource-group MyRG --source-resource MyVM1 --dest-resource MyVM2 --protocol TCP --dest-port 80
```

#### PowerShell Commands

```powershell
# Create Log Analytics Workspace
New-AzOperationalInsightsWorkspace -ResourceGroupName MyRG -Name MyWorkspace -Location eastus

# Create action group
New-AzActionGroup -Name MyActionGroup -ResourceGroupName MyRG -ShortName MyAG -Email @{Name="Admin";EmailAddress="admin@contoso.com"}

# Create metric alert
$condition = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -MetricNamespace "Microsoft.Compute/virtualMachines" -TimeAggregation Average -Operator GreaterThan -Threshold 80
Add-AzMetricAlertRuleV2 -Name MyAlert -ResourceGroupName MyRG -WindowSize 0:5 -Frequency 0:1 -TargetResourceId $vm.Id -Condition $condition -ActionGroupId $actionGroup.Id

# Create Recovery Services vault
New-AzRecoveryServicesVault -Name MyVault -ResourceGroupName MyRG -Location eastus

# Enable backup for VM
$vault = Get-AzRecoveryServicesVault -Name MyVault
Set-AzRecoveryServicesVaultContext -Vault $vault
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName MyRG -Name MyVM -Policy $policy

# Network Watcher operations
$networkWatcher = Get-AzNetworkWatcher -ResourceGroupName NetworkWatcherRG
Test-AzNetworkWatcherConnectivity -NetworkWatcher $networkWatcher -SourceId $vm1.Id -DestinationId $vm2.Id -DestinationPort 80
```

---

#### Cost Considerations

- Log Analytics Workspace: Pay per GB ingested and retained
- Application Insights: Free tier available, then pay per GB
- Backup storage: LRS cheaper than GRS, pay per protected instance
- Site Recovery: Per protected instance per month
- Alert rules: Charges per rule evaluation and notifications sent

#### Security Best Practices

- Use managed identities for authentication when possible
- Implement RBAC for monitoring and backup operations
- Encrypt backup data at rest and in transit
- Use private endpoints for Recovery Services vaults
- Regularly review and audit alert configurations

#### Business Use Cases

- **VM Performance Monitoring**: CPU, memory, disk metrics with alerts
- **Application Monitoring**: Application Insights for web apps
- **Security Monitoring**: Activity logs for unauthorized access attempts
- **Disaster Recovery**: Site Recovery for critical workloads
- **Compliance**: Long-term backup retention for regulatory requirements

#### Real-World Analogies

- **Azure Monitor**: Security camera system recording everything
- **Metrics**: Speedometer showing current speed
- **Logs**: Detailed journey logbook
- **Alerts**: Smoke alarm triggering when threshold exceeded
- **Backup**: Safety deposit box for valuables

#### Common Pitfalls & Misconceptions

- Confusing metrics (numerical) with logs (text/events)
- Not understanding retention periods and costs
- Assuming backup automatically includes all data
- Forgetting to test restore procedures
- Not configuring alert suppression/action groups properly

#### Things to Look Out for on the Exam

- Know the difference between Recovery Services vault and Backup vault
- Understand KQL basics for log queries
- Remember metric retention is 93 days, logs are customizable
- Alert rule components: condition, action group, severity
- Site Recovery supports cross-region failover, Backup is for data recovery
- Network Watcher requires specific NSG flow log configuration
- Know which resources support which backup solutions
- Understand the difference between Connection Monitor v1 and v2

---

#### Practice Test Questions I Got Wrong

##### Question 1

**Question:** You need to create an alert when CPU usage exceeds 80% for 5 minutes. What should you configure? **Options:**

- A. Metric alert with 1-minute frequency and 5-minute window
- B. Log alert with 5-minute frequency
- C. Metric alert with 5-minute frequency and 5-minute window
- D. Activity log alert **Correct Answer:** A **Why I Got It Wrong:** Confused frequency with evaluation window **What to Remember:** Frequency is how often the rule runs, window is the time period evaluated

---

##### Question 2

**Question:** Which vault type should you use for Azure Database for PostgreSQL backup? **Options:**

- A. Recovery Services vault
- B. Azure Backup vault
- C. Key vault
- D. Storage account **Correct Answer:** B **Why I Got It Wrong:** Assumed Recovery Services vault handles all backups **What to Remember:** Azure Backup vault is for newer PaaS services like PostgreSQL, Blobs, and Disks

---

#### Practice Scenarios

##### Scenario 1:

Configure comprehensive monitoring for a multi-tier application with:

- VM performance metrics
- Application Insights for web tier
- SQL database monitoring
- Network connectivity monitoring
- Alert notifications to operations team

---

#### Project Ideas

- Implement a complete monitoring solution for a production workload
- Set up automated backup and disaster recovery for critical applications
- Create a monitoring dashboard with custom KQL queries
- Design alert rules with proper suppression and escalation
- Test full disaster recovery failover scenario