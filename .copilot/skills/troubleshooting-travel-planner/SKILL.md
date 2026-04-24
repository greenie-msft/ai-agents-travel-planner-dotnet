---
name: Troubleshooting Travel Planner
description: >-
  Helps diagnose and resolve common issues with the Travel Planner application,
  including Azure OpenAI region availability, Redis connection problems,
  DTS connectivity, Azure Functions deployment, Static Web App issues, and
  authentication/Entra ID problems. Provides ready-to-use CLI scripts for
  testing region availability and validating infrastructure.
keywords:
  - troubleshooting
  - azure-openai
  - region-availability
  - redis
  - durable-task-scheduler
  - deployment
  - static-web-app
  - authentication
  - entra-id
  - azure-functions
topic-areas:
  - troubleshooting
  - infrastructure
  - deployment
  - infrastructure-as-code
  - devops
suggestion-triggers:
  - "deployment failed"
  - "connection refused"
  - "region not available"
  - "openai provisioning disabled"
  - "redis connection error"
  - "dts connection failed"
  - "static web app cors"
  - "function timeout"
  - "cold start"
  - "authentication failed"
  - "unauthorized"
  - "troubleshoot travel planner"
  - "why is my region blocked"
  - "how to find working region"
  - "redis not connecting"
---

# Troubleshooting Travel Planner

## Quick Diagnostics

When something goes wrong with the Travel Planner app, ask me:

- **"Why is my deployment failing?"** → I'll help diagnose the specific error
- **"Which Azure regions work for OpenAI?"** → I'll show you batch testing scripts
- **"Redis is refusing connections"** → I'll walk through connection validation
- **"My Static Web App has CORS errors"** → I'll help fix routing config
- **"Functions are timing out"** → I'll suggest plan upgrades or timeout fixes

## Key Issues This Skill Helps With

### 🌍 Azure OpenAI Region Availability
- `RegionDoesNotAllowProvisioning` errors
- Finding available regions for your subscription
- Batch testing multiple regions at once (Bash/PowerShell)

### 🔴 Redis Connection Issues
- `Connection refused` for local Redis
- `NOAUTH` or `Connection timeout` for Azure Cache for Redis
- Connection string validation

### ⚙️ Durable Task Scheduler (DTS)
- Connection string validation
- TaskHub not found errors
- Local emulator vs Azure Foundry setup

### ⚡ Azure Functions Deployment
- Zip deployment failures
- Cold start performance optimization
- Function app timeout configuration
- Plan upgrade recommendations (Flex vs Premium)

### 🌐 Static Web App Issues
- CORS blocking API calls
- Build failures with npm/Node
- Routing configuration for API endpoints

### 🔐 Authentication & Entra ID
- Unauthorized/403 errors
- Redirect URI misconfiguration
- App registration validation

## Common Commands I Can Help With

```bash
# Test OpenAI region availability
az cognitiveservices account create --name test-xyz --resource-group rg-temp \
  --kind OpenAI --location <REGION> --sku s0

# Check Redis connectivity
redis-cli -h <redis-host> ping

# Verify function app deployment
az functionapp deployment source show-build-status --name <app-name> \
  --resource-group <rg>

# List available function apps
az functionapp list --output table

# Check Static Web App routes
cat Frontend/staticwebapp.config.json
```

## How to Use This Skill

1. **Describe your error** — Include the error message if possible
2. **Tell me your scenario** — Local dev, staging, or production?
3. **Ask for a specific fix** — "Show me a region testing script"

I'll then provide:
- ✅ Root cause analysis
- ✅ Step-by-step resolution steps
- ✅ Ready-to-use CLI commands or scripts
- ✅ Links to relevant documentation

## Links to Full Guides

- **Full Troubleshooting Guide:** [TROUBLESHOOTING.md](../../TROUBLESHOOTING.md)
- **Azure Functions Docs:** https://docs.microsoft.com/azure/azure-functions
- **Static Web Apps Docs:** https://docs.microsoft.com/azure/static-web-apps
- **Azure OpenAI:** https://docs.microsoft.com/azure/ai-services/openai
- **Azure Cache for Redis:** https://docs.microsoft.com/azure/azure-cache-for-redis
