---
layout: home
title: Home
nav_order: 1
description: "Project Management Buddy turns meeting transcripts into living project plans, artifacts, and updates using Azure AI Foundry and Agent Framework."
permalink: /
---

# Project Management Buddy
{: .fs-9 }

A transcript-first project orchestration assistant built on **Azure AI Foundry**, **Foundry IQ**, **Agent Framework**, and **Azure AI Services**.
{: .fs-6 .fw-300 }

[Open Architecture](docs/architecture/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Open Agent Design](docs/agents/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## Why this app

Project Management Buddy ingests meeting transcripts, resolves whether a meeting maps to an existing project context, and then either:

- creates a new project record and starter plan, or
- enriches an existing project with fresh reports, artifacts, and action items.

This aligns directly to the desired flow: **meeting transcripts → project DB → existing related info decision → create new or add to existing artifacts**.

---

## Quick navigation

- [Overview](docs/overview/)
- [Architecture](docs/architecture/)
- [Multi-Agent Design](docs/agents/)
- [Foundry IQ](docs/foundry-iq/)
- [LLM + Web Search Configuration](docs/llm-configuration/)
- [Deployment](docs/deployment/)
- [Glossary](docs/glossary/)

---

## UX enhancements in this doc set

- Automatic page-level table of contents on core pages.
- Cross-linked glossary terms and acronym index.
- Decision tables for SQL vs Foundry IQ responsibilities.
- Mermaid diagrams for architecture and control flow.
