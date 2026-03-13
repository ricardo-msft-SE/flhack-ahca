---
layout: default
title: LLM + Web Search Configuration
nav_order: 6
---

# LLM + Web Search Configuration

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Model tiering

Use model specialization instead of one model for all tasks:

- **Reasoning model** (for orchestration, planning): `gpt-4.1`
- **Fast extraction model** (for transcript parsing): `gpt-4.1-mini`
- **Optional summary model** (for executive outputs): `gpt-4.1-mini` or equivalent deployment

## Prompting and output contracts

- Enforce JSON schema for extraction outputs.
- Keep system instructions role-specific per agent.
- Include strict rules: no uncited project facts, no direct SQL generation.

## Tool-calling policy

- Allow only vetted tool functions (stored-procedure-backed SQL tools, retrieval tools).
- Restrict tool execution by agent role.
- Require retries with bounded backoff on transient tool failures.

## Web search integration

Use web search only for enrichment scenarios (e.g., external standards, regulatory updates):

1. Query Foundry IQ first.
2. If confidence/context is insufficient, invoke web search connector.
3. Normalize and rank web results.
4. Surface external facts separately from internal project facts.

## Recommended search services

- **Primary internal grounding**: Foundry IQ
- **Optional enterprise search augmentation**: Azure AI Search
- **Optional external web enrichment**: configured web search connector in Foundry toolchain

## Safety and governance settings

- Enable Azure AI Content Safety for prompts and outputs.
- Log prompts, model version, and tool traces.
- Redact PII from transcript chunks before long-term storage where policy requires.

## Performance tuning

- Cache project context summary per project/version.
- Cache transcript extraction outputs by transcript hash.
- Set token budgets per agent to avoid context bloat.
