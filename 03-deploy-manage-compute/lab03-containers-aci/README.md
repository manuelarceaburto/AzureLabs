# Lab 03: Deploy and Manage Containers with Azure Container Instances

## Objectives

By completing this lab, you will be able to:
- [ ] Create and manage container images
- [ ] Deploy containers to Azure Container Instances (ACI)
- [ ] Configure container resources and networking
- [ ] Manage container logs and monitoring
- [ ] Deploy multi-container applications

## Prerequisites

- Active Azure subscription
- Docker installed (for building images)
- Azure Container Registry (ACR) or Docker Hub account
- Azure CLI or PowerShell
- Basic understanding of Docker and containers
- Estimated time: 45 minutes

## Lab Scenario

Contoso Corporation has containerized applications that need to run in Azure without managing infrastructure. You'll deploy containers using Azure Container Instances for quick, scalable deployment.

## Architecture

```
Azure Subscription
├── Azure Container Registry
│   ├── Image: web-app:latest
│   └── Image: api-service:latest
├── Container Instances
│   ├── Container Group: app-group
│   │   ├── Container: web-app
│   │   ├── Container: api-service
│   │   └── Shared IP: public-ip
│   └── Container Group: worker-group
│       └── Container: background-job
└── Storage Account (for logs)
```

## Task 1: Create Azure Container Registry

### Instructions

1. Navigate to **Container registries** in Azure Portal
2. Click **+ Create**
3. Configure registry:
   - **Subscription**: Your subscription
   - **Resource group**: RG-Compute (or new group)
   - **Registry name**: contoso[unique-id] (must be globally unique)
   - **Location**: East US
   - **SKU**: Basic
   - Click **Create**

4. Get registry credentials:
   - Go to created registry > **Access keys**
   - Copy Login server, Username, Password
   - Enable Admin user if not already enabled

5. (Optional) Push image to registry:
   - Build image locally: `docker build -t myapp:1.0 .`
   - Tag image: `docker tag myapp:1.0 contoso.azurecr.io/myapp:1.0`
   - Push to ACR: `docker push contoso.azurecr.io/myapp:1.0`

### Verification

- [ ] Container registry is created
- [ ] Admin access is enabled
- [ ] Credentials are available
- [ ] Can authenticate to registry
- [ ] (Optional) Images are pushed

### PowerShell Example

```powershell
# Create container registry
New-AzContainerRegistry -ResourceGroupName "RG-Compute" `
    -Name "contoso$(Get-Random)" `
    -Sku "Basic" `
    -Location "eastus"

# Get registry details
$registry = Get-AzContainerRegistry -ResourceGroupName "RG-Compute"
$loginServer = $registry.LoginServer
Get-AzContainerRegistryCredential -Registry $registry
```

## Task 2: Deploy Single Container Instance

### Instructions

1. Navigate to **Container Instances** in Azure Portal
2. Click **+ Create**
3. Configure container:
   - **Resource group**: RG-Compute
   - **Container name**: web-container
   - **Image source**: Docker Hub (or ACR)
   - **Image**: nginx:latest (or custom image)
   - **CPU cores**: 1
   - **Memory (GB)**: 1
   - Click **Next: Networking**

4. Configure networking:
   - **DNS name label**: contoso-web
   - **Ports**: 80
   - **Port protocol**: TCP
   - Click **Next: Advanced**

5. Review and create

### Verification

- [ ] Container instance is created
- [ ] Container is running (state = Running)
- [ ] Public IP is assigned
- [ ] DNS name is accessible
- [ ] Container is reachable via HTTP

### Azure CLI Example

```bash
# Create container instance
az container create \
    --resource-group RG-Compute \
    --name web-container \
    --image nginx:latest \
    --dns-name-label contoso-web \
    --ports 80 \
    --cpu 1 \
    --memory 1

# Show container details
az container show \
    --resource-group RG-Compute \
    --name web-container \
    --query "containers[0].properties.instanceView.currentState"

# Get logs
az container logs \
    --resource-group RG-Compute \
    --name web-container
```

## Task 3: Deploy Multi-Container Application

### Instructions

1. Create container group with multiple containers:
   - Navigate to **Container Instances** > **+ Create**
   - **Container group name**: app-group
   - **Image source**: Docker Hub

2. Add first container:
   - **Image**: nginx:latest
   - **Name**: web-frontend
   - **CPU**: 0.5
   - **Memory**: 0.5 GB
   - **Port**: 80/TCP

3. Click **+ Add container** to add second container:
   - **Image**: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
   - **Name**: log-processor
   - **CPU**: 0.5
   - **Memory**: 0.5 GB
   - **Ports**: (none)

4. Configure networking:
   - **DNS name label**: app-group
   - **Ports**: 80

5. Create container group

### Verification

- [ ] Container group is created
- [ ] Both containers are running
- [ ] Containers can communicate (same IP)
- [ ] Web container is accessible
- [ ] Logs from both containers are available

### PowerShell Example

```powershell
# Create container group with multiple containers
$container1 = New-AzContainerInstanceObject `
    -Name "web" `
    -Image "nginx:latest" `
    -MemoryInGb 0.5 `
    -Cpu 0.5

$container2 = New-AzContainerInstanceObject `
    -Name "logger" `
    -Image "busybox" `
    -MemoryInGb 0.5 `
    -Cpu 0.5

New-AzContainerGroup -ResourceGroupName "RG-Compute" `
    -Name "app-group" `
    -Container @($container1, $container2) `
    -IpAddressType Public `
    -Port @(80) `
    -DnsNameLabel "app-group"
```

## Task 4: Manage Container Configuration

### Instructions

1. Configure environment variables:
   - Go to container > Edit
   - Add environment variables:
     - APP_ENV=production
     - LOG_LEVEL=info
   - Save changes (requires restart)

2. Configure resource limits:
   - Go to container > Edit
   - Adjust CPU and memory
   - Monitor actual usage

3. Configure restart policy:
   - Go to container > Edit
   - **Restart policy**: Always, OnFailure, or Never
   - Set for production use

4. Configure health probes:
   - Set startup probe
   - Set liveness probe
   - Monitor health status

### Verification

- [ ] Environment variables are set
- [ ] Resource limits are applied
- [ ] Restart policy is configured
- [ ] Health probes are working
- [ ] Container continues running appropriately

## Task 5: Monitor Container Logs and Performance

### Instructions

1. View container logs:
   - Go to **Container Instances** > **app-group**
   - Click **Containers** > Select container
   - Click **Logs** tab
   - Review logs in real-time
   - Can also use Azure CLI: `az container logs --name web-container`

2. Monitor resource usage:
   - Go to container group > **Monitoring** > **Metrics**
   - Create charts for:
     - CPU usage
     - Memory usage
     - Network In/Out

3. Set up alerts:
   - Create alert for high CPU/Memory usage
   - Configure email notifications

4. Use Application Insights (optional):
   - Enable Application Insights integration
   - Track performance metrics
   - Monitor exceptions

### Verification

- [ ] Can view container logs
- [ ] Metrics dashboard shows usage
- [ ] Alerts are configured
- [ ] Performance data is collected
- [ ] Can troubleshoot issues using logs

### Azure CLI Example

```bash
# Get container logs
az container logs \
    --resource-group RG-Compute \
    --name app-group \
    --container-name web-frontend

# Stream logs (live)
az container logs \
    --resource-group RG-Compute \
    --name app-group \
    --follow

# Get container group details
az container show \
    --resource-group RG-Compute \
    --name app-group \
    --query "containers[*].[name, properties.instanceView.currentState]"
```

## Task 6: Deploy from Container Registry

### Instructions

1. Push custom image to ACR:
   ```bash
   docker build -t myapp:1.0 .
   docker tag myapp:1.0 contoso.azurecr.io/myapp:1.0
   docker push contoso.azurecr.io/myapp:1.0
   ```

2. Deploy from ACR:
   - Go to **Container Instances** > **+ Create**
   - **Image source**: Azure Container Registry
   - **Registry**: Select your registry
   - **Image**: myapp
   - **Tag**: 1.0
   - Configure networking
   - Create

3. Authenticate with ACR:
   - Configure ACR credentials for authentication
   - Use managed identity if possible
   - Monitor deployment

### Verification

- [ ] Image is pushed to ACR successfully
- [ ] Container instance deploys from ACR
- [ ] Authentication works
- [ ] Container is running with correct image

## Cleanup

Remove container instances and registry:

```bash
# Remove container group
az container delete \
    --resource-group RG-Compute \
    --name app-group \
    --yes

# Remove container registry
az acr delete \
    --resource-group RG-Compute \
    --name contoso[unique-id]
```

Or PowerShell:

```powershell
Remove-AzContainerGroup -ResourceGroupName "RG-Compute" -Name "app-group" -Force
Remove-AzContainerRegistry -ResourceGroupName "RG-Compute" -Name "contoso*" -Force
```

## Review

In this lab, you:
- Created an Azure Container Registry
- Deployed single and multi-container instances
- Configured resources and networking
- Monitored container logs and performance
- Managed container lifecycle

Azure Container Instances provides quick, scalable container deployment without managing infrastructure.

## Additional Resources

- [Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/)
- [Container Registries](https://learn.microsoft.com/en-us/azure/container-registry/)
- [Container Images](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-image-formats)
- [Deploying Containers](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-deploy-instance)
- [Multi-Container Groups](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-multi-container-yaml)

## Notes for Instructors

- ACI is best for short-lived containers or non-critical workloads
- For production workloads, consider Azure Kubernetes Service (AKS)
- Container costs scale with CPU and memory allocated
- Always set resource requests and limits
- Use health probes for better reliability
- Push images to ACR for better security
- Consider using managed identities for authentication
- Monitor costs carefully as containers run continuously

---

**Lab Completed:** [Date]  
**Last Updated:** December 31, 2025
