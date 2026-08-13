---
title: "Reference"
description: "Architecture, configuration, enterprise features, and migration guides for the AI Templates stack."
---

Technical reference for the three-repo AI Templates stack after the Deep Agent merge (August 2026).

## Reference guides

| Guide | Description |
|-------|-------------|
| [Stack architecture](/reference/architecture/) | Full-stack diagram, ports, and request flow |
| [Config-as-code](/reference/config-as-code/) | `config/agent/` vs `config/ui/settings.yaml` |
| [Enterprise features](/reference/enterprise-features/) | Langfuse, OTEL, SSO, MCP auth, OPA, HITL — by repo |
| [Migration from pre-Deep Agent](/reference/migration-deep-agent/) | Breaking changes checklist for adopters |

## Template documentation

For hands-on setup, see the template sections:

- [MCP Server Template](/templates/mcp-server/)
- [Agent Template](/templates/agent/)
- [UI Template](/templates/ui/)

## Workshop

- [Zero to Production: Agentic AI Onboarding](/guides/agentic-ai-workshop/) — end-to-end local stack walkthrough

## Announcements

- [Deep Agent merge](/news/deep-agent-merge/)
- [Config-as-code architecture](/news/config-as-code-architecture/)
- [template-ui update](/news/template-ui-deep-agent-update/)

## External resources

AI Templates builds on open specifications and frameworks. These upstream docs explain concepts used throughout this site:

| Resource | What it covers |
|----------|----------------|
| [LangGraph Deep Agents](https://github.com/langchain-ai/deepagents) | Deep Agent orchestration model used by template-agent |
| [LangGraph documentation](https://langchain-ai.github.io/langgraph/) | Graph runtime, streaming, checkpoints, LangGraph API |
| [Agent Skills specification](https://agentskills.io/specification) | `SKILL.md` format for skills in `config/agent/skills/` |
| [Model Context Protocol](https://modelcontextprotocol.io/) | MCP tools and server integration (template-mcp-server) |
| [Langfuse](https://langfuse.com/docs) | Optional tracing and feedback (template-agent) |

Our template repositories remain the source of truth for **how** these are wired in this stack—ports, config paths, and deployment overlays are documented in the guides above.

For questions, visit [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).
