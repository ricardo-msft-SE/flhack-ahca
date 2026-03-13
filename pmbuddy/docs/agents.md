---
layout: default
title: Multi-Agent Design
nav_order: 4
---

# Multi-Agent Design with Agent Framework

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Agent topology

A coordinator pattern is recommended:

- **Orchestrator Agent**: receives user request, manages workflow state, delegates tasks.
- **Transcript Intake Agent**: chunking, entity extraction, decisions/actions detection.
- **Project Match Agent**: determines whether transcript maps to existing project context.
- **Planning Agent**: creates or revises project plan, milestones, risks, dependencies.
- **Artifact Agent**: generates PM outputs (status report, RAID log draft, executive summary).
- **SQL Tool Agent**: executes controlled SQL read/write operations through approved stored procedures.
- **Quality & Policy Agent**: validates confidence, citation coverage, and policy constraints.

## Where SQL Tool Agent adds value over pure Foundry IQ

| Need | Foundry IQ only | SQL Tool Agent + IQ |
|---|---|---|
| Semantic grounding | Strong | Strong |
| Deterministic updates | Limited | Strong |
| Transaction control | Limited | Strong |
| Foreign-key integrity | Limited | Strong |
| Complex joins for reporting | Limited | Strong |
| Audit-ready change history | Moderate | Strong |

Recommendation: keep all authoritative project mutations behind SQL stored procedures and expose them to agents as tools.

## Suggested tool interfaces

- `find_project_candidates(transcript_facts)`
- `create_project(project_payload)`
- `append_meeting_context(project_id, extracted_facts)`
- `upsert_action_items(project_id, action_items)`
- `create_plan_version(project_id, plan_markdown, citations)`

## Foundry agent configuration pattern

```yaml
# example conceptual fragment (agent.yaml)
name: pmbuddy-orchestrator
kind: prompt
model: gpt-4.1
instructions: |
  You orchestrate project-planning workflows from transcripts.
  Always call project_match first. If no match, create new project.
  Persist all approved outputs through sql_tool_agent.
tools:
  - agent: transcript_intake
  - agent: project_match
  - agent: planning
  - agent: artifact
  - agent: sql_tool
  - agent: quality_policy
```

## Control policy

- Require citations for every generated decision/risk/action.
- Block writes when confidence is below threshold until user approval.
- Log every tool call, prompt hash, and output artifact version.

## Recommended confidence thresholds

- `>= 0.85`: auto-save draft artifacts.
- `0.70 - 0.84`: save as review-required.
- `< 0.70`: escalate to human review before persistence.
