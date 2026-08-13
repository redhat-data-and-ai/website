---
title: "Quick Start"
weight: 5
description: "Run the Deep Agent template locally with make install and make local, then test the LangGraph API on port 5002."
---

Get the Deep Agent runtime running locally. This guide covers installation, credentials, infrastructure dependencies, and a first API call.

## Prerequisites

Before you begin, ensure you have:

- **Python 3.13+**
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** package manager
- **[Podman](https://podman.io/)** and **podman-compose** (for Postgres and Redis)
- **Google Vertex AI credentials** — service account JSON for Gemini models
- **Optional:** [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server) on `:5001` for tool calls

## Step 1: Clone and Install

```bash
git clone https://github.com/redhat-data-and-ai/template-agent.git
cd template-agent
make install
```

`make install` creates a `.venv`, installs dependencies with dev extras, and sets up pre-commit hooks.

## Step 2: Configure Environment

Copy the example environment file and set your Vertex AI credentials:

```bash
cp .env.example .env
```

Edit `.env` and set `GOOGLE_APPLICATION_CREDENTIALS_CONTENT` to your Google service account JSON (single-line or as documented in `.env.example`).

{{< info >}}
`make local` creates `.env` from `.env.example` automatically if the file does not exist.
{{< /info >}}

## Step 3: Start the Agent

```bash
make local
```

This command:

1. Starts **PostgreSQL** (pgvector) and **Redis** via Podman Compose
2. Waits for the database to be ready
3. Launches the Aegra dev server on **http://localhost:5002**

Verify in another terminal:

```bash
curl http://localhost:5002/health
```

To stop only the agent process, press Ctrl+C. Postgres and Redis keep running until you run `make local-down`.

## Step 4: Connect an MCP Server (Optional)

MCP servers are defined in `config/agent/mcp.json`. For local development with [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server):

```bash
# In a separate terminal
git clone https://github.com/redhat-data-and-ai/template-mcp-server.git
cd template-mcp-server
make local
curl http://localhost:5001/health
```

The default `mcp.json` URL for `make local` is `http://localhost:5001/mcp`.

If you do not have an MCP server yet, the agent still starts — but tool-dependent workflows need MCP configured. See the [MCP Server Quick Start](/templates/mcp-server/quick-start/).

## Step 5: Test the LangGraph API

Create a conversation thread:

```bash
curl -X POST http://localhost:5002/threads \
  -H "Content-Type: application/json" \
  -d '{}'
```

Stream a message (replace `THREAD_ID` with the ID from the response):

```bash
curl -N -X POST "http://localhost:5002/threads/THREAD_ID/runs/stream" \
  -H "Content-Type: application/json" \
  -d '{
    "assistant_id": "agent",
    "input": {"messages": [{"role": "human", "content": "Hello"}]},
    "stream_mode": "updates"
  }'
```

The assistant ID `agent` is defined in `aegra.json`.

## Step 6: Customize via Config

Most changes do not require Python edits:

1. **Orchestrator prompt** — edit `config/agent/PROMPT.md`
2. **Subagents** — add or modify files in `config/agent/subagents/`
3. **Skills** — edit documents in `config/agent/skills/`
4. **MCP servers** — update `config/agent/mcp.json`
5. **Runtime settings** — adjust `config/agent/runtime/agent.yaml`

See [Architecture](/templates/agent/architecture/) for the full config layout.

## Step 7: Run Tests

```bash
make test           # unit tests
make test-all       # unit + skills evaluations
```

## Next Steps

- **Add a chat UI** — [UI Template Quick Start](/templates/ui/quick-start/)
- **Deploy to OpenShift** — [Deployment Guide](/templates/agent/deployment/)
- **Read the architecture** — [Architecture](/templates/agent/architecture/)

{{< tip >}}
Use `make mock-mcp` in a second terminal if you want a local MCP stub without cloning template-mcp-server. See `make local-with-mock` in the repository Makefile for the two-terminal workflow.
{{< /tip >}}

## Troubleshooting

**Agent won't start**
- Confirm `GOOGLE_APPLICATION_CREDENTIALS_CONTENT` is set in `.env`
- Check port 5002 is free: `lsof -i :5002`
- Ensure Podman is running and Postgres/Redis containers started

**Database connection errors**
- Wait for Postgres readiness ( `make local` does this automatically)
- Run `make local-down` and retry `make local`

**MCP tools not available**
- Verify MCP server is running: `curl http://localhost:5001/health`
- Check `config/agent/mcp.json` URL matches your run mode
- Confirm the MCP name in subagent frontmatter exists in `mcp.json` with `enabled: true`

For more help, visit [GitHub Issues](https://github.com/redhat-data-and-ai/template-agent/issues).
