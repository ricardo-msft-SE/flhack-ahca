---
layout: home
title: Home
nav_order: 1
description: "Policy Bot is a multi-agent AI system built on Azure AI Foundry that answers internal and regulatory policy questions with cited sources and confidence scoring."
permalink: /
---

# Policy Bot — Architectural Design Guide
{: .fs-9 }

A multi-agent, enterprise-grade policy Q&A system built on **Azure AI Foundry**, **Azure OpenAI**, and the **Microsoft Agent Framework** — with built-in confidence scoring, human-in-the-loop escalation, and Foundry IQ-powered knowledge management.
{: .fs-6 .fw-300 }

[Get Started — Overview](docs/overview/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View Architecture](docs/architecture/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What Is Policy Bot?

Policy Bot is a conversational AI assistant that answers questions about organisational policies — including internal HR, operations, and compliance documents, as well as publicly accessible regulatory and fee schedule information.

The system is surfaced through **Microsoft Teams** and a **Web Portal**. It uses a **multi-agent orchestration architecture** to decompose complex queries, retrieve authoritative information from multiple sources, evaluate the confidence of its answers, and — when confidence falls below an acceptable threshold — automatically brings a **human expert** into the conversation.

---

## Key Capabilities

| Capability | Details |
|---|---|
| **Multi-source policy retrieval** | Searches both internal SharePoint/Blob documents and live web sources. |
| **Confidence scoring** | Each answer is accompanied by a numeric confidence score and source citations. |
| **Human-in-the-loop escalation** | Low-confidence or high-complexity queries are routed to a human reviewer via Teams or email. |
| **Fee schedule lookups** | A dedicated agent resolves fee-related queries against structured data. |
| **Foundry IQ grounding** | Knowledge index built and maintained by Azure AI Foundry IQ for precision recall. |
| **Dual chat surfaces** | Microsoft Teams adaptive card bot and a standalone React web portal. |

---

## Documentation Sections

<div class="d-flex flex-wrap gap-3 mt-3">

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>🏗️ <a href="docs/architecture/">Architecture</a></h3>
    <p>End-to-end component diagram, Azure services, and data flow walkthrough.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>🤖 <a href="docs/agents/">Multi-Agent Design</a></h3>
    <p>Agent roles, the orchestration pattern, and Agent Framework SDK usage.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>📊 <a href="docs/confidence-scoring/">Confidence Scoring</a></h3>
    <p>How confidence is calculated, thresholds, and branching logic.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>🧑‍💼 <a href="docs/human-in-the-loop/">Human-in-the-Loop</a></h3>
    <p>Escalation triggers, Teams adaptive card workflow, and resolution tracking.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>⚙️ <a href="docs/llm-configuration/">LLM Configuration</a></h3>
    <p>Azure OpenAI model selection, system prompts, grounding, and web search.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>💡 <a href="docs/foundry-iq/">Foundry IQ</a></h3>
    <p>Knowledge index management, document ingestion, and hybrid search configuration.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>🚀 <a href="docs/deployment/">Deployment</a></h3>
    <p>Prerequisites, infrastructure setup, and step-by-step deployment guide.</p>
  </div>

  <div class="p-4" style="border:1px solid #e1e4e8;border-radius:6px;flex:1 1 250px">
    <h3>📖 <a href="docs/glossary/">Glossary</a></h3>
    <p>Definitions of all terms and acronyms used in this guide.</p>
  </div>

</div>

---

## Technology Stack at a Glance

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend / Channels                  │
│         Microsoft Teams Bot   │   React Web Portal      │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                  Azure Bot Service                       │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│              Azure AI Foundry — Agent Plane              │
│  ┌──────────────┐  ┌─────────────┐  ┌───────────────┐  │
│  │ Orchestrator │  │   Policy    │  │  Web Search   │  │
│  │    Agent     │  │    Agent    │  │     Agent     │  │
│  └──────┬───────┘  └──────┬──────┘  └───────┬───────┘  │
│         │                 │                  │          │
│  ┌──────▼───────┐  ┌──────▼──────┐           │          │
│  │   Fee Sched  │  │ Confidence  │◄──────────┘          │
│  │    Agent     │  │  Evaluator  │                      │
│  └──────────────┘  └──────┬──────┘                      │
└─────────────────────────  │ ──────────────────────────--┘
                     ┌──────▼──────┐
                     │  Escalation │
                     │   Manager   │
                     └──────┬──────┘
                            │  (low confidence)
                     ┌──────▼──────┐
                     │   Human     │
                     │  Reviewer   │
                     └─────────────┘
```

---

## Getting Started

> **Prerequisites:** An Azure subscription with access to Azure AI Foundry, Azure OpenAI, and Azure AI Search. See [Deployment](docs/deployment/) for full setup.
{: .note }

1. Review the [Architecture](docs/architecture/) to understand all components.
2. Follow [LLM Configuration](docs/llm-configuration/) to provision your Azure OpenAI deployment.
3. Set up your [Foundry IQ](docs/foundry-iq/) knowledge index.
4. Configure the [Multi-Agent](docs/agents/) system in the Foundry Agent SDK.
5. Wire up [Human-in-the-Loop](docs/human-in-the-loop/) escalation to Teams.
6. Use the [Deployment](docs/deployment/) guide to ship to production.
