---
title: "Config-as-Code"
weight: 20
description: "How template-agent and template-ui load configuration at runtime without embedding prompts or branding in code."
---

**template-agent** and **template-ui** separate **runtime behavior** from **application code**. Secrets stay in environment variables; operational and presentation settings load from config files at startup (with selective hot-reload on the UI).

**template-mcp-server** uses **environment variables only** (Pydantic settings from `.env`) — no `config/agent/` tree. Tool implementations live under `template_mcp_server/src/tools/`.

## Philosophy

| Principle | template-agent | template-ui |
|-----------|----------------|-------------|
| Agent/UI logic in code | `deep_agent/`, Fastify routers | React components, proxy translation |
| Behavior & branding in config | `config/agent/` | `config/ui/settings.yaml` |
| Secrets in env | `.env` / OpenShift Secrets | `.env` / OpenShift Secrets |
| Stateless deploy | Swap config without rebuilding for most changes | Hot-reload branding and agent endpoint |

## template-agent: `config/agent/`

```
config/agent/
├── PROMPT.md               # Orchestrator prompt + frontmatter
├── subagents/*.md          # Subagent definitions
├── skills/*/               # Skill documents (see Agent Skills spec)
├── mcp.json                # MCP server registry
├── runtime/agent.yaml      # Cache, memory, providers, middleware
└── deployment/values.yaml  # GitOps reference values
```

### What lives in frontmatter (markdown)

Orchestrator and subagent files use YAML frontmatter for:

- `model` — LLM model name or provider block
- `tools` — built-in tool allow list
- `skills` — skill documents to load (see [Agent Skills specification](https://agentskills.io/specification))
- `mcps` — MCP server keys from `mcp.json`

### Skills (`config/agent/skills/`)

Each skill is a directory with a `SKILL.md` file following the [Agent Skills specification](https://agentskills.io/specification). Skills package domain workflows (prompts, references, scripts) that orchestrators and subagents invoke via frontmatter `skills:` lists. Evaluations live under `config/agent/skills/*/evals/`.

### What lives in `runtime/agent.yaml`

- Agent identity (`name`)
- Provider profiles and `resolve_strategy`
- Cache and memory consolidation settings
- Middleware (guardrails, audit, PII)
- OpenTelemetry and observability toggles

### What lives in `.env`

- `POSTGRES_*`, `REDIS_URL` — infrastructure
- `GOOGLE_APPLICATION_CREDENTIALS_CONTENT` — Vertex AI
- `ENABLE_AUTH`, `SSO_*` — authentication
- `LANGFUSE_*` — tracing
- `MCP_TOKEN_ENCRYPTION_KEY` — OAuth/DCR token storage

{{< info >}}
Deep dive: [Agent Architecture](/templates/agent/architecture/) and [news post](/news/config-as-code-architecture/).
{{< /info >}}

## template-ui: `config/ui/settings.yaml`

```
config/ui/
├── settings.yaml           # Runtime config (optional)
├── README.md               # Full schema
└── examples/               # minimal, blue-theme, red-hat-branding, production
```

### Key sections

```yaml
branding:
  title: "My Agent"
  logo_url: "/logo.svg"
  colors:
    light: { primary: "#0066cc", accent: "#a60000", ... }
    dark: { ... }

features:
  debug_mode_default: false
  auth_enabled: false

agent:
  endpoint: ""
  timeout_ms: 30000
  streaming: true
```

### Environment overrides

Env vars override YAML (examples):

| Variable | Overrides |
|----------|-----------|
| `BRANDING_TITLE` | `branding.title` |
| `FEATURE_AUTH_ENABLED` | `features.auth_enabled` |
| `AGENT_ENDPOINT` / `AGENT_HOST` | `agent.endpoint` |

### Hot-reload

Branding and `agent.endpoint` reload without restart. Security settings (`rate_limit`, `session`, `helmet`) require a server restart.

{{< info >}}
Full reference: [UI Configuration](/templates/ui/configuration/).
{{< /info >}}

## template-mcp-server: `.env`

| Variable | Default | Purpose |
|----------|---------|---------|
| `MCP_HOST` | `localhost` | Bind address |
| `MCP_PORT` | `5001` | Server port |
| `MCP_TRANSPORT_PROTOCOL` | `http` | `http`, `sse`, or `streamable-http` |
| `ENABLE_AUTH` | `False` in `.env.example` | OAuth resource server |
| `POSTGRES_*` | compose defaults | OAuth token storage when auth enabled |

See [template-mcp-server README](https://github.com/redhat-data-and-ai/template-mcp-server#configuration) and [Authentication guide](https://github.com/redhat-data-and-ai/template-mcp-server/blob/main/docs/authentication.md).

## Side-by-side comparison

| Concern | template-agent | template-ui | template-mcp-server |
|---------|----------------|-------------|---------------------|
| Prompts / routing | `PROMPT.md`, `subagents/` | N/A (proxies agent) | N/A |
| Tools / MCP | `mcp.json` + frontmatter `mcps` | MCP status UI only | Python tools in `src/tools/` |
| Model selection | Frontmatter `model` | N/A | N/A |
| Branding | N/A | `settings.yaml` `branding` | N/A |
| Auth | `ENABLE_AUTH`, `SSO_*` | `AUTH_ENABLED`, `SSO_*`, `COOKIE_SIGN` | `ENABLE_AUTH`, `SSO_*`, `SESSION_SECRET` |
| Agent URL | N/A (is the agent) | `AGENT_HOST` / `agent.endpoint` | N/A |
| Compliance | Guardrails, audit middleware | OPA `config/compliance/` | Tool-level validation |

## GitOps and OpenShift

- Mount `config/agent/` or subsets via ConfigMaps; secrets via Secrets → env vars
- Mount `config/ui/settings.yaml` as ConfigMap; SSO and cookie signing as Secrets
- Reference values: `config/agent/deployment/values.yaml` in template-agent

## Related documentation

- [Stack Architecture](/reference/architecture/)
- [Enterprise Features](/reference/enterprise-features/)
- [Migration Guide](/reference/migration-deep-agent/)
