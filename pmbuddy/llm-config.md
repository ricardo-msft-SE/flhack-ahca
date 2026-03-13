---
layout: default
title: LLM & Web Search
parent: PM Buddy
nav_order: 4
permalink: /pmbuddy/llm-config/
---

# LLM & Web Search Configuration
{: .no_toc }

Configuration guidance for the Azure OpenAI language model deployment and Bing Search grounding used by PM Buddy agents.

---

## Table of Contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Model Selection

PM Buddy uses **GPT-4o** (`gpt-4o-2024-11-20`) for all agents. This model was selected for:

| Criterion | GPT-4o Advantage |
|---|---|
| Reasoning quality | Best-in-class for structured output, planning, and multi-step tasks |
| Context window | 128K tokens — handles long transcripts without chunking penalties |
| Function calling | Parallel function calls; reliable JSON mode output |
| Cost | ~50% cheaper than GPT-4 Turbo with equal or better performance on PM tasks |
| Latency | Streaming responses for real-time Teams bot interaction |

{: .note }
> **Embedding model:** `text-embedding-3-large` (3072 dimensions) is used for Foundry IQ vector indexing. It is not used for inference — only for generating document and query embeddings.

---

## Azure OpenAI Deployment Configuration

### Foundry Deployment Settings

```yaml
# Azure AI Foundry deployment — conceptual configuration
deployment:
  name: gpt-4o-pm
  model: gpt-4o
  model_version: "2024-11-20"
  sku: GlobalStandard        # Routes to lowest-latency datacenter
  capacity: 100              # 100K tokens per minute (TPM)
  api_version: "2024-12-01-preview"
  content_filter:
    hate: medium
    self_harm: medium
    sexual: medium
    violence: medium
    jailbreak: enabled       # Block prompt injection attempts
```

### Recommended Inference Parameters

Different agents use different temperature settings based on how creative versus deterministic their output should be:

| Agent | Temperature | Max Tokens | Top-P | Reason |
|---|---|---|---|---|
| Orchestrator | 0.1 | 1024 | 1.0 | Routing decisions should be deterministic |
| Transcript Processor | 0.0 | 2048 | 1.0 | Extraction must be accurate; no creativity |
| Project Context | 0.1 | 512 | 1.0 | Scoring logic should be consistent |
| Project Manager | 0.4 | 4096 | 0.95 | Plans need some creativity within structure |
| Report Generator | 0.5 | 3072 | 0.95 | Reports should be readable and varied |
| SQL Agent | 0.0 | 512 | 1.0 | SQL generation must be exact |

```python
# Example: Semantic Kernel agent configuration (Python SDK)
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
from semantic_kernel.connectors.ai.open_ai.prompt_execution_settings.azure_chat_prompt_execution_settings import (
    AzureChatPromptExecutionSettings,
)

transcript_processor_settings = AzureChatPromptExecutionSettings(
    temperature=0.0,
    max_tokens=2048,
    top_p=1.0,
    response_format={"type": "json_object"},  # Enforce JSON output
)

project_manager_settings = AzureChatPromptExecutionSettings(
    temperature=0.4,
    max_tokens=4096,
    top_p=0.95,
)
```

---

## System Prompts

Each agent has a dedicated system prompt stored in Azure AI Foundry as a versioned prompt asset. Key principles across all system prompts:

1. **Role clarity** — Begin with a clear persona (`You are a [role]`)
2. **Scope constraint** — Explicitly state what the agent must NOT do
3. **Output format** — Specify exact output structure (JSON schema, Markdown sections)
4. **Grounding instruction** — Tell the agent to rely on provided context; not hallucinate
5. **Safety instruction** — Never include PII in outputs unless explicitly required

### Orchestrator System Prompt (Full)

```
You are the Project Management Buddy orchestrator. Your sole job is to route user requests
to the correct specialist agents and assemble their results.

RULES:
- Never generate project data yourself — always delegate to specialist agents.
- Always verify a project_id exists before calling the Report Generation Agent.
- If intent is ambiguous, ask the user one clarifying question before proceeding.
- Do not expose internal agent names or tool names to the user.
- Summarize errors in plain language; never expose stack traces or SQL errors.

INTENT CLASSIFICATION:
- transcript_ingestion: User submits transcript text or file.
- project_query: User asks about a project's status, plan, or risks.
- report_request: User requests a specific report type.
- general_question: General PM question (answer using Foundry IQ context).

Return all responses in the user's detected language.
```

---

## JSON Mode and Structured Outputs

The Transcript Processing Agent and SQL Agent use **JSON mode** (`response_format: { type: "json_object" }`) to guarantee parseable output. The Project Management Agent uses **structured outputs** with a JSON schema:

```json
{
  "type": "json_schema",
  "json_schema": {
    "name": "ProjectPlan",
    "schema": {
      "type": "object",
      "properties": {
        "project_charter": { "type": "string" },
        "sql_operations": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "operation": { "type": "string", "enum": ["INSERT", "UPDATE"] },
              "table": { "type": "string" },
              "params": { "type": "object" }
            },
            "required": ["operation", "table", "params"]
          }
        },
        "change_summary": { "type": "string" }
      },
      "required": ["project_charter", "sql_operations", "change_summary"]
    },
    "strict": true
  }
}
```

---

## Web Search — Bing Search API Grounding

### Why Web Search?

The Project Management Agent can optionally ground its plan generation with live web content. This is useful for:

- **Industry benchmarks** — "How long does a typical ERP migration take?"
- **Technology evaluation** — "What are current Azure SQL pricing tiers?"
- **Risk research** — "What are common risks in cloud migrations?"

Without web grounding, the model relies solely on its training data, which may be stale.

### Bing Search Configuration

```yaml
# Bing Search API — Azure AI Foundry connection
connection:
  name: bing-search-grounding
  type: BingGrounding
  api_key: <stored-in-key-vault>
  endpoint: https://api.bing.microsoft.com/v7.0/search
  safe_search: Moderate
  market: en-US
  count: 5           # Web results per query
  freshness: Month   # Prefer results from the last month
```

### Grounding Pattern

Web search is invoked as a Semantic Kernel plugin, not as a direct tool call. The agent:

1. Generates a focused search query from its current reasoning context
2. Calls `bing_search(query)` — returns top-5 result snippets
3. Adds snippets as grounding context in the next reasoning step
4. Cites sources in the output (URL and title)

```python
# Semantic Kernel Bing grounding plugin (conceptual)
from semantic_kernel.connectors.search_services.bing import BingConnector
from semantic_kernel.plugins.web.bing_plugin import BingPlugin

bing = BingConnector(api_key=settings.bing_api_key)
kernel.add_plugin(BingPlugin(bing), plugin_name="web_search")

# Agent can then call:
# result = await kernel.invoke("web_search", "BingPlugin", query="ERP migration timeline benchmarks 2025")
```

### When NOT to Use Web Search

The Orchestrator controls whether web search is enabled per request:

| Condition | Web Search |
|---|---|
| Internal project query (status, tasks) | Disabled — all data in SQL / Foundry IQ |
| New project plan requiring benchmarks | Enabled |
| Report generation from existing data | Disabled |
| General PM best practice question | Enabled |

{: .warning }
> Do not pass transcript content or project data to Bing Search. Only anonymized or generic search queries should reach external search APIs. The Orchestrator enforces this via a query sanitization step before invoking the Bing plugin.

---

## Token Budget and Cost Management

```python
# Per-run token budget enforced by Orchestrator
MAX_TOKENS_PER_RUN = {
    "transcript_ingestion": 8_000,
    "project_create": 12_000,
    "project_update": 6_000,
    "report_generation": 8_000,
}
```

| Control | Mechanism |
|---|---|
| Hard token ceiling | `max_tokens` per agent call |
| Per-run budget | Orchestrator sums consumed tokens; halts on overrun |
| Model tier | GlobalStandard SKU with PTU (Provisioned Throughput Units) for predictable cost |
| Caching | Azure API Management response cache for identical transcript + project combos |
| Monitoring | Azure Monitor workbook tracking daily token spend by agent |

---

## Content Safety and Responsible AI

PM Buddy's Azure OpenAI deployment includes Azure AI Content Safety filters:

| Category | Threshold | Action on Trigger |
|---|---|---|
| Hate/Discrimination | Medium | Block + log |
| Self-harm | Medium | Block + log |
| Sexual content | Medium | Block + log |
| Violence | Medium | Block + log |
| Jailbreak / Prompt injection | All severities | Block + alert |
| PII in output | Custom classifier | Redact before delivery |

Detected violations are logged to Azure Monitor and trigger an alert to the PM Buddy operations team. No transcript content that triggers a safety filter is persisted to the database.
