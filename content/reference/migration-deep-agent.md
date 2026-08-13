---
title: "Migration: Pre-Deep Agent"
weight: 40
description: "Breaking changes checklist for adopters of the old template-agent and template-ui before the August 2026 Deep Agent merge."
---

If you used **template-agent** or **template-ui** before the August 2026 Deep Agent merge, this checklist summarizes what changed and how to migrate.

## Summary of changes

| Area | Before | After |
|------|--------|-------|
| Agent framework | Simple LangGraph script | [Deep Agents](https://github.com/langchain-ai/deepagents) + Aegra CLI |
| Package layout | `template_agent/` | `deep_agent/` + `config/agent/` |
| Run command | `python -m template_agent...` | `make local` / `aegra dev --port 5002` |
| Agent port | Varied (often 8000) | **5002** (LangGraph API) |
| Prompts | Embedded in Python | `config/agent/PROMPT.md` + subagents |
| MCP config | Env vars / Python settings | `config/agent/mcp.json` + frontmatter |
| Dependencies | Agent only | **PostgreSQL + Redis** required |
| Python version | 3.12 | **3.13+** |
| UI backend | FastAPI split or WebSockets | **Fastify BFF** (single Node app) |
| UI structure | `frontend/` + `backend/` dirs | `src/frontend/` + `src/server/` |
| UI → agent | `AGENT_URL`, port 8000/8001 | `AGENT_HOST` / `AGENT_ENDPOINT`, port **5002** |
| UI dev | Separate frontend/backend starts | `npm run dev` (unified) |
| Agent responses | HTML/Tailwind possible | **Markdown-first** rendering |
| UI branding | Code or env only | `config/ui/settings.yaml` + hot-reload |

## template-agent migration steps

1. **Re-clone or merge** from current `main` after [PR #104](https://github.com/redhat-data-and-ai/template-agent/pull/104).

2. **Install prerequisites:** Python 3.13+, Podman, uv.

3. **Replace run workflow:**
   ```bash
   make install
   make local
   curl http://localhost:5002/health
   ```

4. **Move prompts** from Python strings to:
   - `config/agent/PROMPT.md` (orchestrator)
   - `config/agent/subagents/*.md`

5. **Move MCP configuration** to `config/agent/mcp.json`. Wire servers via frontmatter `mcps:` on orchestrator and subagents.

6. **Configure `.env`** for Postgres, Redis, and `GOOGLE_APPLICATION_CREDENTIALS_CONTENT` (see `.env.example`).

7. **Update API clients** to LangGraph API:
   - `POST /threads`
   - `POST /threads/{thread_id}/runs/stream`
   - Assistant ID: `agent`

8. **Update deployment manifests** — health checks on port **5002**, Postgres and Redis components in OpenShift overlay.

## template-ui migration steps

1. **Re-clone or merge** from current `main` after [PR #76](https://github.com/redhat-data-and-ai/template-ui/pull/76).

2. **Install Node.js 22+.**

3. **Replace startup:**
   ```bash
   cp env.template .env
   # AGENT_HOST=http://localhost:5002
   # AUTH_ENABLED=false  # for local dev
   npm install
   npm run dev
   ```

4. **Remove references** to:
   - `python -m backend.main`
   - `AGENT_URL=http://localhost:8000`
   - Separate `frontend/` and `backend/` directory layout (now under `src/`)

5. **Add `config/ui/settings.yaml`** for branding (copy from `config/ui/examples/`).

6. **Expect Markdown** agent responses — UI uses ReactMarkdown auto-detection.

7. **Use new UI features** optionally: HITL banners, task progress, MCP status panel, debug panel.

## API client migration

If your app called the old agent HTTP API directly:

| Old assumption | New approach |
|----------------|--------------|
| Custom `/v1/stream` endpoint | LangGraph `POST /threads/{id}/runs/stream` |
| Single monolithic agent | Orchestrator + subagents; stream includes delegation events |
| Direct browser → agent | Use template-ui BFF or implement LangGraph client |

Use [template-ui](https://github.com/redhat-data-and-ai/template-ui) as the reference BFF implementation.

## Documentation map

| Topic | Updated docs |
|-------|--------------|
| Agent overview | [/templates/agent/](/templates/agent/) |
| Agent quick start | [/templates/agent/quick-start/](/templates/agent/quick-start/) |
| UI overview + BFF | [/templates/ui/](/templates/ui/) |
| Config reference | [/reference/config-as-code/](/reference/config-as-code/) |
| Announcements | [/news/](/news/) |
| Upstream frameworks | [Deep Agents](https://github.com/langchain-ai/deepagents), [LangGraph](https://langchain-ai.github.io/langgraph/), [Agent Skills](https://agentskills.io/specification) |

## Getting help

- [template-agent issues](https://github.com/redhat-data-and-ai/template-agent/issues)
- [template-ui issues](https://github.com/redhat-data-and-ai/template-ui/issues)
- [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions)

{{< warning >}}
There is no automatic upgrade path from pre-Deep Agent forks. Treat this as a **re-adoption** of the template: migrate config and prompts into the new `config/agent/` and `config/ui/` layouts rather than patching old Python modules.
{{< /warning >}}
