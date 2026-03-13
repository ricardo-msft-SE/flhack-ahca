---
layout: default
title: Foundry IQ Strategy
nav_order: 5
---

# Foundry IQ Strategy

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Role of Foundry IQ in PM Buddy

Foundry IQ acts as the semantic knowledge layer for:

- meeting transcript chunks,
- prior plan versions,
- generated artifacts and approvals,
- supporting reference material (methodology docs, templates).

## Ingestion design

1. Ingest transcript text and metadata (`meeting_date`, attendees, source system).
2. Chunk by semantic boundaries (topic / speaker turn windows).
3. Attach project and meeting IDs where available.
4. Index generated artifacts after approval to create a closed learning loop.

## Retrieval strategy

Use a hybrid approach:

- semantic retrieval from Foundry IQ as primary context,
- metadata filters (`project_id`, timeframe, artifact type),
- optional lexical fallback for exact identifiers.

## Grounding policy for generation

- Every plan/report section must include source citations.
- Prefer latest approved plan version + current transcript evidence.
- Exclude stale artifacts using version and date filters.

## Recommended index metadata

- `project_id`
- `meeting_id`
- `artifact_type`
- `version`
- `approved_by`
- `timestamp_utc`
- `sensitivity_label`

## Operating model

- Re-index after each approved meeting cycle.
- Run index quality checks weekly (coverage, stale content, duplication).
- Measure retrieval hit rate and citation precision as core KPIs.
