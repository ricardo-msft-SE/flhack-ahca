---
layout: default
title: Overview
nav_order: 2
---

# Overview

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Problem statement

Teams spend time translating meeting transcripts into plans, statuses, and artifacts. This app automates that workflow while preserving project continuity and governance.

## Target outcomes

- Reduce time from meeting completion to updated project plan.
- Keep one canonical project state for decisions, risks, actions, and deliverables.
- Improve consistency of PM outputs across teams.
- Provide traceable provenance from transcript evidence to generated artifacts.

## Core user journeys

![Project Management Buddy core flow]({{ '/assets/images/arch7.png' | relative_url }})

1. Upload or connect a meeting transcript.
2. System extracts project signals (name, people, milestones, blockers, decisions).
3. System checks for related existing project context.
4. Branch:
   - no match: create a new project record with baseline plan,
   - match: append updates and generate revised artifacts.
5. User reviews and approves generated outputs.

## Scope boundaries

Included:
- transcript understanding,
- project matching,
- plan/report/artifact generation,
- grounded retrieval with Foundry IQ,
- project persistence in Azure SQL Database.

Not included in first iteration:
- task execution in external ALM tools,
- portfolio-level budgeting,
- custom BI dashboards.
