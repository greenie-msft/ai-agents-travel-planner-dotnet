# Agent Deployment Guide

Use this guide when an automation agent is asked to deploy this repository.

## Goal

Deploy the Travel Planner app to Azure and verify that both backend and frontend are working.

## Required Tools

- Azure CLI (az)
- Azure Developer CLI (azd)
- .NET 10 SDK
- Node.js 18+
- Azure Functions Core Tools v4

## Run From

Run all commands from the repository root.

## Standard Deployment Flow

1. Authenticate

```bash
azd auth login
az login
```

2. Select a model-capable region

```bash
azd env set MODEL_LOCATION <region>
```

3. Provision and deploy infrastructure + services

```bash
azd up
```

Important:
- Azure Cache for Redis provisioning can take significantly longer than other resources.
- It is normal for `azd up` to appear paused while Redis is in `Creating` state.
- Do not abort early unless Azure reports an explicit terminal failure.

4. Redeploy API after first deployment

```bash
azd deploy api
```

Why this step exists:
- The Function App endpoint may not be finalized until provisioning completes.
- A second API deploy helps ensure frontend build inputs resolve correctly.

5. Deploy frontend

```bash
azd package web
azd deploy web
```

## Verification Steps

1. Confirm endpoints were produced by azd output.
2. Verify API responds:

```bash
curl -i -X POST "https://<function-app>.azurewebsites.net/api/travel-planner" \
  -H "Content-Type: application/json" \
  -d '{"userName":"Test","preferences":"beach","duration":5,"budget":"2000 USD","travelDates":"June 2026"}'
```

Expected: HTTP 202 with statusQueryUrl.

3. Poll status URL until progress reaches waiting-for-approval or completed state.
4. Verify frontend loads and can call /api endpoints.

## If Deployment Fails

Use the troubleshooting skill included in this repo:
- Skill file: .copilot/skills/troubleshooting-travel-planner/SKILL.md
- Skill name: Troubleshooting Travel Planner

This skill covers:
- Azure OpenAI region availability issues
- Redis connectivity issues
- Durable Task Scheduler connectivity
- Azure Functions deployment failures
- Static Web App and CORS issues
- Authentication and Entra ID issues

## Fast Recovery Checklist

Run these in order when the app deploys but frontend cannot call API:

```bash
azd deploy api
azd deploy web
```

Then retest the frontend and API endpoints.
