---
title: "Agent Template"
weight: 20
description: "Build Deep Agents with LangGraph, config-as-code orchestration, MCP integration, and enterprise observability on OpenShift."
github_repo: "https://github.com/redhat-data-and-ai/template-agent"
---

The Agent Template is a production-ready foundation for building [Deep Agents](https://github.com/langchain-ai/deepagents) with [LangGraph](https://langchain-ai.github.io/langgraph/) via the Aegra CLI. It provides orchestrator and subagent patterns, MCP tool integration, conversation persistence, and deployment overlays for OpenShift and Kind.

{{< info >}}
**August 2026 update:** The template merged the Deep Agent branch into `main`. See the [announcement](/news/deep-agent-merge/) for what changed.
{{< /info >}}

## Overview

This template gives you a complete Deep Agent runtime:

- **Orchestrator + subagents** — delegate work across specialized agents (example domain: fitness assistant)
- **Skills** — reusable workflow documents (`client-intake`, `bmi-report`, `email-formatter`) following the [Agent Skills specification](https://agentskills.io/specification)
- **Config-as-code** — prompts, subagents, MCP wiring, and runtime settings in `config/agent/`
- **MCP integration** — SSO pass-through, OAuth, and Dynamic Client Registration (DCR)
- **Enterprise infrastructure** — PostgreSQL checkpoints, Redis SSE broker, Langfuse tracing, OpenTelemetry metrics

MCP servers and the chat UI are **separate repositories**. This template runs the agent and its dependencies (Postgres, Redis) only.

## Key Features

**Deep Agent orchestration**
The orchestrator routes requests to subagents, manages task queues, and coordinates multi-step workflows with human-in-the-loop (HITL) support.

**MCP tool integration**
Register MCP servers in `mcp.json` and attach them to the orchestrator or individual subagents via markdown frontmatter. Multiple auth modes for enterprise deployments.

**Config-as-code runtime**
Customize agent behavior by editing `PROMPT.md`, `subagents/*.md`, and YAML config — no Python changes for most operational updates. See [Architecture](/templates/agent/architecture/) for details.

**Production deployment**
Red Hat UBI container, OpenShift overlays with HPA and PDB, Kind overlay for local Kubernetes testing. Health endpoints at `/health`, `/readyz`, and `/livez` on port **5002**.

## Architecture

The diagram below illustrates the **example domain** shipped with the template—a **Red Hat Fitness Assistant**—not a generic blank agent. It shows how the runtime components connect when you run the repo out of the box.

In this example, a client ([template-ui](/templates/ui/) or any LangGraph API client) sends requests to the **Aegra API** on port 5002. The **orchestrator** handles intake and routing, then delegates to two subagents:

- **Analyst** — runs health-metric analysis and research using MCP tools (for example BMI calculation)
- **Publisher** — formats output and handles delivery tasks such as email

Tools are provided by [template-mcp-server](/templates/mcp-server/) on port 5001. Prompts, subagent definitions, and MCP wiring live in `config/agent/` and load at startup—no embedded Python prompts.

When you adapt the template for your own domain, you replace the fitness-assistant prompts, subagent names, and skills while keeping the same architecture: orchestrator, subagents, MCP client, PostgreSQL, and Redis.

{{< mermaid >}}
graph TD
    Client[Client or template-ui] --> API[Aegra LangGraph API :5002]
    API --> Orch[Orchestrator]
    Orch --> SubA[Subagent: analyst]
    Orch --> SubB[Subagent: publisher]
    SubA --> MCP[MCP Client]
    SubB --> MCP
    MCP --> MCPSrv[template-mcp-server :5001]
    MCPSrv --> Ext[External Systems]

    Config[config/agent/] --> Orch
    Config --> SubA
    Config --> SubB

    API --> PG[(PostgreSQL)]
    API --> Redis[(Redis)]

    style Orch fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    style MCP fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    style Config fill:#fff3e0,stroke:#ff9800,stroke-width:2px
{{< /mermaid >}}

<p class="figure-caption">Figure 1. Example Deep Agent request flow for the fitness-assistant template: orchestrator delegates to analyst and publisher subagents, which call MCP tools on template-mcp-server. Postgres stores checkpoints and feedback; Redis backs SSE streaming and OAuth tokens.</p>

## Getting Started

{{< info >}}
**Repository**: [template-agent on GitHub](https://github.com/redhat-data-and-ai/template-agent)
{{< /info >}}

```bash
git clone https://github.com/redhat-data-and-ai/template-agent.git
cd template-agent
make install
make local
curl http://localhost:5002/health
```

Follow the [Quick Start](/templates/agent/quick-start/) for credentials, MCP setup, and API testing.

## Use Cases

**Multi-agent workflows**
Orchestrate complex tasks across specialized subagents with clear delegation boundaries.

**Enterprise tool access**
Connect to internal systems via MCP servers with SSO, OAuth, or DCR authentication.

**Configurable domain agents**
Adapt the example fitness assistant to your domain by replacing prompts, skills, and subagent definitions.

**Observable production agents**
Trace conversations with Langfuse, export OpenTelemetry metrics, and collect user feedback via the API.

## Technology Stack

- **LangGraph Deep Agents** — orchestration framework
- **Aegra CLI** — LangGraph Platform dev server
- **Python 3.13+** — with `uv` package manager
- **PostgreSQL** — checkpoints, memory, feedback storage
- **Redis** — SSE broker and OAuth token storage
- **pytest** — unit, integration, and skills evaluations

## Related Templates

- [MCP Server Template](/templates/mcp-server/) — tools for your agent to call
- [UI Template](/templates/ui/) — chat interface for the LangGraph API
- [Stack reference](/reference/architecture/) — ports, request flow, deployment patterns
- [Config-as-code announcement](/news/config-as-code-architecture/) — how configuration is organized

## Next Steps

1. [Quick Start](/templates/agent/quick-start/) — run locally in minutes
2. [Architecture](/templates/agent/architecture/) — config layout, MCP auth, API reference
3. [Deployment Guide](/templates/agent/deployment/) — OpenShift, containers, production hardening
4. [GitHub repository](https://github.com/redhat-data-and-ai/template-agent) — latest README and issues

{{< tip >}}
**Full stack:** Run [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server) on `:5001` and [template-ui](https://github.com/redhat-data-and-ai/template-ui) for a complete chat experience against the agent on `:5002`.
{{< /tip >}}
