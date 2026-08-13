---
title: "Stack Architecture"
weight: 10
description: "Full-stack architecture, default ports, and request flow for MCP, Deep Agent, and UI templates."
---

This page describes how the three AI Templates repositories connect in a typical local or production deployment.

## Full-stack diagram

{{< mermaid >}}
graph TD
    User[User / Browser] --> UI[template-ui<br/>React + Fastify BFF]
    UI --> Agent[template-agent<br/>Aegra LangGraph API]
    Agent --> MCP[template-mcp-server]
    MCP --> Ext[External APIs and data]

    Agent --> PG[(PostgreSQL)]
    Agent --> RedisA[(Redis)]
    UI --> RedisU[(Redis sessions)]

    ConfigA[config/agent/] --> Agent
    ConfigU[config/ui/settings.yaml] --> UI

    style UI fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style Agent fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    style MCP fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
{{< /mermaid >}}

<p class="figure-caption">Figure 1. AI Templates stack: UI BFF proxies the Deep Agent API, which calls MCP tools. Postgres and Redis support agent checkpoints and streaming; UI Redis stores sessions when auth is enabled.</p>

## Default ports (local development)

| Component | Port | Health check |
|-----------|------|--------------|
| template-mcp-server | **5001** | `curl http://localhost:5001/health` |
| template-agent | **5002** | `curl http://localhost:5002/health` |
| template-ui (Vite dev) | **5173** | Browser UI |
| template-ui (production) | **8080** | `npm start` default |
| UI Redis (compose) | **6380** | Session store |
| Agent Redis (compose) | **6379** | SSE broker, OAuth tokens |
| Agent Postgres | **5432** | Checkpoints, memory, feedback |
| MCP Postgres (compose, auth enabled) | **5432** | OAuth token storage — host port conflicts with agent Postgres if both compose stacks run |

{{< tip >}}
**Local Postgres:** template-agent and template-mcp-server both map Postgres to host **5432** in their compose files. Run one database stack, point the other service at a shared instance, or change the host port mapping when running both composes on one machine.
{{< /tip >}}

## Request flow

1. **User** opens the chat UI in a browser (`:5173` in dev).
2. **template-ui BFF** receives chat actions, manages session/auth, and proxies to template-agent.
3. **template-agent** runs the LangGraph Deep Agent graph (assistant ID `agent`).
4. The **orchestrator** may delegate to **subagents** and call **MCP tools** on template-mcp-server.
5. **SSE streams** flow back through the BFF, which translates events for React (HITL, todos, artifacts, MCP status).
6. **Checkpoints** and feedback persist in agent PostgreSQL; per-user MCP OAuth tokens from the agent flow live in agent Redis. When template-mcp-server auth is enabled, the MCP server stores its own OAuth tokens in PostgreSQL.

## Repository responsibilities

| Repository | Role |
|------------|------|
| [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server) | MCP tools and external integrations |
| [template-agent](https://github.com/redhat-data-and-ai/template-agent) | Deep Agent runtime, LangGraph API, config-as-code agent behavior |
| [template-ui](https://github.com/redhat-data-and-ai/template-ui) | Chat UI, BFF proxy, branding, SSO |

Each repo can be deployed independently but is designed to work together.

## Configuration boundaries

| Layer | Config location | Secrets |
|-------|-----------------|---------|
| Agent behavior | `config/agent/` in template-agent | `.env` (DB, Vertex, Langfuse, SSO) |
| UI branding and proxy | `config/ui/settings.yaml` in template-ui | `.env` (SSO, `COOKIE_SIGN`, `AGENT_HOST`) |
| Agent → MCP wiring | `config/agent/mcp.json` in template-agent | Per-server auth in agent `.env` (`MCP_TOKEN_ENCRYPTION_KEY`, etc.) |
| MCP server runtime | `.env` in template-mcp-server (Pydantic settings) | `SSO_*`, `SESSION_SECRET`, `POSTGRES_*` when `ENABLE_AUTH=True` |

See [Config-as-code](/reference/config-as-code/) for details.

## Deployment patterns

| Pattern | Command / path |
|---------|----------------|
| Local MCP only | `make local` in template-mcp-server (`ENABLE_AUTH=False` in `.env`) |
| Local agent only | `make local` in template-agent |
| Local full stack | MCP `make local` + agent `make local` + UI `npm run dev` |
| MCP containers | `make container` in template-mcp-server (server + Postgres) |
| Agent containers | `make container` in template-agent |
| UI + Redis compose | `compose.yml` in template-ui |
| OpenShift MCP | `deployment/openshift/` in template-mcp-server |
| OpenShift agent / UI | `deployment/overlays/openshift/` in template-agent and template-ui |
| Kind | `make kind` (agent) / `deployment/overlays/kind/` (UI) |

## Related documentation

- [Agent Architecture](/templates/agent/architecture/)
- [UI Configuration](/templates/ui/configuration/)
- [Agent Deployment](/templates/agent/deployment/)
- [UI Deployment](/templates/ui/deployment/)
