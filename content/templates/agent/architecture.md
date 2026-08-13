---
title: "Architecture"
weight: 8
description: "Deep Agent config-as-code layout, MCP authentication modes, LangGraph API endpoints, and enterprise runtime features."
---

This page describes how **template-agent** is structured: config-as-code layout, request flow, MCP wiring, and the HTTP API.

## Config-as-code layout

The agent process loads behavior from `config/agent/` at startup. Secrets and infrastructure endpoints come from `.env`; operational settings from YAML.

```
config/agent/
├── PROMPT.md               # Orchestrator prompt + YAML frontmatter
├── subagents/              # Subagent definitions (.md files)
├── skills/                 # Skill directories ([Agent Skills spec](https://agentskills.io/specification))
├── mcp.json                # MCP server registry
├── runtime/agent.yaml      # Cache, memory, providers, middleware
└── deployment/values.yaml  # OpenShift/ArgoCD reference values
```

### Orchestrator (`PROMPT.md`)

The orchestrator prompt uses YAML frontmatter for model, tools, skills, and MCP attachments:

```yaml
---
name: orchestrator
model: gemini-2.5-pro
tools:
  - validate_email
  - queue_task
skills:
  - client-intake
---
```

The markdown body defines routing logic, delegation rules, and domain identity.

### Subagents (`subagents/*.md`)

Each subagent is a markdown file with frontmatter for model, MCPs, and tools:

```yaml
---
name: analyst
model: gemini-2.5-pro
mcps:
  - template-mcp-server
tools:
  - calculate_bmi
  - search_web
---
```

Subagents without their own `mcps` list inherit the orchestrator's MCP configuration.

### Runtime settings (`runtime/agent.yaml`)

Controls cache layers, memory consolidation, provider profiles, middleware (guardrails, audit, PII), and agent identity. OpenShift deployments typically mount secrets as env vars and keep operational settings in this file or a ConfigMap.

## Request flow

{{< mermaid >}}
flowchart TD
    Req[HTTP request] --> Aegra[Aegra HTTP app]
    Aegra --> Graph[LangGraph Deep Agent]
    Graph --> Orch[Orchestrator]
    Orch -->|delegate| Sub[Subagent]
    Sub --> Tools[MCP tools / local tools]
    Tools --> Resp[Streamed response]

    PG[(PostgreSQL)] --- Aegra
    Redis[(Redis)] --- Aegra
{{< /mermaid >}}

1. Client (curl, template-ui, or custom app) calls the LangGraph API on port **5002**
2. Aegra loads the graph defined in `aegra.json` (assistant ID: `agent`)
3. The orchestrator classifies intent and delegates to subagents or skills
4. Subagents call MCP tools or built-in tools
5. Responses stream via SSE; checkpoints persist in PostgreSQL

## MCP configuration

### Registry (`mcp.json`)

MCP servers are registered with URL, transport, auth mode, and enabled flag. Example local entry:

```json
{
  "mcpServers": {
    "template-mcp-server": {
      "url": "http://localhost:5001/mcp",
      "transport": "streamable_http",
      "enabled": true,
      "auth": false
    }
  }
}
```

### URL by run mode

| Mode | Typical MCP URL |
|------|-----------------|
| `make local` (agent on host) | `http://localhost:5001/mcp` |
| `make container` (MCP on host) | `http://host.containers.internal:5001/mcp` |

Alternate URLs are documented as comments in `mcp.json`.

### Auth modes

| Mode | When to use |
|------|-------------|
| `sso` (default) | MCP accepts the same SSO token as the agent |
| `oauth` | MCP has a pre-registered OAuth client; user connects via UI |
| `dcr` | MCP supports OAuth Dynamic Client Registration |

Set `"auth": false` for public local MCP servers. OAuth and DCR tokens are stored encrypted in Redis (`MCP_TOKEN_ENCRYPTION_KEY` required).

### Tool name prefix

When multiple MCP servers are configured, tool names are prefixed with the server key. Use `tool_prefix` in `mcp.json` for shorter names.

## LangGraph API

Standard LangGraph Platform endpoints (assistant ID: `agent`):

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/ok` | GET | Server health |
| `/threads` | POST | Create conversation thread |
| `/threads/{thread_id}/runs/stream` | POST | Run agent (SSE stream) |
| `/threads/{thread_id}` | GET | Get thread state |

### Custom routes

| Endpoint | Description |
|----------|-------------|
| `/health`, `/healthz`, `/readyz`, `/livez` | Health probes |
| `/info` | Agent name and OAuth/DCR MCP server list |
| `/feedback` | Record user feedback (Langfuse + Postgres) |
| `/threads/{thread_id}/token-usage` | Cumulative token usage |
| `/mcp/{name}/connect` | Start OAuth/DCR flow |
| `/mcp/{name}/status` | MCP connection status |

Full curl examples are in the [Quick Start](/templates/agent/quick-start/).

## Enterprise features

| Feature | Purpose |
|---------|---------|
| **Langfuse** | Optional tracing and feedback (`LANGFUSE_*` env vars) |
| **OpenTelemetry** | Metrics and distributed tracing |
| **SSO/OIDC** | `ENABLE_AUTH` and `SSO_*` for authenticated deployments |
| **Guardrails & audit** | Middleware for safety and compliance logging |
| **Token budget** | Per-thread usage tracking |
| **HITL** | Human-in-the-loop interrupts in streaming workflows |

## Project structure

```
template-agent/
├── aegra.json              # LangGraph framework entry point
├── config/agent/           # Config-as-code (see above)
├── deep_agent/
│   ├── aegra/              # Graph, HTTP app, MCP OAuth
│   └── src/                # Config loader, cache, memory, etc.
├── compose.yaml            # Postgres + Redis (+ agent profile)
├── Containerfile
├── deployment/overlays/    # OpenShift and Kind
└── tests/                  # unit, integration, skills evals
```

## Model providers

Default configuration uses **Google Vertex AI** (Gemini). The runtime also supports:

- **vLLM / OpenAI-compatible** endpoints via `VLLM_BASE_URL`
- **MaaS** (Models as a Service) for managed open-source models — set `provider: maas` in frontmatter

See `config/agent/runtime/agent.yaml` and the [template-agent README](https://github.com/redhat-data-and-ai/template-agent) for provider configuration.

## Upstream frameworks

template-agent is built on:

- [LangGraph Deep Agents](https://github.com/langchain-ai/deepagents) — orchestrator, subagents, and delegation patterns
- [LangGraph](https://langchain-ai.github.io/langgraph/) — graph runtime, checkpoints, and LangGraph Platform API
- [Agent Skills](https://agentskills.io/specification) — `SKILL.md` format for skills in `config/agent/skills/`

For MCP tool protocols, see the [Model Context Protocol](https://modelcontextprotocol.io/) and [MCP Server Template](/templates/mcp-server/) docs.

## Next Steps

- [Quick Start](/templates/agent/quick-start/) — run locally
- [Deployment Guide](/templates/agent/deployment/) — OpenShift and production
- [Config-as-code news post](/news/config-as-code-architecture/) — announcement and migration context
