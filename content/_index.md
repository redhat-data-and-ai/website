---
title: "AI Templates"
---

# Build production-ready AI applications

Go beyond demos and proofs of concept. AI Templates deliver battle-tested, production-ready components for building AI applications on Kubernetes, following best practices.

{{< info title="What's New" >}}
**Deep Agent update (August 2026):** The Agent and UI templates have moved to [LangGraph Deep Agents](https://github.com/langchain-ai/deepagents). **template-agent** is now a config-as-code Deep Agent runtime (orchestrator, subagents, skills, MCP integration). **template-ui** has been updated to work with the new Deep Agent streaming API, including HITL interrupts, task progress, and markdown-first rendering. [Read the announcement →](/news/)
{{< /info >}}

## What are AI Templates?

AI Templates are reusable, modular components that accelerate your AI development journey. Each template is designed to work independently or as part of a complete AI application stack:

- **[MCP Server Template](/templates/mcp-server/)** - Build Model Context Protocol servers that extend AI agent capabilities
- **[Agent Template](/templates/agent/)** - Deep Agent orchestration with config-as-code, subagents, skills, and enterprise observability
- **[UI Template](/templates/ui/)** - React chat UI for the Deep Agent streaming API

## Why Use AI Templates?

**Fast Development**  
Get from idea to production in hours, not weeks. Our templates handle the infrastructure complexity.

**Enterprise-Ready**  
Built-in security, authentication, observability, and audit trails. Production-ready from day one.

**Modular Architecture**  
Use what you need. Mix and match templates or integrate with your existing systems.

**Kubernetes-Native**  
Deploy anywhere - OpenShift, EKS, GKE, or any Kubernetes cluster.

## Get Started

1. **[Choose a Template](/templates/)** - Browse our template catalog
2. **[Set Up Your Tools](/tools/)** - Configure Cursor IDE or Claude Desktop
3. **[Read the Documentation](/templates/)** - Follow quick start guides for each template

## Template Ecosystem

{{< mermaid >}}
graph TD
    A[User Interface<br/>template-ui BFF] --> B[AI Deep Agent Layer<br/>template-agent]
    B --> C[MCP Server<br/>template-mcp-server]
    C --> D[External Systems<br/>Databases, APIs, Services]

    style A fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style B fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    style C fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    style D fill:#fff3e0,stroke:#ff9800,stroke-width:2px
{{< /mermaid >}}

## Open Source & Community

All templates are open source and available on [GitHub](https://github.com/redhat-data-and-ai). Contributions are welcome!

**License:** Apache 2.0  
**Community:** [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions)  
**Issues:** Report issues in individual template repositories
