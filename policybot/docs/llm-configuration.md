---
layout: default
title: LLM Configuration
nav_order: 7
description: "Azure OpenAI model selection, system prompt design, grounding configuration, Bing web search setup, and inference parameter tuning for Policy Bot."
permalink: /docs/llm-configuration/
---

# LLM Configuration
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Model Selection

Policy Bot uses two Azure OpenAI models:

| Model | Role | Reason |
|---|---|---|
| **GPT-4o** | Primary generation model for all agents | 128k context window; strong instruction-following; function/tool calling; vision support for future policy document image processing |
| **GPT-4o-mini** | Confidence Evaluator binary checks; Escalation Manager routing | Cost-efficient for high-frequency, constrained tasks |
| **text-embedding-3-large** | Document and query embedding | 3,072-dimension embeddings deliver best-in-class semantic similarity for Azure AI Search hybrid retrieval |

### Azure OpenAI Deployment Configuration

```bash
# Provision via Azure CLI
az cognitiveservices account create \
  --name "policybot-aoai" \
  --resource-group "rg-policybot-prod" \
  --kind "OpenAI" \
  --sku "S0" \
  --location "eastus2"

# GPT-4o deployment
az cognitiveservices account deployment create \
  --name "policybot-aoai" \
  --resource-group "rg-policybot-prod" \
  --deployment-name "gpt-4o" \
  --model-name "gpt-4o" \
  --model-version "2024-11-20" \
  --model-format "OpenAI" \
  --sku-capacity 80 \
  --sku-name "GlobalStandard"

# Embedding model deployment
az cognitiveservices account deployment create \
  --name "policybot-aoai" \
  --resource-group "rg-policybot-prod" \
  --deployment-name "text-embedding-3-large" \
  --model-name "text-embedding-3-large" \
  --model-version "1" \
  --model-format "OpenAI" \
  --sku-capacity 120 \
  --sku-name "Standard"
```

{: .note }
> For production workloads with predictable query volume, consider **Provisioned Throughput Units (PTU)** for GPT-4o to eliminate throttling. PTU pricing is more cost-effective above ~40 TPM sustained load.

---

## Inference Parameters

Parameters are configured **per agent** and stored in Azure AI Foundry's agent configuration:

| Parameter | Orchestrator | Internal Policy | Web Search | Fee Schedule | Confidence Evaluator |
|---|---|---|---|---|---|
| `temperature` | 0.0 | 0.1 | 0.1 | 0.0 | 0.0 |
| `top_p` | 1.0 | 0.95 | 0.95 | 1.0 | 1.0 |
| `max_tokens` | 1,024 | 2,048 | 2,048 | 512 | 512 |
| `frequency_penalty` | 0.0 | 0.3 | 0.3 | 0.0 | 0.0 |
| `presence_penalty` | 0.0 | 0.2 | 0.2 | 0.0 | 0.0 |

**Parameter rationale:**

- **temperature = 0.0** for agents that must be deterministic (routing, fee lookups, evaluation).
- **temperature = 0.1** for generation agents to allow slight variability in phrasing without sacrificing accuracy.
- **frequency_penalty / presence_penalty** on generation agents reduce repetitive phrasing common in policy language.

---

## System Prompt Design

### Orchestrator System Prompt

```
You are the Policy Bot Orchestrator for [Organisation].
Your role is to:
1. Understand the user's policy question and classify its primary topic.
2. Decompose the question into sub-queries if it has multiple parts.
3. Route sub-queries to the appropriate specialist agents.
4. Assemble the specialist agents' responses into a single, coherent answer.
5. Present the answer with source citations and confidence score.

CONSTRAINTS:
- Do NOT make up policy information. Only synthesise from agent responses.
- If specialist agents return no relevant information, state that clearly.
- Always include at least one source citation.
- Format your final answer as a JSON tool call: assemble_response(answer, sources, confidence).

TOPIC CLASSIFICATION:
- HR / Benefits
- Compliance / Regulatory
- Fee Schedule
- Operations / Procedures
- General Information
```

### Internal Policy Agent System Prompt

```
You are an internal policy retrieval expert for [Organisation].
You answer questions EXCLUSIVELY from the document excerpts provided to you.

RULES (strictly enforced):
1. If an answer is NOT present in the provided excerpts, respond:
   "I could not find this information in the internal policy library."
2. Always cite the document name, section number, and page number.
3. Use the EXACT language from the policy document where possible.
   Do not paraphrase in ways that alter meaning.
4. Do NOT extrapolate, infer, or apply your own judgment.
5. If multiple policy documents contain conflicting information, present BOTH
   and flag the conflict: "Note: conflicting policy versions found."

OUTPUT FORMAT:
{
  "answer": "...",
  "citations": [{"document": "...", "section": "...", "excerpt": "..."}],
  "conflict_detected": false
}
```

### Confidence Evaluator System Prompt

```
You are a quality assurance evaluator for an AI policy assistant.

Given a user QUESTION, an AI-generated ANSWER, and the SOURCE DOCUMENTS used,
you will evaluate the answer on three dimensions:

1. FAITHFULNESS (0.0–1.0): Is every claim in the answer directly supported by the sources?
   Count all factual claims in the answer. Score = supported claims / total claims.

2. RELEVANCE (provided externally as reranker scores — do not re-score).

3. COMPLETENESS (0.0–1.0): Does the answer address all parts of the question?
   List each distinct question component. Score = addressed / total components.

Return ONLY the following JSON — no explanatory text:
{
  "faithfulness": float,
  "completeness": float,
  "supported_claims": int,
  "total_claims": int,
  "addressed_components": int,
  "total_components": int,
  "reasoning": "one-sentence explanation of the lowest dimension score"
}
```

---

## Response Grounding and Hallucination Prevention

Policy Bot applies **multiple layers** of hallucination prevention beyond temperature tuning:

### Layer 1: Context Window Stuffing

The generation prompt includes the full retrieved document chunks. The model is instructed to answer **only** from these chunks:

```python
def build_generation_prompt(query: str, chunks: list[dict]) -> str:
    chunk_text = "\n\n---\n\n".join(
        f"[Source: {c['title']}, {c['section']}]\n{c['text']}" 
        for c in chunks
    )
    return f"""
DOCUMENT EXCERPTS:
{chunk_text}

USER QUESTION: {query}

INSTRUCTIONS: Answer the question using ONLY the document excerpts above.
If the answer is not in the excerpts, say so explicitly.
Cite the document title and section for every claim.
"""
```

### Layer 2: Citation Verification

After generation, a lightweight post-process step verifies that each cited source actually exists in `chunks`:

```python
def verify_citations(response: dict, chunks: list[dict]) -> dict:
    retrieved_titles = {c["title"] for c in chunks}
    verified_citations = []
    for citation in response.get("citations", []):
        if citation["document"] in retrieved_titles:
            verified_citations.append(citation)
        else:
            # Remove hallucinated citation; flag for monitoring
            log_hallucinated_citation(citation, response)
    response["citations"] = verified_citations
    return response
```

### Layer 3: Confidence Gate

Even after layers 1 and 2, the Confidence Evaluator can downgrade an answer that shows signs of unsupported claims before it reaches the user. See [Confidence Scoring](../confidence-scoring/).

---

## Bing Web Search Configuration

### Resource Provisioning

```bash
az cognitiveservices account create \
  --name "policybot-bing" \
  --resource-group "rg-policybot-prod" \
  --kind "Bing.Search.v7" \
  --sku "S2" \
  --location "global"
```

### Search Query Optimisation

The Web Search Agent reformulates raw user queries into effective Bing search strings using a dedicated sub-call:

```python
SEARCH_REFORMULATION_PROMPT = """
Convert the following policy question into 1-3 specific web search queries
optimised for finding official regulatory or government sources.

Prefer: site:.gov, site:.org, official agency websites
Avoid: forums, opinion pieces, commercial sites

Question: {question}

Return JSON: {"queries": ["query1", "query2"]}
"""
```

### Domain Filtering

```python
TRUSTED_DOMAINS = [
    "site:cms.gov",
    "site:ahca.myflorida.com",
    "site:irs.gov",
    "site:doh.state.fl.us",
    "site:medicaid.gov",
]

def build_bing_query(base_query: str, domain_filter: str = None) -> str:
    if domain_filter:
        return f"{base_query} ({domain_filter})"
    # Auto-add gov/org filter for regulatory queries
    return f"{base_query} (site:.gov OR site:.org)"
```

### Bing API Call Pattern

```python
import httpx

async def bing_search(query: str, count: int = 5) -> list[dict]:
    headers = {
        "Ocp-Apim-Subscription-Key": os.environ["BING_SEARCH_API_KEY"]
    }
    params = {
        "q": query,
        "count": count,
        "mkt": "en-US",
        "responseFilter": "Webpages",
        "freshness": "Month",  # Prefer recent results for regulatory content
    }
    async with httpx.AsyncClient() as client:
        resp = await client.get(
            "https://api.bing.microsoft.com/v7.0/search",
            headers=headers,
            params=params,
            timeout=10.0,
        )
        resp.raise_for_status()
        return resp.json().get("webPages", {}).get("value", [])
```

{: .important }
> **Security note:** The Bing API key is never embedded in code. It is retrieved from Azure Key Vault at startup using Managed Identity: `key_vault_client.get_secret("bing-search-api-key")`.

---

## Token Budget Management

Each agent enforces a token budget to control costs and prevent context overflow:

```python
from tiktoken import encoding_for_model

AGENT_TOKEN_BUDGETS = {
    "orchestrator": {"context": 16_000, "completion": 1_024},
    "internal_policy": {"context": 32_000, "completion": 2_048},
    "web_search": {"context": 16_000, "completion": 2_048},
    "fee_schedule": {"context": 4_000, "completion": 512},
    "confidence_evaluator": {"context": 8_000, "completion": 512},
}

def trim_chunks_to_budget(
    chunks: list[dict], model: str, context_budget: int
) -> list[dict]:
    enc = encoding_for_model(model)
    total = 0
    trimmed = []
    for chunk in chunks:
        tokens = len(enc.encode(chunk["text"]))
        if total + tokens > context_budget:
            break
        trimmed.append(chunk)
        total += tokens
    return trimmed
```

---

## Cost Estimation

| Component | Volume | Est. Cost/Month |
|---|---|---|
| GPT-4o input (generation) | 500K tokens | ~$1.25 |
| GPT-4o output (generation) | 200K tokens | ~$2.00 |
| text-embedding-3-large | 2M tokens | ~$0.13 |
| Bing Search API (S2, 1K calls) | 1,000 calls | ~$7.00 |
| GPT-4o-mini (evaluation) | 300K tokens | ~$0.05 |
| **Estimated total** | | **~$10–15/month** (at 500 queries/day) |

{: .note }
> Estimates based on pay-as-you-go pricing as of March 2025. Use PTU for sustained high-volume workloads.
