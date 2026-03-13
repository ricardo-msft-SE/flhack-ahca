---
layout: default
title: Deployment
nav_order: 9
description: "Step-by-step deployment guide for Policy Bot infrastructure, agent configuration, Teams bot manifest, and web portal publishing."
permalink: /docs/deployment/
---

# Deployment
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Azure Subscription** | Owner or Contributor role required |
| **Azure CLI** | v2.58+ — `az --version` |
| **Azure AI Foundry access** | AI Foundry hub created in your subscription |
| **Python** | 3.11+ |
| **Node.js** | 20+ (for Web Portal) |
| **Git** | 2.40+ |
| **Teams Admin access** | Required to publish the Teams Bot to your org directory |
| **GitHub Copilot (optional)** | Recommended for assisted development |

---

## Resource Group and Region

```bash
# Set variables (customise for your environment)
SUBSCRIPTION_ID="<your-subscription-id>"
RESOURCE_GROUP="rg-policybot-prod"
LOCATION="eastus2"
FOUNDRY_HUB="policybot-foundry-hub"
PROJECT_NAME="policybot"

az account set --subscription $SUBSCRIPTION_ID

az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
```

{: .note }
> All services should be in the **same region** to avoid data egress costs and cross-region latency. `eastus2` provides the broadest Azure OpenAI model availability.

---

## Step 1: Provision Azure AI Foundry Hub

```bash
# Install the ML extension
az extension add --name ml

# Create the Foundry hub
az ml workspace create \
  --kind hub \
  --resource-group $RESOURCE_GROUP \
  --name $FOUNDRY_HUB \
  --location $LOCATION \
  --public-network-access Disabled

# Create the Foundry project under the hub
az ml workspace create \
  --kind project \
  --resource-group $RESOURCE_GROUP \
  --name $PROJECT_NAME \
  --hub-id "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.MachineLearningServices/workspaces/$FOUNDRY_HUB" \
  --location $LOCATION
```

---

## Step 2: Provision Azure OpenAI and Embeddings

```bash
# Create Azure OpenAI account
az cognitiveservices account create \
  --name "policybot-aoai" \
  --resource-group $RESOURCE_GROUP \
  --kind "OpenAI" \
  --sku "S0" \
  --location $LOCATION

# Deploy GPT-4o
az cognitiveservices account deployment create \
  --name "policybot-aoai" \
  --resource-group $RESOURCE_GROUP \
  --deployment-name "gpt-4o" \
  --model-name "gpt-4o" \
  --model-version "2024-11-20" \
  --model-format "OpenAI" \
  --sku-capacity 80 \
  --sku-name "GlobalStandard"

# Deploy GPT-4o-mini
az cognitiveservices account deployment create \
  --name "policybot-aoai" \
  --resource-group $RESOURCE_GROUP \
  --deployment-name "gpt-4o-mini" \
  --model-name "gpt-4o-mini" \
  --model-version "2024-07-18" \
  --model-format "OpenAI" \
  --sku-capacity 60 \
  --sku-name "GlobalStandard"

# Deploy text-embedding-3-large
az cognitiveservices account deployment create \
  --name "policybot-aoai" \
  --resource-group $RESOURCE_GROUP \
  --deployment-name "text-embedding-3-large" \
  --model-name "text-embedding-3-large" \
  --model-version "1" \
  --model-format "OpenAI" \
  --sku-capacity 120 \
  --sku-name "Standard"
```

---

## Step 3: Provision Azure AI Search

```bash
az search service create \
  --name "policybot-search" \
  --resource-group $RESOURCE_GROUP \
  --sku "standard" \
  --location $LOCATION \
  --partition-count 1 \
  --replica-count 2

# Enable semantic ranker (included in S1+)
az search service update \
  --name "policybot-search" \
  --resource-group $RESOURCE_GROUP \
  --semantic-search "standard"
```

---

## Step 4: Provision Supporting Services

```bash
# Cosmos DB (conversation history)
az cosmosdb create \
  --name "policybot-cosmos" \
  --resource-group $RESOURCE_GROUP \
  --locations regionName=$LOCATION failoverPriority=0 isZoneRedundant=False \
  --default-consistency-level "Session"

az cosmosdb sql database create \
  --account-name "policybot-cosmos" \
  --resource-group $RESOURCE_GROUP \
  --name "policybotdb"

az cosmosdb sql container create \
  --account-name "policybot-cosmos" \
  --resource-group $RESOURCE_GROUP \
  --database-name "policybotdb" \
  --name "escalations" \
  --partition-key-path "/session_id" \
  --throughput 400

# Key Vault
az keyvault create \
  --name "policybot-kv" \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --enable-rbac-authorization true

# App Insights
az monitor app-insights component create \
  --app "policybot-insights" \
  --location $LOCATION \
  --resource-group $RESOURCE_GROUP \
  --kind "web"
```

---

## Step 5: Configure Managed Identity and RBAC

```bash
# Create Managed Identity for the bot application
az identity create \
  --name "policybot-identity" \
  --resource-group $RESOURCE_GROUP

IDENTITY_PRINCIPAL=$(az identity show \
  --name "policybot-identity" \
  --resource-group $RESOURCE_GROUP \
  --query principalId -o tsv)

# Assign roles
SEARCH_ID=$(az search service show \
  --name "policybot-search" \
  --resource-group $RESOURCE_GROUP \
  --query id -o tsv)

AOAI_ID=$(az cognitiveservices account show \
  --name "policybot-aoai" \
  --resource-group $RESOURCE_GROUP \
  --query id -o tsv)

KV_ID=$(az keyvault show \
  --name "policybot-kv" \
  --resource-group $RESOURCE_GROUP \
  --query id -o tsv)

# Cognitive Services User → AOAI
az role assignment create \
  --assignee $IDENTITY_PRINCIPAL \
  --role "Cognitive Services OpenAI User" \
  --scope $AOAI_ID

# Search Index Data Reader → AI Search
az role assignment create \
  --assignee $IDENTITY_PRINCIPAL \
  --role "Search Index Data Reader" \
  --scope $SEARCH_ID

# Key Vault Secrets User → Key Vault
az role assignment create \
  --assignee $IDENTITY_PRINCIPAL \
  --role "Key Vault Secrets User" \
  --scope $KV_ID
```

---

## Step 6: Deploy the Agent Application

```bash
# Clone the Policy Bot repository
git clone https://github.com/ricardo-msft-SE/flhack-ahca
cd flhack-ahca

# Create Python virtual environment
python -m venv .venv
.venv\Scripts\activate  # Windows
# source .venv/bin/activate  # Linux/macOS

# Install dependencies
pip install -r requirements.txt

# Configure environment variables
cp .env.example .env
# Edit .env with your resource endpoints and IDs

# Deploy agents to Foundry
python scripts/deploy_agents.py \
  --project-endpoint "https://$PROJECT_NAME.api.azureml.ms" \
  --resource-group $RESOURCE_GROUP

# Run integration tests
pytest tests/integration/ -v
```

---

## Step 7: Deploy Azure Bot Service

```bash
# Register the bot
az bot create \
  --name "policybot" \
  --resource-group $RESOURCE_GROUP \
  --kind "registration" \
  --endpoint "https://policybot-app.azurewebsites.net/api/messages" \
  --msa-app-type "UserAssignedMSI" \
  --msa-app-msi-resource-id "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.ManagedIdentity/userAssignedIdentities/policybot-identity"

# Enable Teams channel
az bot teams create \
  --name "policybot" \
  --resource-group $RESOURCE_GROUP
```

---

## Step 8: Deploy Web Portal

```bash
cd web-portal

# Install dependencies
npm install

# Build production bundle
npm run build

# Deploy to Azure Static Web Apps
az staticwebapp create \
  --name "policybot-portal" \
  --resource-group $RESOURCE_GROUP \
  --source "." \
  --location "eastus2" \
  --branch "main" \
  --login-with-github
```

---

## Step 9: Publish Teams Bot

1. Navigate to the [Teams Developer Portal](https://dev.teams.microsoft.com).
2. Import the `teams-manifest/` folder from the repository.
3. Update `manifest.json` with your Bot ID and endpoint.
4. Under **Distribute**, select **Submit to org**.
5. Have your Teams Admin approve the app in the [Teams Admin Center](https://admin.teams.microsoft.com).

---

## Step 10: Configure Foundry IQ Knowledge Index

```bash
# Run the initial document ingestion
python scripts/setup_knowledge_index.py \
  --project-endpoint "https://$PROJECT_NAME.api.azureml.ms" \
  --search-endpoint "https://policybot-search.search.windows.net" \
  --blob-container "policy-docs" \
  --sharepoint-site "https://[org].sharepoint.com/sites/PolicyLibrary"

# Verify index document count
echo "Checking index document count..."
python scripts/verify_index.py
```

---

## Environment Variables Reference

| Variable | Source | Description |
|---|---|---|
| `AZURE_FOUNDRY_ENDPOINT` | Foundry Project | Project endpoint URL |
| `AZURE_OPENAI_ENDPOINT` | Azure OpenAI | Service endpoint |
| `AZURE_SEARCH_ENDPOINT` | Azure AI Search | Service endpoint |
| `AZURE_SEARCH_INDEX` | Config | Index name (`policy-index`) |
| `COSMOS_DB_ENDPOINT` | Cosmos DB | Account endpoint |
| `APPINSIGHTS_CONNECTION_STRING` | App Insights | Telemetry connection |
| `CONFIDENCE_THRESHOLD` | App Config | Default: `0.55` |
| `AUTO_ANSWER_THRESHOLD` | App Config | Default: `0.75` |
| `TEAMS_ESCALATION_WEBHOOK` | Key Vault | Teams incoming webhook URL |

{: .important }
> Never commit `.env` files to source control. All secrets must be stored in Azure Key Vault and retrieved via Managed Identity at runtime.

---

## Post-Deployment Validation Checklist

- [ ] Knowledge index document count > 0
- [ ] Test query returns grounded results with citations
- [ ] Confidence score computed and returned in response
- [ ] Low-confidence query triggers Teams adaptive card
- [ ] Human reviewer can submit reviewed answer
- [ ] Reviewed answer delivered to user via bot
- [ ] Application Insights shows telemetry
- [ ] Bot responds in Teams (test with personal chat)
- [ ] Web portal loads and accepts chat input
- [ ] All RBAC role assignments verified (`az role assignment list`)
