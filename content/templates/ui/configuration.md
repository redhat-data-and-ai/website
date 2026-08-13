---
title: "Configuration"
weight: 8
description: "Runtime branding, feature flags, agent endpoint, and environment overrides for template-ui without rebuilding."
---

template-ui supports **runtime configuration** through `config/ui/settings.yaml` and environment variables. Most branding and agent-endpoint changes apply without a rebuild.

## Configuration sources

| Source | Purpose | Precedence |
|--------|---------|------------|
| `config/ui/settings.yaml` | Branding, features, agent endpoint, security defaults | Base |
| Environment variables | Overrides for deployment (OpenShift Secrets, `.env`) | Higher |
| Built-in defaults | Used when `settings.yaml` is absent | Fallback |

Agent endpoint resolution:

```
config/ui/settings.yaml → agent.endpoint
        ↓ (if empty)
AGENT_HOST or AGENT_ENDPOINT env var
        ↓ (if empty)
http://localhost:5002
```

## settings.yaml structure

`config/ui/settings.yaml` is optional. Copy an example to get started:

```bash
# Local development (auth off, debug panel on)
cp config/ui/examples/minimal.yaml config/ui/settings.yaml

# Red Hat branding
cp config/ui/examples/red-hat-branding.yaml config/ui/settings.yaml

# Production-hardened (OPA enabled)
cp config/ui/examples/production.yaml config/ui/settings.yaml
```

### Branding

```yaml
branding:
  title: "My Custom Agent"
  logo_url: "/custom-logo.svg"
  favicon_url: "/custom-favicon.ico"
  colors:
    light:
      primary: "#0066cc"
      accent: "#a60000"
      background: "#ffffff"
      foreground: "#1a1a1a"
    dark:
      primary: "#4dabf7"
      accent: "#f56e6e"
      background: "#0a1628"
      foreground: "#f0f4f8"
```

Place custom assets in `public/` and reference them by URL path.

### Features

```yaml
features:
  debug_mode_default: false
  auth_enabled: true
```

Set `auth_enabled: false` for local development without SSO. Maps to `FEATURE_AUTH_ENABLED` env override.

### Agent connection

```yaml
agent:
  endpoint: ""
  timeout_ms: 30000
  streaming: true
```

Leave `endpoint` empty to use `AGENT_HOST` from `.env` (default `http://localhost:5002`).

## Environment variable overrides

Environment variables take precedence over YAML:

```bash
BRANDING_TITLE="Production Agent"
BRANDING_LOGO_URL="/prod-logo.svg"
FEATURE_AUTH_ENABLED=true
AGENT_ENDPOINT=https://prod-agent.example.com
```

Common variables (see `env.template` for the full list):

| Variable | Description |
|----------|-------------|
| `PORT` | Server port (default `8080`) |
| `ENVIRONMENT` | `development` \| `production` \| `test` |
| `AUTH_ENABLED` | Enable SSO authentication |
| `AGENT_HOST` | template-agent URL |
| `COOKIE_SIGN` | Session cookie signing secret (32+ chars) |
| `SSO_CLIENT_ID` / `SSO_CLIENT_SECRET` | OIDC credentials |
| `SSO_ISSUER_HOST` | OIDC issuer URL |
| `SSO_CALLBACK_URL` | OAuth redirect URI |

## Hot-reload behavior

| Category | Hot-reload? |
|----------|-------------|
| Branding (title, logo, colors) | Yes |
| `agent.endpoint`, `agent.timeout_ms` | Yes |
| Security (`rate_limit`, `session`, `helmet`) | No — requires restart |

The config watcher logs which category changed when you edit `settings.yaml` during development.

## Validation

Invalid configuration fails at startup with a clear error:

```text
Config validation error: branding.colors.light.primary must be a valid hex color (got 'not-a-color')
```

## OPA compliance policies

Production deployments can enforce UI compliance rules with Open Policy Agent:

```
config/compliance/
├── policy.rego    # Deny rules (package compliance.ui)
├── data.json      # Static data for policies
└── README.md      # Policy authoring guide
```

See the repository's `config/compliance/README.md` for built-in rules and testing with `opa eval`.

## BFF proxy behavior

The Fastify server in `src/server/router/proxy.router.ts` proxies LangGraph SSE from template-agent and translates events for React components:

- Subagent indicators and task rewriting
- HITL interrupt banners
- Todo/task progress and sidebar
- Artifact viewer payloads
- MCP connection status
- Intermediate streaming responses

The frontend does not implement agent logic—it renders events from the BFF.

## Next Steps

- [Quick Start](/templates/ui/quick-start/) — run locally
- [Deployment Guide](/templates/ui/deployment/) — OpenShift and production
- [Repository `config/ui/README.md`](https://github.com/redhat-data-and-ai/template-ui/blob/main/config/ui/README.md) — full schema reference
