---
title: "Deployment Guide"
weight: 10
description: "Deploy template-ui with Podman, OpenShift Kustomize overlays, or Kind for production chat against Deep Agent."
---

This guide covers deploying **template-ui** to production, including the Fastify BFF, Redis sessions, SSO, and connection to template-agent.

## Prerequisites

- template-agent deployed and reachable (see [Agent Deployment](/templates/agent/deployment/))
- Container registry access
- OpenShift or Kubernetes cluster (or Podman for single-container runs)
- SSO provider configured (if `auth_enabled: true`)
- Redis available for production sessions (compose uses port **6380** locally)

## Build for production

```bash
npm run build
```

Output:

- `dist/frontend/` — built React application
- `dist/server/` — compiled Fastify BFF

Run locally:

```bash
npm start
```

Serves API and static frontend on `PORT` (default **8080**).

## Container deployment

```bash
podman build -t your-registry/template-ui:latest .
podman run -p 8080:8080 --env-file .env your-registry/template-ui:latest
```

The `Containerfile` uses a Red Hat UBI base image.

## Local compose (UI + Redis)

```bash
# compose.yml runs UI and Redis together
podman-compose up
```

Redis supports session storage when `AUTH_ENABLED=true`.

## OpenShift deployment

Kustomize overlays live in `deployment/overlays/openshift/`:

- BuildConfig, ImageStream, Route
- ConfigMap for `settings.yaml`
- Secret references for SSO and `COOKIE_SIGN`

Base manifests are in `deployment/base/`. Apply the overlay appropriate to your cluster namespace.

### Production settings checklist

| Setting | Notes |
|---------|--------|
| `AGENT_ENDPOINT` or `settings.yaml` `agent.endpoint` | HTTPS URL to template-agent Route |
| `FEATURE_AUTH_ENABLED=true` | Enable SSO in production |
| `COOKIE_SIGN` | Strong secret in OpenShift Secret |
| `SSO_*` | OIDC client aligned with Route URL |
| `config/ui/settings.yaml` | Branding via ConfigMap mount |
| `config/compliance/` | OPA policies for regulated environments |

## Kind (local Kubernetes)

```bash
# See deployment/overlays/kind/ in the repository
```

NodePort overlay for local cluster testing without OpenShift Routes.

## Environment configuration

### Agent connection

Point the BFF at your deployed template-agent:

```bash
AGENT_ENDPOINT=https://agent.apps.example.com
```

Or mount `config/ui/settings.yaml`:

```yaml
agent:
  endpoint: "https://agent.apps.example.com"
  timeout_ms: 60000
  streaming: true
```

### Authentication

**Development:** `AUTH_ENABLED=false` — dummy user, no SSO.

**Production:** `AUTH_ENABLED=true` with:

```bash
SSO_CLIENT_ID=...
SSO_CLIENT_SECRET=...
SSO_ISSUER_HOST=https://your-idp.example.com
SSO_CALLBACK_URL=https://chat.apps.example.com/auth/callback/oidc
```

### Security

- Use **Helmet** and **rate limiting** settings in `settings.yaml` (production example config)
- Store secrets in OpenShift Secrets, not ConfigMaps
- Enable **OPA compliance** policies when required (`config/compliance/policy.rego`)

## Observability

- **OpenTelemetry** tracing via Fastify plugins
- **`/version`** endpoint for build identification
- Correlate UI requests with agent traces via shared request IDs (when OTEL enabled on both services)

## Stack deployment order

1. **template-mcp-server** — tools (`:5001` or cluster Service)
2. **template-agent** — Deep Agent API (`:5002` or Route)
3. **template-ui** — chat BFF pointing `AGENT_ENDPOINT` at the agent

Verify end-to-end after each layer:

```bash
curl https://agent.apps.example.com/health
curl https://chat.apps.example.com/version
```

## Scaling

- **UI replicas** — stateless BFF; scale horizontally behind a Route/Ingress
- **Redis** — use a managed Redis service for shared sessions across replicas
- **Agent** — scale template-agent independently; UI proxies to a single agent Service endpoint

## Next Steps

- [Configuration](/templates/ui/configuration/) — branding and feature flags
- [Agent Deployment](/templates/agent/deployment/) — deploy the Deep Agent backend
- [GitHub Issues](https://github.com/redhat-data-and-ai/template-ui/issues)

For community discussion, visit [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).
