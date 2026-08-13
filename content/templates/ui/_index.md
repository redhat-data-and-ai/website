---
title: "UI Template"
weight: 30
description: "React chat UI with a Fastify backend-for-frontend that proxies the Deep Agent LangGraph streaming API."
github_repo: "https://github.com/redhat-data-and-ai/template-ui"
---

The UI Template is a production-ready chat interface for [template-agent](/templates/agent/) Deep Agents. It combines a React frontend with a **Fastify backend-for-frontend (BFF)** that proxies the LangGraph API, translates streaming events for the UI, and handles sessions, auth, and runtime branding.

{{< info >}}
**August 2026 update:** template-ui was reworked for the Deep Agent backend. See the [announcement](/news/template-ui-deep-agent-update/).
{{< /info >}}

## What is a backend-for-frontend (BFF)?

A **backend-for-frontend** is a server tailored to your UI's needs. The browser does **not** call template-agent directly.

Instead:

1. The **React app** talks only to the Fastify BFF (same origin in production)
2. The **BFF** proxies requests to template-agent's LangGraph API on port **5002**
3. The BFF **translates** SSE streams into UI-friendly events (subagent progress, HITL interrupts, artifacts, MCP status)
4. The BFF handles **sessions**, SSO, rate limiting, and **runtime branding** from `config/ui/settings.yaml`

Agent logic stays in template-agent. The UI focuses on presentation, auth, and stream shaping. See [Configuration](/templates/ui/configuration/) for the full settings reference.

## Overview

This template provides:

- **React chat UI** — streaming responses, markdown rendering, conversation history
- **Fastify BFF** — LangGraph API proxy with Deep Agent event translation
- **Runtime configuration** — branding, feature flags, and agent endpoint without rebuilds
- **Deep Agent UI features** — HITL interrupt banners, task/todo progress, artifact viewer, MCP status panel
- **Enterprise options** — SSO/OAuth, OPA compliance policies, OpenTelemetry tracing, Redis sessions

## Key Features

**Deep Agent streaming**
Proxies `/threads` and `/runs/stream` from template-agent. Supports subagent indicators, intermediate responses, and markdown-first agent output.

**Runtime branding**
Change title, logo, colors, and feature flags via `config/ui/settings.yaml` or environment variables. Branding and agent endpoint changes hot-reload without restart.

**Settings and personalization**
Profile, memory, rules editor, appearance settings, and always-allowed tools (where enabled in agent backend).

**Debug and feedback**
Debug panel for stream inspection; feedback buttons wired to the agent feedback API.

**Production deployment**
Kustomize overlays for Kind and OpenShift, `compose.yml` with Redis, Playwright e2e tests, and UBI-based Containerfile.

## Architecture

{{< mermaid >}}
graph TD
    Browser[Web Browser] --> React[React + Vite :5173]
    React --> BFF[Fastify BFF]
    BFF --> Agent[template-agent LangGraph API :5002]
    Agent --> MCP[template-mcp-server :5001]

    BFF --> Redis[(Redis sessions)]
    Config[config/ui/settings.yaml] --> BFF

    style React fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style BFF fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    style Agent fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    style MCP fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
{{< /mermaid >}}

<p class="figure-caption">Figure 1. template-ui request flow: the browser talks to the Fastify BFF, which proxies and translates Deep Agent streams from template-agent.</p>

## Getting Started

{{< info >}}
**Repository**: [template-ui on GitHub](https://github.com/redhat-data-and-ai/template-ui)
{{< /info >}}

```bash
git clone https://github.com/redhat-data-and-ai/template-ui.git
cd template-ui
cp env.template .env
# Set AGENT_HOST=http://localhost:5002 and AUTH_ENABLED=false for local dev
npm install
npm run dev
```

Open **http://localhost:5173**. Ensure [template-agent](/templates/agent/quick-start/) is running on `:5002` first.

## Technology Stack

**Frontend:** React 19, TypeScript, Vite, PatternFly 6, Tailwind CSS, React Markdown

**Backend (BFF):** Fastify, TypeScript, OAuth2/SSO, Redis session store, OPA compliance plugin

## Related Templates

- [Agent Template](/templates/agent/) — Deep Agent runtime the UI connects to
- [MCP Server Template](/templates/mcp-server/) — tools used by the agent
- [Stack reference](/reference/architecture/) — full-stack ports and BFF request flow
- [template-ui update announcement](/news/template-ui-deep-agent-update/)

## Next Steps

1. [Quick Start](/templates/ui/quick-start/) — local dev with template-agent
2. [Configuration](/templates/ui/configuration/) — branding, feature flags, agent endpoint
3. [Deployment Guide](/templates/ui/deployment/) — OpenShift, containers, production hardening
4. [GitHub repository](https://github.com/redhat-data-and-ai/template-ui)

{{< tip >}}
**Full stack ports:** MCP `:5001` → Agent `:5002` → UI `:5173` (dev) or `:8080` (production).
{{< /tip >}}
