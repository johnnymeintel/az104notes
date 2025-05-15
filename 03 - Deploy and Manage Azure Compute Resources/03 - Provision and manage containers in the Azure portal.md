# Provision and Manage Containers in the Azure Portal

### Objectives

- **Create and manage an Azure container registry**
    - **Registry Tiers**:
        - Basic: Development/testing, limited storage/throughput
        - Standard: Production workloads, increased storage/throughput
        - Premium: Geo-replication, content trust, private endpoints
    - **Key Features**:
        - Docker image storage and management
        - Helm chart repositories
        - OCI artifacts support
        - Vulnerability scanning (Standard/Premium)
        - Content trust with digital signing (Premium)
    - **Access Methods**:
        - Admin user (username/password) - not recommended for production
        - Service principal authentication
        - Azure AD individual identity
        - Managed identity access
    - **Geo-replication** (Premium only):
        - Multiple registry replicas across regions
        - Local registry access for distributed teams
        - Resilience against regional failures

- **Provision a container by using Azure Container Instances**
    - **Quick Deployment Features**:
        - No VM management required
        - Per-second billing
        - Linux and Windows containers
        - Public IP and FQDN assignment
        - Custom DNS name label
    - **Container Configuration**:
        - CPU and memory allocation (0.5-4 vCPU, 0.5-16 GB RAM)
        - GPU support (Preview)
        - Environment variables
        - Command override
        - Restart policies: Always, Never, OnFailure
    - **Networking Options**:
        - Public IP with DNS name
        - Private deployment in VNet
        - Port configuration
    - **Storage Options**:
        - Azure Files volume mount
        - Empty directory volume
        - Git repository volume
        - Secret volume

- **Provision a container by using Azure Container Apps**
    - **Platform Features**:
        - Built on Kubernetes
        - Serverless container hosting
        - Automatic scaling to zero
        - HTTPS ingress with custom domains
        - Built-in traffic splitting
    - **Application Types**:
        - Web APIs and microservices
        - Background processing jobs
        - Event-driven applications
    - **Deployment Options**:
        - Single container revision
        - Multiple container apps in environment
        - Dapr integration for microservices
    - **Environment Configuration**:
        - Container Apps Environment (shared platform)
        - Virtual network integration
        - Log Analytics workspace
        - Internal/external ingress

- **Manage sizing and scaling for containers, including Azure Container Instances and Azure Container Apps**
    - **Container Instances Sizing**:
        - Fixed resource allocation
        - CPU: 0.5 to 4 cores per container
        - Memory: 0.5 to 16 GB per container
        - Container groups for multi-container scenarios
        - Manual scaling only (deploy more instances)
    - **Container Apps Scaling**:
        - Automatic horizontal scaling
        - Scale rules based on:
            - HTTP traffic
            - CPU/Memory utilization
            - Azure Service Bus queue length
            - Custom metrics via KEDA
        - Min/max replica configuration
        - Scale to zero capability
    - **Cost Optimization**:
        - Container Instances: Per-second billing
        - Container Apps: Per-second active billing
        - Choose based on workload patterns
        - Consider reserved capacity for predictable workloads

### Common Commands

```bash
# Azure CLI Commands

# Create Container Registry
az acr create \
  --resource-group MyRG \
  --name myregistry \
  --sku Standard \
  --location eastus

# Enable admin user
az acr update \
  --name myregistry \
  --admin-enabled true

# Get login credentials
az acr credential show \
  --name myregistry

# Login to registry
az acr login --name myregistry

# Build image in ACR
az acr build \
  --registry myregistry \
  --image myapp:v1 .

# List images in registry
az acr repository list \
  --name myregistry

# Show tags for an image
az acr repository show-tags \
  --name myregistry \
  --repository myapp

# Create Container Instance from Docker Hub
az container create \
  --resource-group MyRG \
  --name mycontainer \
  --image nginx \
  --cpu 1 \
  --memory 1.5 \
  --ports 80 \
  --dns-name-label mydnsname

# Create Container Instance from ACR
az container create \
  --resource-group MyRG \
  --name myappcontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-username $(az acr credential show --name myregistry --query username -o tsv) \
  --registry-password $(az acr credential show --name myregistry --query passwords[0].value -o tsv) \
  --cpu 2 \
  --memory 4 \
  --ports 80 443

# Create Container Instance with environment variables
az container create \
  --resource-group MyRG \
  --name myenvcontainer \
  --image myapp:latest \
  --environment-variables KEY1=value1 KEY2=value2 \
  --secure-environment-variables SECRET_KEY=secretvalue

# Create Container Instance with Azure Files volume
az container create \
  --resource-group MyRG \
  --name myfilecontainer \
  --image myapp:latest \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name myshare \
  --azure-file-volume-mount-path /mount/files

# Show container logs
az container logs \
  --resource-group MyRG \
  --name mycontainer

# Attach to running container
az container attach \
  --resource-group MyRG \
  --name mycontainer

# Create Container Apps environment
az containerapp env create \
  --resource-group MyRG \
  --name mycontainerapps-env \
  --location eastus

# Deploy Container App
az containerapp create \
  --resource-group MyRG \
  --name mycontainerapp \
  --environment mycontainerapps-env \
  --image nginx \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10

# Update Container App with scaling rules
az containerapp update \
  --resource-group MyRG \
  --name mycontainerapp \
  --set-env-vars "API_KEY=value" \
  --cpu 0.5 \
  --memory 1.0

# Create Container App with custom scaling
az containerapp create \
  --resource-group MyRG \
  --name myscalableapp \
  --environment mycontainerapps-env \
  --image myapp:latest \
  --target-port 80 \
  --ingress external \
  --min-replicas 1 \
  --max-replicas 20 \
  --scale-rule-name http-rule \
  --scale-rule-type http \
  --scale-rule-metadata concurrentRequests=50
```

```powershell
# PowerShell Commands

# Create Container Registry
New-AzContainerRegistry `
  -ResourceGroupName "MyRG" `
  -Name "myregistry" `
  -Sku "Standard" `
  -Location "eastus"

# Enable admin user
Update-AzContainerRegistry `
  -ResourceGroupName "MyRG" `
  -Name "myregistry" `
  -EnableAdminUser

# Get registry credentials
Get-AzContainerRegistryCredential `
  -ResourceGroupName "MyRG" `
  -Name "myregistry"

# Create Container Instance
New-AzContainerGroup `
  -ResourceGroupName "MyRG" `
  -Name "mycontainer" `
  -Image "nginx" `
  -Location "eastus" `
  -Cpu 1 `
  -MemoryInGB 1.5 `
  -Port 80 `
  -DnsNameLabel "mydnsname"

# Create Container Instance from ACR
$registryCred = Get-AzContainerRegistryCredential -ResourceGroupName "MyRG" -Name "myregistry"
$securePassword = ConvertTo-SecureString $registryCred.Password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($registryCred.Username, $securePassword)

New-AzContainerGroup `
  -ResourceGroupName "MyRG" `
  -Name "myappcontainer" `
  -Image "myregistry.azurecr.io/myapp:v1" `
  -Location "eastus" `
  -Cpu 2 `
  -MemoryInGB 4 `
  -Port 80,443 `
  -RegistryCredential $credential

# Create Container Instance with environment variables
$envVars = @{
  "KEY1" = "value1"
  "KEY2" = "value2"
}

$secureEnvVars = @{
  "SECRET_KEY" = "secretvalue"
}

New-AzContainerGroup `
  -ResourceGroupName "MyRG" `
  -Name "myenvcontainer" `
  -Image "myapp:latest" `
  -Location "eastus" `
  -EnvironmentVariable $envVars `
  -SecureEnvironmentVariable $secureEnvVars

# Get container logs
Get-AzContainerInstanceLog `
  -ResourceGroupName "MyRG" `
  -ContainerGroupName "mycontainer"

# Create Container App Environment (requires Az.App module)
New-AzContainerAppManagedEnv `
  -ResourceGroupName "MyRG" `
  -Name "mycontainerapps-env" `
  -Location "eastus"

# Create Container App
New-AzContainerApp `
  -ResourceGroupName "MyRG" `
  -Name "mycontainerapp" `
  -Location "eastus" `
  -ManagedEnvironmentId "/subscriptions/.../mycontainerapps-env" `
  -Image "nginx" `
  -TargetPort 80 `
  -IngressExternal `
  -MinReplica 0 `
  -MaxReplica 10

# Get Container Instance details
Get-AzContainerGroup `
  -ResourceGroupName "MyRG" `
  -Name "mycontainer"

# Remove container resources
Remove-AzContainerGroup `
  -ResourceGroupName "MyRG" `
  -Name "mycontainer"

Remove-AzContainerRegistry `
  -ResourceGroupName "MyRG" `
  -Name "myregistry"
```

### Additional Information

#### Container Registry SKU Comparison

|Feature|Basic|Standard|Premium|
|---|---|---|---|
|Storage|10 GB|100 GB|500 GB|
|Throughput|Low|Medium|High|
|Geo-replication|❌|❌|✅|
|Content Trust|❌|❌|✅|
|Private Link|❌|❌|✅|
|Customer-managed keys|❌|❌|✅|

#### Container Instances vs Container Apps

|Feature|Container Instances|Container Apps|
|---|---|---|
|Use Case|Short-lived tasks, simple containers|Microservices, web apps|
|Scaling|Manual only|Automatic (0-N replicas)|
|Networking|Basic (public IP or VNet)|Advanced (ingress, traffic split)|
|Platform|Standalone containers|Kubernetes-based|
|Pricing|Per-second|Per-second when active|
|Complexity|Simple|More features, more complex|

#### Container Sizing Guidelines

- **Container Instances**:
    - Development/test: 0.5 CPU, 1 GB RAM
    - Web apps: 1-2 CPU, 2-4 GB RAM
    - Processing jobs: 2-4 CPU, 4-8 GB RAM
- **Container Apps**:
    - Microservices: 0.25-0.5 CPU, 0.5-1 GB RAM
    - APIs: 0.5-1 CPU, 1-2 GB RAM
    - Background jobs: 0.5-2 CPU, 1-4 GB RAM

#### Common Scaling Patterns

1. **HTTP-based scaling**: Scale based on concurrent requests
2. **Queue-based scaling**: Scale based on message queue length
3. **CPU/Memory scaling**: Scale based on resource utilization
4. **Schedule-based scaling**: Scale based on time of day
5. **Event-driven scaling**: Scale based on external events

#### Security Best Practices

- Use managed identities instead of registry admin user
- Enable vulnerability scanning in Container Registry
- Use private endpoints for registry access
- Implement network policies for container apps
- Store secrets in Key Vault, not environment variables
- Regular base image updates and scanning

#### Common Exam Scenarios

- Choosing between Container Instances and Container Apps
- Understanding registry SKU capabilities
- Configuring appropriate scaling rules
- Container networking options
- Security considerations for container deployments