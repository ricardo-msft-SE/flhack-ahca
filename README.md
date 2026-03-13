# flhack-ahca

**Florida Hackathon — AHCA Policy Bot**

This repository contains the architectural design documentation and source code for the **AHCA Policy Bot**, a multi-agent AI system built on Microsoft Azure AI Foundry and Azure AI Services.

## 📖 Documentation

Live documentation site: [https://ricardo-msft-se.github.io/flhack-ahca/policybot/](https://ricardo-msft-se.github.io/flhack-ahca/policybot/)

The documentation covers:

- **Architecture** — Component diagram, Azure services, and data flow
- **Multi-Agent Design** — Orchestrator, Policy, Web Search, Fee Schedule, Confidence Evaluator, and Escalation agents
- **Confidence Scoring** — Three-dimensional evaluation framework with weighted geometric mean scoring
- **Human-in-the-Loop** — Teams adaptive card escalation workflow
- **LLM Configuration** — Azure OpenAI, Bing Search, grounding, and hallucination prevention
- **Foundry IQ** — Knowledge index, document ingestion, hybrid search configuration
- **Deployment** — Step-by-step Azure infrastructure setup guide
- **Glossary** — All terms and acronyms defined

## 🏗️ Repository Structure

```
├── policybot/           # Jekyll documentation site
│   ├── _config.yml      # Jekyll + Just the Docs configuration
│   ├── index.md         # Home page
│   ├── docs/            # Documentation pages
│   └── _data/           # Glossary and navigation data
├── .github/
│   └── workflows/
│       └── pages.yml    # GitHub Actions: Jekyll build + Pages deploy
└── README.md
```

## 🚀 Running the Docs Locally

```bash
cd policybot
bundle install
bundle exec jekyll serve --livereload
# Visit http://localhost:4000/flhack-ahca/policybot/
```

## 🛠️ Technology Stack

- **Azure AI Foundry** — Agent orchestration platform
- **Azure OpenAI (GPT-4o)** — Primary LLM
- **Azure AI Search** — Hybrid retrieval (vector + BM25 + semantic reranker)
- **Foundry IQ** — Knowledge index lifecycle management
- **Bing Search API** — Web policy source retrieval
- **Microsoft Teams** — Primary chat channel
- **Azure Bot Service** — Bot hosting and channel routing
- **Python** — Agent implementation language (Foundry Agent SDK)
- **Jekyll + Just the Docs** — Documentation site

## 📄 License

This project is licensed under the MIT License.
