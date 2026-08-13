---
title: "template-ui updated for Deep Agent streaming"
date: 2026-08-12T14:00:00-06:00
description: "The UI template now proxies the Deep Agent LangGraph API with HITL, task progress, artifacts, and runtime branding configuration."
tags:
  - announcement
  - deep-agent
  - template-ui
---

[template-ui](https://github.com/redhat-data-and-ai/template-ui) has been reworked to work with the new **template-agent** Deep Agent backend ([PR #76](https://github.com/redhat-data-and-ai/template-ui/pull/76)). The UI is not a separate Deep Agent package—it is the chat front end for the agent's LangGraph API.

## What changed

**Deep Agent integration**
- Proxies LangGraph SSE streams from template-agent (`:5002` by default)
- Translates streaming events for the React frontend
- Supports subagent indicators, HITL interrupt banners, todo/task progress, artifact viewing, and MCP connection status panels
- Renders agent responses as **Markdown** (not HTML/Tailwind from the model)

**Runtime configuration**
- New `config/ui/settings.yaml` — branding, feature flags, agent endpoint, security settings
- Environment overrides: `BRANDING_TITLE`, `AGENT_ENDPOINT`, `FEATURE_AUTH_ENABLED`, and others
- Hot-reload for branding and agent endpoint changes without rebuild

**New UI capabilities**
- Settings: profile, memory, rules editor, appearance
- Debug panel for stream inspection
- Feedback buttons wired to the agent feedback API
- Reconnecting banner, session expired modal, error recovery
- Redis for sessions (compose includes Redis on port **6380**)

**Deployment**
- Kustomize overlays for Kind and OpenShift
- Expanded CI: CodeRabbit, OpenSSF Scorecard, release-please, license scan

## Stack wiring

| Component | Default port |
|-----------|--------------|
| template-mcp-server | 5001 |
| template-agent | 5002 |
| template-ui | 5173 (dev) / 8080 (production) |

Agent endpoint resolution: `config/ui/settings.yaml` → `AGENT_HOST` env var → `http://localhost:5002`.

## What this means for you

- **Run template-agent first** — the UI expects the LangGraph API on `:5002`.
- **Copy `env.template` to `.env`** and set `AGENT_HOST` if not using `settings.yaml`.
- **Use `npm run dev`** for local development (Node.js 22+).
- **Pair with [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server)** for tools the agent can call.

For architecture details on how the UI connects to the agent, see the [UI Template](/templates/ui/) documentation (updating in a follow-on release).

Related: [Deep Agent merge](/news/deep-agent-merge/) · [Config-as-code architecture](/news/config-as-code-architecture/)
