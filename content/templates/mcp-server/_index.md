---
title: "MCP Server Template"
weight: 10
description: "Build Model Context Protocol servers that extend AI agent capabilities with specialized tools, resources, and integrations."
github_repo: "https://github.com/redhat-data-and-ai/template-mcp-server"
---

Welcome to the MCP Server Developer documentation!

This section provides everything you need to design, develop, deploy, and maintain Model Context Protocol (MCP) servers. Whether you're building your first MCP server or deploying enterprise-ready solutions, you'll find comprehensive guidance here.

## Key Areas

- **[Quick Start](quick-start)** - Get your first MCP server running in minutes using the enterprise template
- **[Taking Your MCP Server to Production](taking-mcp-server-to-production)** - Deploy, monitor, and scale your MCP server
- **[Development Best Practices](best-practices)** - Proven patterns for maintainable, secure MCP servers
- **[Enterprise MCP Template](enterprise-mcp-template)** - Complete development tutorial and template documentation

## What is an MCP Server?

An MCP server is a specialized service that exposes **tools**, **resources**, and **prompts** to AI agents through the standardized Model Context Protocol. It acts as a bridge between AI agents and external systems, enabling agents to:

- **Execute functions** (tools) like database queries, API calls, or file operations
- **Access data** (resources) from files, databases, or external systems  
- **Use templates** (prompts) for consistent AI interactions

### Key Benefits

- **Standardized Integration**: Consistent interface for all AI agent interactions
- **Language Agnostic**: Build servers in Python, TypeScript, Rust, or any language
- **Secure**: Built-in authentication, authorization, and access controls
- **Scalable**: Handle multiple concurrent agent connections efficiently
- **Reusable**: One MCP server can serve multiple AI agents and applications

## MCP Server Architecture

{{< info >}}
**Logical Architecture**: This diagram shows the **logical MCP protocol architecture** with separate Tools, Resources, and Prompts registries. This remains accurate regardless of implementation approach - whether you implement each registry separately or use our **tools-first pattern** where resources and prompts are implemented as tools for universal client compatibility.
{{< /info >}}

{{< mermaid >}}
graph TB
    A[AI Agent] --> B[MCP Client]
    B --> C[MCP Protocol]
    C --> D[MCP Server]

    D --> E[Tools Registry]
    D --> F[Resources Registry]
    D --> G[Prompts Registry]

    E --> H[Database Tool]
    E --> I[API Tool]
    E --> J[File Tool]

    F --> K[Configuration Files]
    F --> L[Data Assets]

    G --> M[Query Templates]
    G --> N[Review Prompts]

    H --> O[External Database]
    I --> P[External API]
    J --> Q[File System]

    style D fill:#e1f5fe
    style E fill:#f3e5f5
    style F fill:#e8f5e8
    style G fill:#fff3e0
{{< /mermaid >}}

{{< tip >}}
**Implementation Note**: Our Enterprise MCP Template uses a **tools-first architecture** where traditional resources and prompts are implemented as tools. This provides universal client compatibility while maintaining the same logical architecture shown above.
{{< /tip >}}

## Platform Integration

The AI Templates ecosystem provides infrastructure for MCP server development and deployment:

### Development & Testing

- **Local Development**: Modern Python tooling with UV package manager
- **Enterprise Template**: Production-ready boilerplate with Kubernetes deployment
- **Comprehensive Testing**: Unit tests, integration tests, and container validation
- **Quality Assurance**: Pre-commit hooks, linting, and automated CI/CD pipelines

### Production Deployment

- **Kubernetes Platform**: Container orchestration with enterprise security
- **SSL/TLS**: Automatic certificate management and secure communications
- **Load Balancing**: High-availability deployment with automatic scaling
- **Health Monitoring**: Liveness and readiness probes with metrics collection

### Integration Ecosystem

- **External APIs**: Integration with web services and data sources
- **Databases**: Direct database connectivity for data operations
- **Source Control**: Git integration and CI/CD automation
- **Observability**: Performance monitoring for AI operations

## Development Approaches

### Learning Path (Recommended for New Developers)

1. **Start with [Quick Start](quick-start)** - Get your first MCP server running in minutes
2. **Apply [Best Practices](best-practices/)** - Learn proven patterns and security guidelines  
3. **Deploy with [Taking Your MCP Server to Production](taking-mcp-server-to-production/)** - Move to production
4. **Deep Dive with [Enterprise MCP Template](enterprise-mcp-template/)** - Master the fundamentals

### Fast Track (Experienced Developers)

1. **Use the Enterprise MCP Template** - Start with production-ready foundation (accessed via Quick Getting Started)
2. **Customize Tools** - Implement your specific business logic using tools-first architecture
3. **Deploy to Kubernetes** - Use included deployment manifests
4. **Monitor & Scale** - Leverage built-in observability and scaling

## Target Use Cases

MCP servers excel at providing AI agents with:

### **Data Access & Analytics**

- Database querying and data retrieval
- Real-time analytics and reporting
- Data validation and quality checks

### **System Integration**

- API orchestration and workflow automation
- File system operations and content management
- Configuration management and deployment automation

### **Domain Expertise**

- Business rule enforcement and validation
- Specialized calculations and processing
- Industry-specific knowledge and templates

## Overview

As an MCP Server Developer, you will typically be involved in:

- **Tool Development**: Creating custom tools that expose specific functionality to AI agents
- **Resource Management**: Implementing secure access to data sources, files, and external systems
- **Prompt Engineering**: Designing reusable prompt templates for consistent AI interactions
- **Architecture Design**: Structuring servers for scalability, security, and maintainability
- **Production Deployment**: Using enterprise templates and Kubernetes for reliable production hosting
- **Performance Optimization**: Implementing caching, connection pooling, and efficient data access patterns
- **Security Implementation**: Ensuring proper authentication, authorization, and data protection

## Getting Started

Ready to build your first MCP server? Choose your path:

{{< tip >}}
**New to MCP?** Start with [Quick Start](quick-start) to get your first server running in minutes, then explore the [Enterprise MCP Template](enterprise-mcp-template) for deep understanding.

**Need production deployment?** Start with [Quick Start](quick-start) to access the Enterprise MCP Template, then follow [Taking Your MCP Server to Production](taking-mcp-server-to-production) for deployment guidance.
{{< /tip >}}

### Quick Links

- **[Get started in minutes](quick-start)** with the tools-first template
- **[Learn fundamentals](enterprise-mcp-template)** with the comprehensive template documentation
- **[Deploy at scale](taking-mcp-server-to-production/)** using OpenShift manifests
- **[Stack reference](/reference/architecture/)** — how MCP fits with agent and UI on ports 5001 → 5002 → 5173
- **[Agentic AI Workshop](/guides/agentic-ai-workshop/)** — run all three templates together

For questions and support, visit our [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).

