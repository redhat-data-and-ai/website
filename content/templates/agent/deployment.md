---
title: "Deployment Guide"
weight: 10
description: "Deploy the Deep Agent template with Podman Compose, OpenShift overlays, or Kind for production and local Kubernetes testing."
---

This guide covers deploying **template-agent** from local containers to OpenShift, including dependencies, secrets, and production hardening.

## Prerequisites

Before deploying to production:

- Agent tested locally with `make local`
- Google Vertex AI credentials (or configured vLLM/MaaS endpoint)
- PostgreSQL and Redis available (managed services or cluster components)
- MCP servers deployed and reachable from the agent namespace
- Container registry access (OpenShift ImageStream or external registry)

## Local container stack

Run the full stack (Postgres, Redis, agent, optional Jaeger) in containers:

```bash
make container
```

- Agent: **http://localhost:5002**
- Jaeger UI (if enabled): **http://localhost:16686**

Stop the stack:

```bash
make container-down
```

For detached development with log tailing:

```bash
make dev        # start stack in background, tail logs
make dev-down   # stop stack
make dev-clean  # stop and remove volumes
```

## Build the container image

The repository includes a `Containerfile` (Red Hat UBI-based). Build with Podman:

```bash
podman build -t your-registry/template-agent:latest .
podman push your-registry/template-agent:latest
```

Or use the OpenShift `BuildConfig` in `deployment/overlays/openshift/buildconfig.yaml`.

## OpenShift deployment

Manifests live in `deployment/overlays/openshift/` and include:

- Deployment, Service, Route
- ImageStream and BuildConfig
- ConfigMap and Secret patches
- Redis component patch
- HPA and PodDisruptionBudget

Deploy with the repository Makefile:

```bash
make deploy openshift NAMESPACE=your-project
```

This applies Kustomize overlays, substitutes the namespace, and creates required secrets from your `.env` file.

### Production configuration

Set these before or during deployment:

| Setting | Purpose |
|---------|---------|
| `GOOGLE_APPLICATION_CREDENTIALS_CONTENT` | Vertex AI credentials (Secret) |
| `POSTGRES_*` | Managed or in-cluster PostgreSQL |
| `REDIS_URL` | Managed or in-cluster Redis |
| `AGENT_PUBLIC_BASE_URL` | Public HTTPS URL for OAuth MCP callbacks |
| `ENABLE_AUTH` | Enable SSO/OIDC for authenticated access |
| `LANGFUSE_*` | Optional tracing in production |
| `SSL_KEYFILE` / `SSL_CERTFILE` | TLS termination (if not handled by Route) |
| `MCP_TOKEN_ENCRYPTION_KEY` | Required for OAuth/DCR MCP in production |

Reference values for GitOps/ArgoCD are in `config/agent/deployment/values.yaml`.

### Health checks

The agent exposes standard probes on port **5002**:

```bash
curl http://localhost:5002/health
curl http://localhost:5002/readyz
```

OpenShift Route or Ingress should target the agent Service on this port.

## Kind (local Kubernetes)

For full-stack Kubernetes testing without OpenShift:

```bash
make kind
```

See `deployment/overlays/kind/README.md` in the repository for cluster setup details.

## Custom CA certificates

Corporate CA trust without rebuilding the image:

**Compose** — set in `.env`:

```bash
CUSTOM_CA_FILE=./certs/ca.pem
```

**Kubernetes** — mount a Secret and set `CUSTOM_CA_PATH=/etc/custom-ca/ca.pem`

**Standalone container** — set `CUSTOM_CA_URL` to download a PEM at startup.

## Security considerations

- **Secrets** — store credentials in OpenShift Secrets, not ConfigMaps
- **Network policies** — restrict agent-to-MCP and agent-to-database traffic
- **SSO** — enable `ENABLE_AUTH` and configure `SSO_*` for multi-user deployments
- **MCP tokens** — use `MCP_TOKEN_ENCRYPTION_KEY` and rotate with `MCP_TOKEN_ENCRYPTION_KEY_PREVIOUS`
- **Audit** — middleware emits scrubbed audit events; configure retention per your compliance requirements
- **TLS** — terminate at Route/Ingress or configure `SSL_KEYFILE` and `SSL_CERTFILE`

## Observability

**Langfuse** — set `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, and `LANGFUSE_BASE_URL` for trace and feedback correlation.

**OpenTelemetry** — enable with `ENABLE_OTEL` and exporter endpoints. The `make container` target can start Jaeger for local trace inspection.

**Token usage** — query `/threads/{thread_id}/token-usage` for per-conversation consumption.

## Scaling

The OpenShift overlay includes an **HPA** (Horizontal Pod Autoscaler) and **PDB** (Pod Disruption Budget). Tune CPU/memory requests in `deployment/overlays/openshift/deployment.yaml` based on model latency and context window size.

Considerations:

- **PostgreSQL** — shared checkpoint store; size connection pools for replica count
- **Redis** — required for SSE broker and OAuth tokens; use a managed Redis service in production
- **Memory** — increase limits for large context windows or multiple concurrent streams

## Stack deployment order

For a full AI Templates deployment:

1. **template-mcp-server** — tools on `:5001` (or cluster Service)
2. **template-agent** — Deep Agent on `:5002`
3. **template-ui** — chat UI pointing `AGENT_HOST` or `settings.yaml` agent endpoint at the agent Route

## Next Steps

- [Architecture](/templates/agent/architecture/) — config and API reference
- [UI Deployment](/templates/ui/deployment/) — deploy the chat front end
- [GitHub Issues](https://github.com/redhat-data-and-ai/template-agent/issues) — deployment questions

For community discussion, visit [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).
