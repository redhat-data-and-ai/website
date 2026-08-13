---
title: "Config-as-code: stateless Deep Agent runtime"
date: 2026-08-12T09:00:00-06:00
description: "template-agent loads orchestrator prompts, subagents, skills, and MCP wiring from config/agent/ at runtime—no embedded Python prompts."
tags:
  - announcement
  - deep-agent
  - template-agent
  - architecture
---

With the Deep Agent merge, **template-agent** separates **runtime** from **configuration**. The agent process is stateless with respect to prompts and behavior: everything operational loads from `config/agent/` at startup.

## The config/agent/ layout

```
config/agent/
├── PROMPT.md               # Orchestrator prompt + YAML frontmatter
├── subagents/              # Subagent definitions (.md files)
├── skills/                 # Skill documents and evals
├── mcp.json                # MCP server registry
└── runtime/agent.yaml      # Cache, memory, providers, middleware
```

**Secrets and endpoints** (database passwords, API keys, Redis URL) still live in `.env`. **Operational settings** (model defaults, provider profiles, middleware) live in `runtime/agent.yaml`.

## What you configure without Python

| Concern | Where |
|---------|--------|
| Orchestrator identity and routing | `PROMPT.md` frontmatter + body |
| Subagent models, tools, MCPs | `subagents/*.md` frontmatter |
| MCP server URLs and auth mode | `mcp.json` |
| Runtime behavior (cache, memory, OTEL) | `runtime/agent.yaml` |

Example frontmatter in a subagent file:

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

## MCP auth modes

MCP servers in `mcp.json` support three auth modes:

| Mode | Use when |
|------|----------|
| `sso` (default) | MCP accepts the same SSO token as the agent |
| `oauth` | MCP has a pre-registered OAuth client |
| `dcr` | MCP supports OAuth Dynamic Client Registration |

## Enterprise features in the runtime

Beyond config-as-code, the merged runtime adds:

- **Langfuse** — optional tracing and feedback correlation
- **OpenTelemetry** — metrics and distributed tracing hooks
- **Guardrails and audit** — middleware for safety and compliance logging
- **Token budget tracking** — per-thread usage endpoints
- **Personalization** — memory and rules (API in development)

## What this means for you

- **Customize behavior by editing markdown and YAML**, not Python, for most agent changes.
- **Version-control your agent config** separately from application code if you deploy config via ConfigMaps or GitOps.
- **Validate MCP names** — every entry in a subagent's `mcps:` list must exist in `mcp.json` with `enabled: true`.
- **Check ports** — agent API is `:5002`; MCP default is `:5001`.

Related: [Deep Agent merge](/news/deep-agent-merge/) · [template-ui update](/news/template-ui-deep-agent-update/)
