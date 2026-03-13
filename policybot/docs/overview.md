---
layout: default
title: Overview
nav_order: 2
description: "Problem statement and solution overview for the Policy Bot system."
permalink: /docs/overview/
---

# Overview
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Problem Statement

Organisations managing complex policy environments — such as healthcare agency compliance, HR handbooks, operational procedures, and regulatory fee schedules — face a recurring challenge: **staff spend disproportionate time searching for policy answers through siloed documents, intranet wikis, and subject-matter experts**.

This creates:

- **Inconsistent answers** — different staff relay different interpretations of the same policy.
- **High SME burden** — policy experts are pulled into routine lookups rather than high-value work.
- **Auditability gaps** — verbal or ad-hoc answers lack traceable source citations.
- **Slow response times** — queries routed through email chains or ticketing introduce days of latency.

---

## Solution: Policy Bot

Policy Bot is a conversational AI assistant deployed to **Microsoft Teams** and a **Web Portal**. It answers policy questions in natural language, cites its sources, and quantifies its confidence in every answer.

When a question is too complex or the confidence score falls below a configurable threshold, the system **automatically escalates to a human reviewer** via a structured Teams adaptive card workflow, ensuring no question goes unanswered.

![Policy Bot flow diagram]({{ '/assets/images/arch10.png' | relative_url }})

_Figure: Policy Bot solution flow (arch10)._ 

---

## How It Aligns to the Flow Diagram

The attached architecture sketch maps directly to the following system capabilities:

| Diagram Element | System Implementation |
|---|---|
| **Python** | Agent Framework agents are authored in Python using the Foundry Agent SDK. |
| **Agent (central node)** | Orchestrator Agent — the primary AI decision-maker. |
| **Policies → WEB** | Web Search Agent calls Bing Search API for public regulatory content. |
| **Policies → INTERNAL** | Internal Policy Agent queries the Foundry IQ knowledge index (Azure AI Search). |
| **Q ↕ A** | The user-facing conversational turn-taking loop. |
| **CITE SOURCES / CONFIDENCE LEVELS** | Every response includes source citations and a 0–1 confidence score. |
| **IF COMPLEX → ALERT HUMAN** | Escalation Manager triggers when confidence < threshold. |
| **WORKFLOW** | Agent Framework workflow graph governs the multi-step agent pipeline. |
| **CHAT BOT** | Azure Bot Service host connected to the Agent orchestration layer. |
| **TEAMS (CHAT)** | Microsoft Teams channel integration via the Bot Framework Teams adapter. |
| **WEB PORTAL** | React SPA with Direct Line REST API to Azure Bot Service. |
| **1 AGENT OR MANY** | Configurable: single orchestrator for simple queries, full multi-agent for complex. |
| **FEE SCHEDULES** | Fee Schedule Agent that analyzes complex tabular data from spreadsheets and policy documents using LLM-powered table understanding. |
| **AI SEARCH SERVICE / LLM** | Azure AI Search (retrieval) + Azure OpenAI GPT-4o (generation). |

---

## Key Stakeholders

| Role | Interaction |
|---|---|
| **Policy Staff / End Users** | Ask questions via Teams or Web Portal, receive cited answers. |
| **Policy SMEs / Reviewers** | Receive escalated questions, provide authoritative answers via Teams card. |
| **IT / Platform Team** | Maintain the Azure AI Foundry deployment, index freshness, and model endpoints. |
| **Compliance / Audit Team** | Access the conversation log and source citation audit trail. |

---

## Design Principles

{: .highlight }
> Policy Bot is designed around **trust through transparency**: every answer must be explainable, cited, and confidence-quantified. If the system cannot be confident, it says so — and coordinates expert review.

1. **Grounded responses only** — the LLM is constrained to answer from retrieved sources. Hallucination guards are applied via system prompt instructions and temperature tuning.
2. **Fail-safe escalation** — any failure mode (low confidence, tool error, ambiguous query) routes to human review rather than producing an unchecked answer.
3. **Source citation required** — every response includes document title, section reference, and URL or SharePoint path.
4. **Separation of concerns** — each agent has a single, well-scoped responsibility. This simplifies testing, auditing, and replacement.
5. **Channel-agnostic core** — the multi-agent orchestration logic is independent of the chat surface (Teams, Web Portal, or future channels).
