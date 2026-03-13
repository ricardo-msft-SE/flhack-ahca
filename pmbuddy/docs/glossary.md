---
layout: default
title: Glossary
nav_order: 8
---

# Glossary

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Terms

| Term | Definition |
|---|---|
{% for item in site.data.glossary %}
| {{ item.term }} | {{ item.definition }} |
{% endfor %}

## Acronyms quick list

- **AOAI**: Azure OpenAI
- **HITL**: Human in the Loop
- **KPI**: Key Performance Indicator
- **PMO**: Project Management Office
- **RAID**: Risks, Assumptions, Issues, Dependencies
