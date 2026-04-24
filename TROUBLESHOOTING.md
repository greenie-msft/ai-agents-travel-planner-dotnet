# Troubleshooting Guide — Travel Planner Application

This guide helps you resolve common issues when deploying and running the Travel Planner application.

---

## Azure OpenAI Region Availability — `RegionDoesNotAllowProvisioning`

When deploying with `azd up`, you may encounter errors like:

```
RegionDoesNotAllowProvisioning: Location 'East US' is not accepting creation
of new Cognitive Services at this time.
```

This means your Azure subscription cannot create new OpenAI services in the selected region.

### How to Find a Working Region

Use the Azure CLI to test which regions allow Cognitive Services (Azure OpenAI) provisioning:

#### Quick Single-Region Test (Bash)

```bash
# Create a temporary resource group
az group create --name rg-openai-test-temp --location eastus --output none

# Test a specific region
az cognitiveservices account create \
  --name "openaitest-$(shuf -i 10000-99999 -n 1)" \
  --resource-group rg-openai-test-temp \
  --kind OpenAI \
  --location <REGION_TO_TEST> \
  --sku s0

# Clean up
az group delete --name rg-openai-test-temp --yes --no-wait
```

If it succeeds, the region works. If blocked, try another.

#### Batch Test Multiple Regions (Bash)

```bash
rg="rg-openai-test-$RANDOM"
az group create --name $rg --location eastus --output none

for r in eastus eastus2 westus westus3 canadacentral swedencentral \
         uksouth southcentralus; do
    name="openaitest$RANDOM"
    if az cognitiveservices account create --name "$name" --resource-group "$rg" \
        --kind OpenAI --location "$r" --sku s0 &>/dev/null; then
        echo "✅ $r"
        az cognitiveservices account delete --name "$name" --resource-group "$rg" --yes &>/dev/null
    else
        echo "❌ $r"
    fi
done

# Clean up
az group delete --name $rg --yes --no-wait
```

#### Batch Test Multiple Regions (PowerShell)

```powershell
$rg = "rg-openai-test-$(Get-Random -Maximum 9999)"
az group create --name $rg --location eastus --output none

$regions = @("eastus", "eastus2", "westus", "westus3", "canadacentral", 
             "swedencentral", "uksouth", "southcentralus")

foreach ($r in $regions) {
    $name = "openaitest$(Get-Random -Maximum 99999)"
    $result = az cognitiveservices account create --name $name --resource-group $rg `
        --kind OpenAI --location $r --sku s0 2>&1
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✅ $r"
        az cognitiveservices account delete --name $name --resource-group $rg --yes 2>$null
    } else {
        Write-Host "❌ $r"
    }
}

# Clean up
az group delete --name $rg --yes --no-wait
```

### Once You Find a Working Region

Set it in your azd environment and deploy:

```bash
azd env set AZURE_LOCATION <working-region>
azd up
```

### Common Regions Often Available for OpenAI

- `eastus` (most common)
- `westus`
- `swedencentral`
- `canadacentral`
- `uksouth`

---

## Redis Connection Failures

### Error: `Connection refused` or `WRONGTYPE` 

**Local Development:**

Ensure Redis is running locally:

```bash
# On macOS with Homebrew
brew services start redis

# Or with Docker
docker run -d -p 6379:6379 redis:latest
```

**Verify connection:**

```bash
redis-cli ping
# Should return: PONG
```

### Error: `Azure Cache for Redis` — `NOAUTH` or connection timeout

Check your `local.settings.json` has the correct connection string:

```json
"REDIS_CONNECTION_STRING": "your-redis-cache.redis.cache.windows.net:6380?ssl=True"
```

In Azure, you need:
- The connection string from Azure Portal → Your Redis → Access Keys
- SSL enabled (port 6380)
- Firewall rules allowing your Static Web App or Function App

---

## Durable Task Scheduler (DTS) Connection Issues

### Error: `DTS connection failed` or `TaskHub not found`

Check your connection string in `local.settings.json`:

```json
"DURABLE_TASK_SCHEDULER_CONNECTION_STRING": "Endpoint=http://localhost:8080;Authentication=None"
```

For local development, ensure the DTS emulator is running or using the correct endpoint.

For Azure Foundry, use:

```bash
azd env get-values | grep DURABLE_TASK_SCHEDULER
```

---

## Azure Functions Deployment Issues

### Error: `Function app not found` or `Zip deploy failed`

**Check function app exists:**

```bash
az functionapp list --output table
```

**Check deployment logs:**

```bash
az functionapp deployment source show-build-status --name <function-app-name> \
  --resource-group <resource-group> --deployment-status-details
```

**Rebuild and redeploy:**

```bash
azd up
```

### Error: `cold start` or `function timeout`

- Check function timeout settings in Azure Portal
- Verify function is using Flex Consumption plan for better performance
- Consider using Premium plan if sustained high throughput

---

## Static Web App (SWA) Deployment Issues

### Error: `Build failed` with npm/node errors

Check `Frontend/package.json` dependencies are compatible:

```bash
cd Frontend
npm install
npm run build
```

### Error: `CORS` — `Access to XMLHttpRequest blocked`

Verify `staticwebapp.config.json` has correct API routing:

```json
{
  "routes": [
    {
      "route": "/api/*",
      "methods": ["GET", "POST", "PUT", "DELETE"],
      "allowedRoles": ["anonymous", "authenticated"]
    }
  ]
}
```

---

## Authentication Issues

### Error: `Unauthorized` or Entra ID redirect failing

Verify Entra ID app registration:

```bash
az ad app list --filter "displayName eq 'travel-planner'" --output table
```

Check app has correct redirect URIs for your Static Web App URL.

---

## Common Errors Overview Table

| Error | Cause | Solution |
|-------|-------|----------|
| `RegionDoesNotAllowProvisioning` | Azure OpenAI not available in region | Run region test script above |
| `Connection refused` (Redis) | Redis not running locally | `brew services start redis` |
| `NOAUTH` (Redis) | Missing/wrong connection string | Update `REDIS_CONNECTION_STRING` |
| `DTS connection failed` | DTS endpoint unreachable | Verify connection string & endpoint |
| `Cold start` (Functions) | Consumption plan overhead | Upgrade to Flex or Premium plan |
| `CORS` (SWA) | API routing misconfigured | Update `staticwebapp.config.json` |
| `Unauthorized` | Entra ID misconfigured | Verify app registration & redirect URIs |

---

## Getting Help

- **Azure Resources Status:** https://status.azure.com
- **Azure Functions Docs:** https://docs.microsoft.com/azure/azure-functions
- **Static Web Apps Docs:** https://docs.microsoft.com/azure/static-web-apps
- **Azure OpenAI Docs:** https://docs.microsoft.com/azure/ai-services/openai
