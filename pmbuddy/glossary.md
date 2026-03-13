---
layout: default
title: Glossary
parent: PM Buddy
nav_order: 6
permalink: /pmbuddy/glossary/
---

# Glossary
{: .no_toc }

Key terms and definitions used throughout the PM Buddy documentation. Terms are grouped by category and sorted alphabetically within each group.

---

## Table of Contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

{% assign sorted_terms = site.data.glossary | sort: "term" %}
{% assign categories = sorted_terms | map: "category" | uniq | sort %}

{% for category in categories %}

## {{ category }}

{% for entry in sorted_terms %}{% if entry.category == category %}

### {{ entry.term }}

{{ entry.definition }}

{% endif %}{% endfor %}
{% endfor %}

---

*Missing a term? [Open an issue](https://github.com/ricardo-msft-SE/flhack-ahca/issues/new) to suggest an addition.*
