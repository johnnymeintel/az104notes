# Create and Configure Azure App Service

### Objectives

- **Provision an App Service plan**
    - **Pricing Tiers**:
        - Free (F1): Limited features, shared infrastructure
        - Shared (D1): Shared infrastructure, custom domains
        - Basic (B1-B3): Dedicated compute, manual scaling
        - Standard (S1-S3): Auto-scaling, staging slots, backups
        - Premium (P1-P3): Enhanced performance, more slots
        - PremiumV2 (P1v2-P3v2): Dv2-series VMs
        - PremiumV3 (P1v3-P3v3): Dv3-series VMs
        - Isolated (I1-I3): App Service Environment, VNet isolation
    - **Operating System Options**:
        - Windows: .NET, .NET Core, Node.js, PHP, Java
        - Linux: Node.js, Python, PHP, .NET Core, Ruby, Java
    - **Region Selection**: Must match app location
    - **Resource Group**: Logical container for related resources

- **Configure scaling for an App Service plan**
    - **Manual Scaling**:
        - Scale up/down: Change pricing tier
        - Scale out/in: Change instance count manually
        - Available in Basic tier and above
    - **Automatic Scaling** (Standard tier and above):
        - Metric-based: CPU, Memory, HTTP Queue Length
        - Schedule-based: Time of day, day of week
        - Combined rules: Multiple conditions
    - **Scaling Limits**:
        - Basic: Up to 3 instances
        - Standard: Up to 10 instances
        - Premium: Up to 30 instances
        - Isolated: Up to 100 instances
    - **Scaling Considerations**:
        - Stateless applications scale better
        - Session affinity affects scaling efficiency
        - Cool-down periods prevent rapid scaling

- **Create an App Service**
    - **Runtime Stack Options**:
        - .NET, .NET Core, ASP.NET
        - Java (8, 11, 17)
        - Node.js
        - PHP
        - Python
        - Ruby (Linux only)
        - Docker containers
    - **Configuration Settings**:
        - Application settings (environment variables)
        - Connection strings
        - Stack settings (runtime version)
        - Platform settings (32/64-bit, WebSockets)
    - **Deployment Options**:
        - Git deployment
        - GitHub Actions
        - Azure DevOps
        - FTP/FTPS
        - ZIP deploy

- **Configure certificates and Transport Layer Security (TLS) for an App Service**
    - **Certificate Types**:
        - App Service Managed Certificate (free)
        - Purchased SSL certificates
        - Uploaded certificates (.pfx, .cer)
        - Key Vault certificates
    - **TLS/SSL Settings**:
        - Minimum TLS version (1.0, 1.1, 1.2)
        - HTTPS Only enforcement
        - Client certificate mode (optional, required)
    - **Certificate Binding**:
        - SNI SSL (Server Name Indication)
        - IP SSL (requires dedicated IP)
    - **Free Managed Certificate Requirements**:
        - Standard tier or higher
        - Not available for root domains
        - Auto-renewal every 6 months

- **Map an existing custom DNS name to an App Service**
    - **Domain Types**:
        - Root/apex domain (contoso.com)
        - Subdomain (www.contoso.com)
        - Wildcard domain (*.contoso.com)
    - **DNS Record Types**:
        - A record: Maps to IP address
        - CNAME record: Maps to Azure hostname
        - TXT record: Domain verification
    - **Verification Process**:
        - Add verification TXT record
        - Configure A or CNAME record
        - Add custom domain in App Service
        - Bind SSL certificate

- **Configure backup for an App Service**
    - **Requirements**:
        - Standard tier or higher
        - Azure Storage account in same subscription
        - Storage container for backups
    - **Backup Components**:
        - App configuration
        - File content
        - Database (optional, connection string required)
    - **Backup Types**:
        - Manual: On-demand backups
        - Scheduled: Automatic at intervals
    - **Retention Settings**:
        - Keep all backups
        - Retention days (0-9999)
        - Keep at least one backup
    - **Size Limits**:
        - 10GB for app + database
        - 4GB for database alone

- **Configure networking settings for an App Service**
    - **Inbound Features**:
        - App-assigned address
        - Access restrictions (IP-based)
        - Service endpoints
        - Private endpoints
    - **Outbound Features**:
        - Hybrid Connections
        - Gateway-required VNet Integration
        - Regional VNet Integration
        - NAT Gateway
    - **Network Security**:
        - IP restrictions
        - Service tags
        - Mutual TLS authentication
        - Virtual network integration

- **Configure deployment slots for an App Service**
    - **Slot Capabilities**:
        - Separate instance of app
        - Different configuration settings
        - Unique hostname
        - Swap with production
    - **Configuration Options**:
        - Slot-specific settings
        - Deployment slot settings
        - Connection strings
        - App settings
    - **Swap Operations**:
        - Swap with preview
        - Auto swap (DevOps scenarios)
        - Swap with traffic routing
    - **Traffic Routing**:
        - Percentage-based routing
        - Manual routing
        - Testing in production
    - **Slot Limits**:
        - Standard: 5 slots
        - Premium: 20 slots
        - Isolated: 20 slots

### Common Commands

```bash
# Azure CLI Commands

# Create App Service Plan
az appservice plan create \
  --name MyPlan \
  --resource-group MyRG \
  --location eastus \
  --sku S1 \
  --is-linux false

# Create Linux App Service Plan
az appservice plan create \
  --name MyLinuxPlan \
  --resource-group MyRG \
  --location eastus \
  --sku P1V2 \
  --is-linux true

# Scale App Service Plan (scale up)
az appservice plan update \
  --name MyPlan \
  --resource-group MyRG \
  --sku P1V2

# Scale App Service Plan (scale out)
az appservice plan update \
  --name MyPlan \
  --resource-group MyRG \
  --number-of-workers 3

# Create Web App
az webapp create \
  --name MyWebApp \
  --resource-group MyRG \
  --plan MyPlan \
  --runtime "DOTNET|5.0"

# Create Linux Web App
az webapp create \
  --name MyLinuxApp \
  --resource-group MyRG \
  --plan MyLinuxPlan \
  --runtime "PYTHON|3.9"

# Configure autoscale settings
az monitor autoscale create \
  --resource-group MyRG \
  --name autoscale1 \
  --resource MyPlan \
  --resource-type Microsoft.Web/serverfarms \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add autoscale rule
az monitor autoscale rule create \
  --resource-group MyRG \
  --autoscale-name autoscale1 \
  --condition "CpuPercentage > 70 avg 5m" \
  --scale out 2

# Create managed certificate
az webapp config ssl create \
  --resource-group MyRG \
  --name MyWebApp \
  --hostname www.contoso.com

# Upload custom certificate
az webapp config ssl upload \
  --resource-group MyRG \
  --name MyWebApp \
  --certificate-file ~/certs/certificate.pfx \
  --certificate-password "CertificatePassword"

# Bind certificate to custom domain
az webapp config ssl bind \
  --resource-group MyRG \
  --name MyWebApp \
  --certificate-thumbprint [thumbprint] \
  --ssl-type SNI

# Configure TLS settings
az webapp config set \
  --resource-group MyRG \
  --name MyWebApp \
  --min-tls-version 1.2 \
  --https-only true

# Add custom domain
az webapp config hostname add \
  --resource-group MyRG \
  --webapp-name MyWebApp \
  --hostname www.contoso.com

# Create backup
az webapp config backup create \
  --resource-group MyRG \
  --webapp-name MyWebApp \
  --container-url "https://account.blob.core.windows.net/container?sastoken" \
  --backup-name "backup1"

# Configure scheduled backup
az webapp config backup update \
  --resource-group MyRG \
  --webapp-name MyWebApp \
  --frequency 1d \
  --retain-one true \
  --retention-period-in-days 30

# Configure IP restrictions
az webapp config access-restriction add \
  --resource-group MyRG \
  --name MyWebApp \
  --priority 100 \
  --action Allow \
  --ip-address 192.168.1.0/24

# Configure VNet integration
az webapp vnet-integration add \
  --resource-group MyRG \
  --name MyWebApp \
  --vnet MyVNet \
  --subnet MySubnet

# Create deployment slot
az webapp deployment slot create \
  --name MyWebApp \
  --resource-group MyRG \
  --slot staging

# Configure slot-specific setting
az webapp config appsettings set \
  --resource-group MyRG \
  --name MyWebApp \
  --slot staging \
  --settings Environment=Staging

# Swap deployment slots
az webapp deployment slot swap \
  --resource-group MyRG \
  --name MyWebApp \
  --slot staging \
  --target-slot production

# Configure traffic routing
az webapp traffic-routing set \
  --resource-group MyRG \
  --name MyWebApp \
  --distribution staging=20
```

```powershell
# PowerShell Commands

# Create App Service Plan
New-AzAppServicePlan `
  -Name "MyPlan" `
  -ResourceGroupName "MyRG" `
  -Location "eastus" `
  -Tier "Standard" `
  -WorkerSize "Small" `
  -NumberOfWorkers 1

# Create Linux App Service Plan
New-AzAppServicePlan `
  -Name "MyLinuxPlan" `
  -ResourceGroupName "MyRG" `
  -Location "eastus" `
  -Tier "PremiumV2" `
  -Linux

# Scale App Service Plan (scale up)
Set-AzAppServicePlan `
  -Name "MyPlan" `
  -ResourceGroupName "MyRG" `
  -Tier "PremiumV2" `
  -WorkerSize "Medium"

# Scale App Service Plan (scale out)
Set-AzAppServicePlan `
  -Name "MyPlan" `
  -ResourceGroupName "MyRG" `
  -NumberOfWorkers 3

# Create Web App
New-AzWebApp `
  -Name "MyWebApp" `
  -ResourceGroupName "MyRG" `
  -AppServicePlan "MyPlan" `
  -Location "eastus"

# Set runtime stack
Set-AzWebApp `
  -Name "MyWebApp" `
  -ResourceGroupName "MyRG" `
  -AppSettings @{
    "WEBSITE_NODE_DEFAULT_VERSION" = "14-lts"
  }

# Create managed certificate
New-AzWebAppSSLBinding `
  -ResourceGroupName "MyRG" `
  -WebAppName "MyWebApp" `
  -Name "www.contoso.com" `
  -SslState "SniEnabled"

# Upload certificate
New-AzWebAppCertificate `
  -ResourceGroupName "MyRG" `
  -WebAppName "MyWebApp" `
  -CertificateFilePath "~/certs/certificate.pfx" `
  -CertificatePassword (ConvertTo-SecureString "password" -AsPlainText -Force)

# Configure TLS settings
Set-AzWebApp `
  -Name "MyWebApp" `
  -ResourceGroupName "MyRG" `
  -HttpsOnly $true `
  -MinimumTlsVersion "1.2"

# Add custom domain
Set-AzWebAppHostnameDomainBinding `
  -ResourceGroupName "MyRG" `
  -WebAppName "MyWebApp" `
  -Name "www.contoso.com"

# Create backup
New-AzWebAppBackup `
  -ResourceGroupName "MyRG" `
  -Name "MyWebApp" `
  -BackupName "backup1" `
  -StorageAccountUrl "https://account.blob.core.windows.net/container?sastoken"

# Configure access restrictions
Add-AzWebAppAccessRestrictionRule `
  -ResourceGroupName "MyRG" `
  -WebAppName "MyWebApp" `
  -Name "AllowOffice" `
  -Priority 100 `
  -Action "Allow" `
  -IpAddress "192.168.1.0/24"

# Create deployment slot
New-AzWebAppSlot `
  -ResourceGroupName "MyRG" `
  -Name "MyWebApp" `
  -Slot "staging"

# Configure slot settings
Set-AzWebAppSlot `
  -ResourceGroupName "MyRG" `
  -Name "MyWebApp" `
  -Slot "staging" `
  -AppSettings @{
    "Environment" = "Staging"
  }

# Swap deployment slots
Switch-AzWebAppSlot `
  -ResourceGroupName "MyRG" `
  -Name "MyWebApp" `
  -SourceSlotName "staging" `
  -DestinationSlotName "production"

# Get App Service details
Get-AzWebApp `
  -ResourceGroupName "MyRG" `
  -Name "MyWebApp"
```

### Additional Information

#### App Service Plan Pricing Tiers Comparison

|Feature|Free/Shared|Basic|Standard|Premium|Isolated|
|---|---|---|---|---|---|
|Custom Domain|❌/✅|✅|✅|✅|✅|
|SSL|❌|✅|✅|✅|✅|
|Auto Scale|❌|❌|✅|✅|✅|
|Deployment Slots|❌|❌|5|20|20|
|Backup/Restore|❌|❌|✅|✅|✅|
|VNet Integration|❌|❌|❌|✅|✅|

#### Runtime Stack Support

|Platform|Windows|Linux|
|---|---|---|
|.NET|✅|✅ (.NET Core)|
|Java|✅|✅|
|Node.js|✅|✅|
|PHP|✅|✅|
|Python|❌|✅|
|Ruby|❌|✅|

#### Common Scaling Scenarios

1. **Business Hours Scaling**: Scale up during 9-5, scale down evenings
2. **Weekend Scaling**: Different instance counts for weekdays/weekends
3. **Load-Based Scaling**: Scale based on CPU/memory thresholds
4. **Event-Based Scaling**: Pre-scale for known traffic events
5. **Geographic Scaling**: Different scales for different regions

#### SSL/TLS Best Practices

- Always use TLS 1.2 or higher
- Implement HTTPS redirect
- Use managed certificates for simplicity
- Consider wildcard certificates for subdomains
- Monitor certificate expiration

#### Backup Strategy Guidelines

- Daily backups for production apps
- Retain backups for compliance period
- Test restore procedures regularly
- Include database in backups when applicable
- Store backups in geo-redundant storage

#### Common Exam Scenarios

- Choosing appropriate App Service plan tier
- Understanding scaling limitations per tier
- SSL certificate binding options
- Deployment slot swap behavior
- Network integration requirements
- Backup configuration and limitations