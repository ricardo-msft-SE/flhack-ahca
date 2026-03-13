---
layout: default
title: Operational Costs
nav_order: 10
description: "Estimated Azure runtime costs for Policy Bot across two deployment scenarios: 50-user internal pilot and 1,000-user public-facing production."
permalink: /docs/costs/
---

# Operational Costs
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .warning }
> All figures are **estimates** based on Azure public pricing as of Q1 2026 and representative usage assumptions. Actual costs vary with query complexity, index size, retrieval depth, and regional pricing. Always validate against the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) before budgeting.

---

## Usage Assumptions

The two scenarios differ in user volume and expected query patterns. All other architectural assumptions are shared.

### Shared assumptions

| Parameter | Value |
|---|---|
| Avg. query tokens — user input | 80 tokens |
| Avg. Orchestrator context (input) | 800 tokens |
| Avg. Internal Policy Agent context (retrieved chunks + prompt) | 4,000 tokens |
| Avg. generation output per main agent | 800 tokens |
| Fraction of queries invoking Web Search Agent | 60 % |
| Fraction of queries invoking Fee Schedule Agent | 20 % |
| Fraction of queries invoking Legal & Regulatory Agent (`o3`) | 15 % |
| Fraction of queries invoking Document Vision Agent (`gpt-4o`) | 10 % |
| Avg. Confidence Evaluator context (input + output) | 1,700 tokens |
| Avg. embedding tokens per query | 500 tokens |
| Avg. Bing Search API calls per query (Web Search agent) | 3 calls |

### Scenario 1 — Internal pilot (50 users/month)

| Parameter | Value |
|---|---|
| Active users/month | 50 |
| Avg. queries/user/month | 10 |
| **Total queries/month** | **500** |

### Scenario 2 — Public-facing production (1,000 users/month)

| Parameter | Value |
|---|---|
| Active users/month | 1,000 |
| Avg. queries/user/month | 20 |
| **Total queries/month** | **20,000** |

---

## Token Volume Breakdown

Per-query weighted token volumes (based on invocation rates above):

| Model | Per-query input tokens | Per-query output tokens |
|---|---|---|
| `gpt-4.1` (Orchestrator + Internal Policy + Fee Schedule) | ~5,200 | ~1,160 |
| `gpt-4.1-mini` (Web Search + Confidence Eval + Escalation) | ~3,200 | ~600 |
| `o3` (Legal & Regulatory, 15% invocation) | ~450 | ~150 |
| `gpt-4o` (Document Vision, 10% invocation) | ~200 | ~50 |
| `text-embedding-3-large` | ~500 | — |

Monthly token volumes:

| Model | Scenario 1 (500 q) | Scenario 2 (20,000 q) |
|---|---|---|
| `gpt-4.1` input | 2.60 M | 104 M |
| `gpt-4.1` output | 0.58 M | 23.2 M |
| `gpt-4.1-mini` input | 1.60 M | 64 M |
| `gpt-4.1-mini` output | 0.30 M | 12 M |
| `o3` input | 0.23 M | 9 M |
| `o3` output | 0.08 M | 3 M |
| `gpt-4o` input | 0.10 M | 4 M |
| `gpt-4o` output | 0.03 M | 1 M |
| Embeddings | 0.25 M | 10 M |

---

## Reference Pricing (Q1 2026, Pay-as-you-go)

| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|---|---|---|
| `gpt-4.1` | $2.00 | $8.00 |
| `gpt-4.1-mini` | $0.40 | $1.60 |
| `o3` | $10.00 | $40.00 |
| `gpt-4o` | $2.50 | $10.00 |
| `text-embedding-3-large` | $0.13 | — |

---

## Scenario 1 — Internal Pilot (50 users / ~500 queries per month)

### LLM and embedding costs

| Model | Input cost | Output cost | Subtotal |
|---|---|---|---|
| `gpt-4.1` | 2.60 M × $2.00 = **$5.20** | 0.58 M × $8.00 = **$4.64** | **$9.84** |
| `gpt-4.1-mini` | 1.60 M × $0.40 = **$0.64** | 0.30 M × $1.60 = **$0.48** | **$1.12** |
| `o3` | 0.23 M × $10.00 = **$2.30** | 0.08 M × $40.00 = **$3.00** | **$5.30** |
| `gpt-4o` | 0.10 M × $2.50 = **$0.25** | 0.03 M × $10.00 = **$0.25** | **$0.50** |
| Embeddings | 0.25 M × $0.13 = **$0.03** | — | **$0.03** |
| **LLM total** | | | **~$17/month** |

### Infrastructure costs

| Service | SKU | Est. monthly cost |
|---|---|---|
| Azure AI Search | Standard S1 (1 replica, 1 partition) | $250 |
| Azure App Service | P1v3 (1 instance) | $140 |
| Azure Bot Service | S1 | $12 |
| Bing Search API | S2 (500 q × 60% × 3 calls = 900 calls) | $1 |
| Azure Cosmos DB | Serverless | $5 |
| Azure Blob Storage | LRS Hot (~10 GB docs) | $5 |
| Azure Key Vault | Standard | $1 |
| Azure Monitor / App Insights | Pay-per-use | $5 |
| **Infrastructure total** | | **~$419/month** |

### Scenario 1 total

| Component | Monthly cost |
|---|---|
| LLM + embeddings | ~$17 |
| Infrastructure | ~$419 |
| **Total** | **~$436 / month** |

{: .highlight }
> At $436/month for 50 users, the **effective cost per user is ~$8.70/month** or roughly **$0.87 per query**. The dominant cost driver is Azure AI Search, not LLM inference — reflecting the read-heavy RAG retrieval pattern.

---

## Scenario 2 — Public-Facing Production (1,000 users / ~20,000 queries per month)

### LLM and embedding costs

| Model | Input cost | Output cost | Subtotal |
|---|---|---|---|
| `gpt-4.1` | 104 M × $2.00 = **$208** | 23.2 M × $8.00 = **$186** | **$394** |
| `gpt-4.1-mini` | 64 M × $0.40 = **$26** | 12 M × $1.60 = **$19** | **$45** |
| `o3` | 9 M × $10.00 = **$90** | 3 M × $40.00 = **$120** | **$210** |
| `gpt-4o` | 4 M × $2.50 = **$10** | 1 M × $10.00 = **$10** | **$20** |
| Embeddings | 10 M × $0.13 = **$1** | — | **$1** |
| **LLM total** | | | **~$670/month** |

### Infrastructure costs

| Service | SKU / scaling note | Est. monthly cost |
|---|---|---|
| Azure AI Search | Standard S2 (2 replicas, 2 partitions for HA + throughput) | $1,000 |
| Azure App Service | P2v3 × 2 instances (auto-scale) | $570 |
| Azure API Management | Developer → Standard tier for rate limiting & auth | $100 |
| Azure Front Door | Standard (CDN + WAF for public endpoint) | $55 |
| Azure Bot Service | S1 | $50 |
| Bing Search API | S2 (20,000 × 60% × 3 = 36,000 calls/month) | $180 |
| Azure Cosmos DB | Serverless (20 k sessions × ops) | $80 |
| Azure Blob Storage | LRS Hot (~50 GB + object operations) | $20 |
| Azure Key Vault | Standard | $5 |
| Azure Monitor / App Insights | Pay-per-use (higher log volume) | $50 |
| **Infrastructure total** | | **~$2,110/month** |

### Scenario 2 total

| Component | Monthly cost |
|---|---|
| LLM + embeddings | ~$670 |
| Infrastructure | ~$2,110 |
| **Total** | **~$2,780 / month** |

{: .highlight }
> At $2,780/month for 1,000 users, the **effective cost per user is ~$2.78/month** or roughly **$0.14 per query**. The costs exhibit sub-linear scaling: 40× more queries cost only ~6× more than Scenario 1, because infrastructure fixed costs (Search, App Service) dominate at low volume but amortise across larger query volumes.

---

## Cost Drivers and Optimisation Levers

### Primary cost drivers by scenario

| Rank | Scenario 1 (pilot) | Scenario 2 (production) |
|---|---|---|
| 1st | Azure AI Search (~57%) | Azure AI Search (~36%) |
| 2nd | `o3` Legal Agent (~1%) | `gpt-4.1` inference (~14%) |
| 3rd | App Service (~32%) | App Service (~21%) |
| 4th | `gpt-4.1` inference (~2%) | `o3` Legal Agent (~8%) |

### Levers to reduce costs

| Lever | Savings potential | Trade-off |
|---|---|---|
| **Provisioned Throughput (PTU) for `gpt-4.1`** | 20–40% on `gpt-4.1` at sustained load | Upfront commitment, least-flexible at low volume |
| **Reduce `o3` invocation rate** | High — `o3` output is 5–20× `gpt-4.1-mini` per token | Route only confirmed regulatory queries; tighten Orchestrator classifier |
| **Azure AI Search tier right-sizing** | 30–50% if S1 is sufficient at scale | Monitor query latency under load before downgrading |
| **Result caching** | 10–30% on LLM costs | Effective for repeated FAQ-class queries; reduces freshness for live regulatory updates |
| **Embedding cache** | ~5% | Low complexity; cache by query hash |
| **Reduce Web Search invocation rate** | Small | Tighten Orchestrator classification to invoke only when internal search yields low confidence |

---

## Cost Comparison Summary

| | Scenario 1 (50 users) | Scenario 2 (1,000 users) |
|---|---|---|
| Total / month | **~$436** | **~$2,780** |
| Per user / month | ~$8.70 | ~$2.78 |
| Per query | ~$0.87 | ~$0.14 |
| LLM share | ~4% | ~24% |
| Infrastructure share | ~96% | ~76% |
| Dominant cost | Azure AI Search | Azure AI Search + `gpt-4.1` |
