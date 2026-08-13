---
title: "template-agent merges Deep Agent into main"
date: 2026-08-11
description: "The Agent template now builds on LangGraph Deep Agents with orchestrator, subagents, skills, and enterprise infrastructure."
tags:
  - announcement
  - deep-agent
  - template-agent
---

The [template-agent](https://github.com/redhat-data-and-ai/template-agent) repository has merged its `deep-agent` development branch into `main` ([PR #104](https://github.com/redhat-data-and-ai/template-agent/pull/104)). This is the largest architectural change since the template launched.

## What changed

The template is no longer a simple LangGraph script with embedded prompts. It is now a **Deep Agent** runtime built on [LangGraph Deep Agents](https://github.com/langchain-ai/deepagents) and the Aegra CLI.

**Agent capabilities:**
- Orchestrator with analyst and publisher subagents (example domain: fitness assistant)
- Skills: `client-intake`, `bmi-report`, `email-formatter`
- MCP integration with SSO pass-through, OAuth, and Dynamic Client Registration (DCR)

**Infrastructure:**
- Aegra dev server with Redis-backed SSE streaming on port **5002**
- PostgreSQL for checkpoints, memory, and feedback storage
- Langfuse tracing and OpenTelemetry metrics
- OpenShift and Kind deployment overlays

## What stayed the same

The three-repo model is unchanged: **template-agent**, [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server), and [template-ui](https://github.com/redhat-data-and-ai/template-ui) still work together as a stack. MCP and UI remain separate repositories.

## What this means for you

- **Clone fresh** if you forked the old template — the `deep_agent/` package and `config/agent/` layout replace the previous `template_agent/` structure.
- **Use `make local`** instead of `python -m agent_template` — the agent runs via Aegra on `:5002`.
- **Set Vertex AI credentials** in `.env` (`GOOGLE_APPLICATION_CREDENTIALS_CONTENT`) before first run.
- **Read the config-as-code announcement** for how prompts and subagents are defined without Python edits.

Next: [Config-as-code architecture](/news/config-as-code-architecture/) · [Agent Template docs](/templates/agent/) (updating)
