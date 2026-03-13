---
layout: default
title: Deployment
nav_order: 7
---

# Deployment to GitHub Pages Path

## Table of contents
{: .no_toc }
1. TOC
{:toc}

## Target publishing location

- Repository: `https://github.com/ricardo-msft-SE/flhack-ahca`
- Docs folder: `/pmbuddy`
- Expected URL path: `https://ricardo-msft-se.github.io/flhack-ahca/pmbuddy/`

## Why this works

`_config.yml` is set with:

```yaml
baseurl: "/flhack-ahca/pmbuddy"
url: "https://ricardo-msft-se.github.io"
```

This ensures generated links resolve under `/pmbuddy`.

## Local preview

From `pmbuddy/`:

```bash
bundle install
bundle exec jekyll serve
```

Browse:
- `http://127.0.0.1:4000/flhack-ahca/pmbuddy/`

## GitHub Pages notes

- If using a Pages workflow, point build source to `pmbuddy/`.
- If using branch-based Pages, ensure the published artifact contains this subfolder output.
- Keep `baseurl` aligned with repo and folder path.

## Operational checklist

- [ ] Foundry project and model deployments configured.
- [ ] Foundry IQ index configured and seeded.
- [ ] SQL schema + stored procedures deployed.
- [ ] Agent configurations promoted across environments.
- [ ] Monitoring and alerts configured (latency, failures, low confidence rate).
