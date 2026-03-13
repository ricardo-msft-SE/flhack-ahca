---
layout: default
title: PM Buddy
nav_order: 2
has_children: true
permalink: /pmbuddy/
---

# Project Management Buddy
{: .no_toc }

An AI-powered assistant that transforms meeting transcripts into structured project records, up-to-date plans, and actionable reports — all built on **Microsoft Azure AI Foundry** and **Azure AI Services**.

---

## Table of Contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## What Is PM Buddy?

PM Buddy automates the manual overhead of project management by listening to meeting transcripts and intelligently routing their content into your project database. It answers three core questions in real time:

1. **Is there an existing project this transcript belongs to?**
2. **If yes — what should be updated?**
3. **If no — what does a new project record look like?**

After resolving that routing decision, PM Buddy produces reports, plans, and artifacts that stakeholders can act on immediately.

---

## Core Capabilities

| Capability | Description |
|---|---|
| Transcript Ingestion | Accepts raw meeting transcripts from Teams, Zoom, or file upload |
| Semantic Project Matching | Uses Foundry IQ (RAG) to find semantically related projects |
| Structured DB Operations | Creates or updates project records in Azure SQL via a dedicated SQL agent |
| Plan Generation | Drafts milestones, tasks, and timelines using GPT-4o |
| Report & Artifact Creation | Generates status reports, RACI charts, and meeting summaries |
| Web-Grounded Research | Integrates Bing Search to enrich plans with current industry context |

---

## Technology Stack at a Glance

| Layer | Technology |
|---|---|
| LLM | Azure OpenAI — GPT-4o |
| Agent Framework | Azure AI Foundry Agent Service + Semantic Kernel |
| Knowledge / RAG | Azure AI Foundry IQ (Azure AI Search — vector + semantic) |
| Structured Data | Azure SQL Database |
| Document Storage | Azure Blob Storage |
| Web Search | Bing Search API (grounding) |
| Transcript Processing | Azure AI Document Intelligence + Speech-to-Text |
| Orchestration Runtime | Azure Functions / Azure Container Apps |
| Security | Microsoft Entra ID, Azure Key Vault |

---

## How It Works — One-Sentence Summary

> *Meeting transcripts are processed by AI agents that check a project database for related context, either creating a new project or enriching an existing one, and then generate reports and artifacts for the team.*

Explore the sections in order for a detailed breakdown:

- **[Data Flow]({{ site.baseurl }}/pmbuddy/data-flow/)** — step-by-step walkthrough of the flow diagram
- **[System Architecture]({{ site.baseurl }}/pmbuddy/architecture/)** — component-level design
- **[Multi-Agent Design]({{ site.baseurl }}/pmbuddy/agents/)** — agent roles and communication
- **[LLM & Web Search]({{ site.baseurl }}/pmbuddy/llm-config/)** — model configuration and grounding
- **[Foundry IQ]({{ site.baseurl }}/pmbuddy/foundry-iq/)** — RAG knowledge base design
- **[Glossary]({{ site.baseurl }}/pmbuddy/glossary/)** — key terms and definitions
