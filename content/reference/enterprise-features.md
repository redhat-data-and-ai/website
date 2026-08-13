---
title: "Enterprise Features"
weight: 30
description: "Matrix of enterprise capabilities across template-agent, template-ui, and template-mcp-server."
---

This matrix maps enterprise features to the repository that implements them. Use it when planning security, observability, and compliance for a full-stack deployment.

## Feature matrix

| Feature | template-agent | template-ui | template-mcp-server |
|---------|----------------|-------------|---------------------|
| **SSO / OIDC** | `ENABLE_AUTH`, `SSO_*` env | `AUTH_ENABLED`, `SSO_*`, sessions | `ENABLE_AUTH`, `SSO_*`, `SESSION_SECRET` |
| **Langfuse tracing** | `LANGFUSE_*` env | — | — |
| **OpenTelemetry** | OTEL env, metrics endpoints | OTEL Fastify plugin | — (structlog JSON logging) |
| **User feedback API** | `/feedback` routes | Feedback UI buttons | — |
| **MCP SSO pass-through** | Forwards user Bearer token | — | Bearer introspection on `tools/call` |
| **MCP OAuth** | Per-user OAuth client flow, Redis tokens | MCP connect UI | OAuth authorization server (`/auth/*`), Postgres tokens |
| **MCP DCR** | Registers OAuth client to MCP | MCP status panel | `POST /auth/register` |
| **HITL interrupts** | Graph interrupt points | InterruptBanner UI | — |
| **Guardrails** | Middleware in agent runtime | — | — |
| **Audit logging** | Audit middleware + emitter | — | — |
| **PII scrubbing** | Detector/scrubber middleware | — | — |
| **OPA compliance** | — | `config/compliance/policy.rego` | — |
| **Rate limiting** | — | `settings.yaml` security section | — |
| **Session management** | Thread/checkpoint isolation | Redis + `COOKIE_SIGN` | PostgreSQL OAuth token storage (when auth enabled) |
| **Token budget** | `/threads/{id}/token-usage` | — | — |
| **Personalization** | Memory/rules API (evolving) | Settings UI (memory, rules) | — |
| **OpenShift overlays** | `deployment/overlays/openshift/` | `deployment/overlays/openshift/` | `deployment/openshift/` |
| **HPA / PDB** | OpenShift overlay | — | — |

## Authentication flows

### Agent SSO

When `ENABLE_AUTH=true` on template-agent, the LangGraph API expects authenticated requests. template-ui forwards the user session when configured as the BFF.

### UI SSO

template-ui uses OAuth2/OIDC with cookie-based sessions. Production requires `COOKIE_SIGN` (32+ characters) and aligned `SSO_CALLBACK_URL` with your Route hostname.

### MCP authentication

**Agent side** — defined per server in `config/agent/mcp.json`:

| `auth_mode` | Credential flow |
|-------------|-----------------|
| `sso` | Agent forwards user's SSO Bearer token to MCP |
| `oauth` | User connects via UI; agent stores tokens encrypted in Redis |
| `dcr` | Agent registers an OAuth client with MCP (`POST /auth/register`) |

Requires `MCP_TOKEN_ENCRYPTION_KEY` for OAuth/DCR on the agent in production.

**MCP server side** — when `ENABLE_AUTH=True` on template-mcp-server:

| Mode | `USE_EXTERNAL_BROWSER_AUTH` | Notes |
|------|----------------------------|-------|
| No auth | any (`ENABLE_AUTH=False`) | Default in `.env.example`; all endpoints open |
| Local dev auth | `True` | Browser OAuth; token cached in memory |
| Production auth | `False` | Full OAuth server; tokens in PostgreSQL |

`tools/list` is unauthenticated so agents can discover tools; `tools/call` requires a valid Bearer token when auth is enabled. See [Authentication](https://github.com/redhat-data-and-ai/template-mcp-server/blob/main/docs/authentication.md) in template-mcp-server.

## Observability stack

| Signal | Agent | UI |
|--------|-------|-----|
| Traces | Langfuse + OTEL | OTEL via Fastify |
| Metrics | OTEL exporters | — |
| Feedback | Langfuse + Postgres | UI → agent `/feedback` |
| Debug | Server logs, skills evals | Debug panel (dev) |

Local trace inspection: `make container` in template-agent can start Jaeger alongside the agent.

## Compliance

**template-agent:** Guardrails middleware, audit event emission (scrubbed), PII detection middleware. Configure in `runtime/agent.yaml`.

**template-ui:** OPA policies in `config/compliance/` deny requests based on Rego rules. See `config/compliance/README.md` in the repository.

## Security checklist (production)

- [ ] TLS on all Routes (`template-agent`, `template-ui`, MCP)
- [ ] Secrets in OpenShift Secrets, not ConfigMaps
- [ ] `AGENT_PUBLIC_BASE_URL` set for MCP OAuth callbacks
- [ ] `MCP_TOKEN_ENCRYPTION_KEY` rotated with `MCP_TOKEN_ENCRYPTION_KEY_PREVIOUS` during rotation
- [ ] `ENABLE_AUTH` / `AUTH_ENABLED` aligned across UI and agent
- [ ] OPA policies reviewed if `production.yaml` UI config is used
- [ ] Network policies restrict agent ↔ MCP ↔ database traffic

## Related documentation

- [Agent Deployment](/templates/agent/deployment/)
- [UI Deployment](/templates/ui/deployment/)
- [Config-as-code](/reference/config-as-code/)
- [MCP Server enterprise template](/templates/mcp-server/enterprise-mcp-template/)
