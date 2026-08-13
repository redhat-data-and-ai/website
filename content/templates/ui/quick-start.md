---
title: "Quick Start"
weight: 5
description: "Run template-ui locally with npm run dev, connect to template-agent on port 5002, and open the chat UI."
---

Get the chat UI running against a local Deep Agent. This guide assumes you will run template-agent separately.

## Prerequisites

- **Node.js 22+** and **npm 8+**
- **[template-agent](/templates/agent/quick-start/)** running on **http://localhost:5002**
- **Optional:** [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server) on `:5001` for tool-enabled agent workflows

## Step 1: Start the Agent

In a separate terminal, start template-agent:

```bash
git clone https://github.com/redhat-data-and-ai/template-agent.git
cd template-agent
make install
make local
curl http://localhost:5002/health
```

## Step 2: Clone and Configure the UI

```bash
git clone https://github.com/redhat-data-and-ai/template-ui.git
cd template-ui
cp env.template .env
```

Edit `.env` for local development:

```bash
PORT=8080
ENVIRONMENT=development
AUTH_ENABLED=false
AGENT_HOST=http://localhost:5002
COOKIE_SIGN=your-secret-with-minimum-length-of-32-characters
```

{{< info >}}
`AUTH_ENABLED=false` uses a dummy development user so you can test without SSO credentials.
{{< /info >}}

### Optional: runtime settings file

For branding and feature flags without code changes:

```bash
cp config/ui/examples/minimal.yaml config/ui/settings.yaml
```

See [Configuration](/templates/ui/configuration/) for the full schema.

## Step 3: Install and Run

```bash
npm install
npm run dev
```

Or use the Makefile shortcut:

```bash
make dev
```

The app is available at **http://localhost:5173** (Vite dev server). The Fastify BFF runs as part of `npm run dev` and proxies to `AGENT_HOST`.

## Step 4: Test the Chat

1. Open **http://localhost:5173** in your browser
2. Send a message in the chat interface
3. Confirm streaming responses appear (word-by-word or chunked)
4. If the agent has MCP tools configured, ask what tools are available

## Step 5: Verify the Stack

| Service | URL | Check |
|---------|-----|-------|
| template-agent | http://localhost:5002/health | `curl` returns OK |
| template-mcp-server | http://localhost:5001/health | Optional, for tools |
| template-ui | http://localhost:5173 | Chat loads and streams |

## Step 6: Production Build (Optional)

```bash
npm run build
npm start
```

Production mode serves the built React app and API from the Fastify server (default port **8080**).

## Next Steps

- **Customize branding** — [Configuration](/templates/ui/configuration/)
- **Deploy** — [Deployment Guide](/templates/ui/deployment/)
- **Read agent docs** — [Agent Architecture](/templates/agent/architecture/)

{{< tip >}}
Use the debug panel (enabled in `minimal.yaml` example config) to inspect raw stream events during development.
{{< /tip >}}

## Troubleshooting

**UI loads but chat does not respond**
- Confirm template-agent is running: `curl http://localhost:5002/health`
- Check `AGENT_HOST` in `.env` points to `:5002`
- Review the terminal running `npm run dev` for proxy errors

**Authentication errors in development**
- Set `AUTH_ENABLED=false` in `.env`
- Ensure `COOKIE_SIGN` is at least 32 characters

**Port conflicts**
```bash
lsof -ti:5173 | xargs kill -9
lsof -ti:8080 | xargs kill -9
```

For more help, visit [GitHub Issues](https://github.com/redhat-data-and-ai/template-ui/issues).
