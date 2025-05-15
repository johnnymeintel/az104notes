# Monitor Resources in Azure

### Objectives

- **Interpret metrics in Azure Monitor**
    - **Platform Metrics**: Automatically collected from Azure resources
        - CPU percentage, network in/out, disk read/write
        - Available in near real-time (1-minute granularity)
        - Retained for 93 days
        - No configuration required
    - **Guest OS Metrics**: Require diagnostic extension
        - Memory usage, process information
        - More detailed performance counters
        - Windows: WAD extension, Linux: LAD extension
    - **Metric Aggregations**:
        - Average, Sum, Minimum, Maximum, Count
        - Time granularity: 1 min, 5 min, 15 min, 30 min, 1 hour, 6 hours, 12 hours, 1 day
    - **Multi-dimensional Metrics**:
        - Filter and split by dimensions
        - Example: Disk metrics by LUN, Network by direction

- **Configure log settings in Azure Monitor**
    - **Log Types**:
        - Activity logs: Subscription-level events
        - Resource logs: Resource-specific operations
        - Azure AD logs: Sign-ins and audit logs
        - Custom logs: Application and OS logs
    - **Diagnostic Settings**:
        - Send logs to: Log Analytics, Storage Account, Event Hub, Partner solutions
        - Configure per resource or at scale using Azure Policy
        - Retention policies per destination
    - **Log Analytics Workspace**:
        - Central repository for log data
        - Data retention: 31-730 days (2 years)
        - Pricing tiers: Per GB or Capacity Reservation
        - Data ingestion limits and caps

- **Query and analyze logs in Azure Monitor**
    - **Kusto Query Language (KQL) Basics**:
        - Table | where | summarize | sort | project
        - Time filters: ago(), between(), datetime()
        - Aggregations: count(), avg(), sum(), max(), min()
        - String operations: contains, startswith, endswith
    - **Query Management**:
        - Save queries for reuse
        - Export results (CSV, JSON)
        - Pin to dashboards
        - Schedule queries for reports

- **Set up alert rules, action groups, and alert processing rules in Azure Monitor**
    - **Alert Types**:
        - Metric alerts: Threshold-based on metrics
        - Log alerts: Based on log query results
        - Activity log alerts: Subscription/resource events
        - Smart detection alerts: AI-based anomaly detection
    - **Alert Components**:
        - Target resource(s)
        - Signal type (metric/log/activity)
        - Condition logic and threshold
        - Severity level (0-4)
        - Action groups for notification
    - **Action Groups**:
        - Email/SMS/Push/Voice notifications
        - Azure Functions, Logic Apps, Webhooks
        - ITSM integration, Runbooks
        - Multiple actions per group
    - **Alert Processing Rules**:
        - Suppress alerts during maintenance
        - Route alerts to different action groups
        - Add action groups to existing alerts
        - Filter by resource type, tags, or time

- **Configure and interpret monitoring of virtual machines, storage accounts, and networks by using Azure Monitor Insights**
    - **VM Insights**:
        - Performance charts and health metrics
        - Process and dependency mapping
        - Guest OS metrics without agent configuration
        - Service Map for application dependencies
        - At-scale views across subscriptions
    - **Storage Insights**:
        - Capacity utilization and trends
        - Transaction metrics and latency
        - Availability and error rates
        - Operation-level monitoring
    - **Network Insights**:
        - Network topology visualization
        - Connection metrics and health
        - NSG flow analytics
        - ExpressRoute monitoring
        - VPN gateway diagnostics
    - **Container Insights**:
        - Kubernetes cluster monitoring
        - Container performance metrics
        - Pod and node health
        - Application logs from containers

- **Use Azure Network Watcher and Connection Monitor**
    - **Network Watcher Features**:
        - IP flow verify: Check if packet is allowed/denied
        - Next hop: Determine packet routing path
        - Connection troubleshoot: Test connectivity
        - Packet capture: Network traffic analysis
        - VPN troubleshoot: Diagnose gateway issues
    - **NSG Flow Logs**:
        - Traffic flow through NSGs
        - Version 1: Basic flow information
        - Version 2: Enhanced with bytes/packets
        - Traffic Analytics for visualization
    - **Connection Monitor**:
        - Continuous connectivity monitoring
        - Multi-endpoint testing
        - Latency and packet loss metrics
        - HTTP/HTTPS status code monitoring
        - Topology visualization
    - **Network Performance Monitor**:
        - End-to-end network monitoring
        - ExpressRoute monitoring
        - Service connectivity monitor
        - Performance metrics and alerts

### Common Commands

```bash
# Azure CLI Commands

# Create Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group MyRG \
  --workspace-name MyWorkspace \
  --location eastus \
  --retention-time 90

# Enable diagnostic settings for a resource
az monitor diagnostic-settings create \
  --name MyDiagnostics \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm} \
  --workspace MyWorkspace \
  --logs '[{"category": "Administrative", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

# Query logs
az monitor log-analytics query \
  --workspace MyWorkspace \
  --analytics-query "AzureActivity | where Level == 'Error' | take 10" \
  --time-span P1D

# Create metric alert
az monitor metrics alert create \
  --name HighCPUAlert \
  --resource-group MyRG \
  --scopes /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm} \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-group MyActionGroup \
  --severity 2

# Create action group
az monitor action-group create \
  --resource-group MyRG \
  --name MyActionGroup \
  --short-name MyAG \
  --email admin adminmail@contoso.com \
  --sms 1 +1234567890 \
  --webhook MyWebhook https://myapp.com/alert

# Create log alert
az monitor log-analytics workspace show --resource-group MyRG --workspace-name MyWorkspace --query customerId -o tsv
az monitor scheduled-query create \
  --resource-group MyRG \
  --name ErrorLogAlert \
  --scopes /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/{workspace} \
  --condition "count > 5" \
  --condition-query "AzureActivity | where Level == 'Error'" \
  --action-groups MyActionGroup

# Enable VM Insights
az vm extension set \
  --resource-group MyRG \
  --vm-name MyVM \
  --name DependencyAgentLinux \
  --publisher Microsoft.Azure.Monitoring.DependencyAgent \
  --version 9.10

# Network Watcher operations
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus westus \
  --enabled

# IP flow verify
az network watcher test-ip-flow \
  --resource-group MyRG \
  --vm MyVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.0.4:80 \
  --remote 10.0.0.5:4000

# Create Connection Monitor
az network watcher connection-monitor create \
  --resource-group MyRG \
  --name MyConnectionMonitor \
  --location eastus \
  --source-resource MyVM1 \
  --dest-resource MyVM2 \
  --dest-port 80

# Start packet capture
az network watcher packet-capture create \
  --resource-group MyRG \
  --vm MyVM \
  --name MyCapture \
  --storage-account MyStorageAccount \
  --time-limit 60

# Enable NSG flow logs
az network watcher flow-log create \
  --resource-group MyRG \
  --name MyFlowLog \
  --nsg MyNSG \
  --storage-account MyStorageAccount \
  --enabled true \
  --retention 30 \
  --format JSON \
  --log-version 2
```

```powershell
# PowerShell Commands

# Create Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
  -ResourceGroupName MyRG `
  -Name MyWorkspace `
  -Location eastus `
  -RetentionInDays 90

# Enable diagnostic settings
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName MyRG -Name MyWorkspace
$vm = Get-AzVM -ResourceGroupName MyRG -Name MyVM

Set-AzDiagnosticSetting `
  -ResourceId $vm.Id `
  -Name "MyDiagnostics" `
  -WorkspaceId $workspace.ResourceId `
  -Enabled $true `
  -Category Administrative, Security

# Query logs
$query = @"
AzureActivity 
| where Level == 'Error' 
| take 10
"@

Invoke-AzOperationalInsightsQuery `
  -WorkspaceId $workspace.CustomerId `
  -Query $query `
  -Timespan (New-TimeSpan -Days 1)

# Create action group
$email1 = New-AzActionGroupReceiver -Name 'admin' -EmailReceiver -EmailAddress 'admin@contoso.com'
$sms1 = New-AzActionGroupReceiver -Name 'sms1' -SmsReceiver -CountryCode '1' -PhoneNumber '2345678901'

Set-AzActionGroup `
  -ResourceGroupName MyRG `
  -Name MyActionGroup `
  -ShortName MyAG `
  -Receiver $email1,$sms1

# Create metric alert
$condition = New-AzMetricAlertRuleV2Criteria `
  -MetricName "Percentage CPU" `
  -MetricNamespace "Microsoft.Compute/virtualMachines" `
  -TimeAggregation Average `
  -Operator GreaterThan `
  -Threshold 80

Add-AzMetricAlertRuleV2 `
  -Name HighCPUAlert `
  -ResourceGroupName MyRG `
  -WindowSize 00:05:00 `
  -Frequency 00:01:00 `
  -TargetResourceId $vm.Id `
  -Condition $condition `
  -ActionGroupId $actionGroup.Id `
  -Severity 2

# Enable VM Insights
Set-AzVMExtension `
  -ResourceGroupName MyRG `
  -VMName MyVM `
  -Name "MicrosoftMonitoringAgent" `
  -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
  -ExtensionType "MicrosoftMonitoringAgent" `
  -TypeHandlerVersion "1.0" `
  -Settings @{"workspaceId" = $workspace.CustomerId} `
  -ProtectedSettings @{"workspaceKey" = (Get-AzOperationalInsightsWorkspaceSharedKeys -ResourceGroupName MyRG -Name MyWorkspace).PrimarySharedKey}

# Network Watcher operations
$networkWatcher = Get-AzNetworkWatcher -ResourceGroupName NetworkWatcherRG -Name NetworkWatcher_eastus

# Test connectivity
Test-AzNetworkWatcherConnectivity `
  -NetworkWatcher $networkWatcher `
  -SourceId $vm1.Id `
  -DestinationId $vm2.Id `
  -DestinationPort 80

# IP flow verify
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vm.Id `
  -Direction Inbound `
  -Protocol TCP `
  -LocalIPAddress 10.0.0.4 `
  -LocalPort 80 `
  -RemoteIPAddress 10.0.0.5 `
  -RemotePort 4000

# Create Connection Monitor
New-AzNetworkWatcherConnectionMonitor `
  -NetworkWatcher $networkWatcher `
  -Name MyConnectionMonitor `
  -SourceResourceId $vm1.Id `
  -DestinationResourceId $vm2.Id `
  -DestinationPort 80

# Enable NSG flow logs
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName MyRG -Name MyNSG
$sa = Get-AzStorageAccount -ResourceGroupName MyRG -Name mystorageaccount

Set-AzNetworkWatcherFlowLog `
  -NetworkWatcher $networkWatcher `
  -TargetResourceId $nsg.Id `
  -StorageId $sa.Id `
  -EnableFlowLog $true `
  -FormatType Json `
  -FormatVersion 2 `
  -EnableRetention $true `
  -RetentionPolicyDays 30
```

### Additional Information

#### Common KQL Queries

```kql
// Top 10 resources by error count
AzureActivity
| where Level == "Error"
| summarize ErrorCount = count() by ResourceId
| top 10 by ErrorCount desc

// VM restart events
AzureActivity
| where OperationName == "Restart Virtual Machine"
| project TimeGenerated, ResourceGroup, Resource, Caller

// Failed login attempts
SigninLogs
| where ResultType != 0
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress
| where FailedAttempts > 5

// Storage account throttling
StorageBlobLogs
| where StatusCode == 503
| summarize ThrottleCount = count() by bin(TimeGenerated, 5m), AccountName
```

#### Alert Rule Best Practices

1. Set appropriate evaluation frequency and window size
2. Use dynamic thresholds for adaptive alerting
3. Configure alert suppression to avoid noise
4. Test alerts before enabling in production
5. Use severity levels consistently
6. Document alert purpose and remediation steps

#### Network Monitoring Tools Comparison

|Tool|Purpose|Key Features|
|---|---|---|
|IP Flow Verify|Check NSG rules|Source/destination validation|
|Next Hop|Routing analysis|Effective route determination|
|Packet Capture|Traffic analysis|Filtered network traces|
|Connection Monitor|Continuous monitoring|Multi-point connectivity|
|Connection Troubleshoot|One-time test|Point-to-point diagnostics|

#### Common Monitoring Scenarios

1. **VM High CPU Alert**: Metric alert when CPU > 80% for 5 minutes
2. **Storage Account Latency**: Monitor E2E latency > 1000ms
3. **Failed Deployments**: Activity log alerts for deployment failures
4. **Network Connectivity**: Connection monitor for critical services
5. **Application Errors**: Log alerts based on custom application logs