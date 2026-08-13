---
title: "MCP Template Deep Dive"
weight: 6
description: "A comprehensive exploration of the enterprise MCP template architecture, directory structure, and design decisions including FastAPI vs FastMCP, containerization, and testing strategies."
---

This deep dive explores the enterprise MCP template's architecture, walking through each directory and file to understand the design decisions that make it production-ready. You'll learn how **FastAPI and FastMCP work together**, how the modular structure supports enterprise deployments, and the reasoning behind each component.

## Template Structure Overview

```text
template-mcp-server/
├── template_mcp_server/              # Main package (nested structure)
│   ├── src/                         # Source code
│   │   ├── __init__.py
│   │   ├── main.py                  # Application entry point
│   │   ├── api.py                   # FastAPI application
│   │   ├── mcp.py                   # MCP server integration
│   │   ├── settings.py              # Configuration management
│   │   └── tools/                   # MCP tools (tools-first architecture)
│   │       ├── multiply_tool.py    # Example CPU-bound tool
│   │       ├── code_review_tool.py # Example I/O-bound tool
│   │       └── redhat_logo_tool.py # Example resource-as-tool
│   └── utils/
│       └── pylogger.py              # Structured logging
├── deployment/openshift/            # OpenShift / Kustomize manifests
├── tests/                           # Comprehensive test suite
├── examples/                        # Usage examples
├── pyproject.toml                   # Python project configuration
├── Containerfile                    # Container build instructions
└── README.md
```

{{< info >}}
**Repository**: [template-mcp-server](https://github.com/redhat-data-and-ai/template-mcp-server)

This template implements a "tools-first" architecture where **all functionality** is exposed as MCP tools.
{{< /info >}}

## Key Architectural Decisions

### FastAPI + FastMCP: Why both?

The template uses **FastAPI** as the HTTP application layer and **FastMCP** for MCP protocol handling. They are complementary, not alternatives:

#### Combined stack

- **FastAPI** — health checks, OAuth routes, middleware, OpenAPI docs, production HTTP serving
- **FastMCP** — MCP tool registration, JSON-RPC `/mcp` endpoint, transport integration
- **Production scale** — async/await for concurrent connections
- **Enterprise security** — OAuth2, CORS, rate limiting, input validation via FastAPI middleware
- **Transport flexibility** — HTTP, SSE, and streamable-HTTP (`MCP_TRANSPORT_PROTOCOL`)
- **Type safety** — Pydantic settings and tool schemas

#### FastMCP alone

FastMCP is simpler for minimal MCP servers but offers fewer built-in enterprise HTTP and auth integrations. This template wraps FastMCP inside FastAPI for production deployments.

```python
# FastAPI + FastMCP (template uses both)
from fastapi import FastAPI
from fastmcp import FastMCP

app = FastAPI(title="Enterprise MCP Server")
mcp = FastMCP("enterprise-server")

@app.get("/health")
async def health():
    return {"status": "healthy"}
```

### Nested Package Structure

The template uses `template_mcp_server/src/` instead of flat `src/` for enterprise benefits:

#### Container Isolation

- **Clean builds**: Only the package gets copied to containers
- **Layer optimization**: Better container layer caching and smaller images
- **Security**: Excludes development artifacts from production deployments

#### Package Management

- **Namespace protection**: Prevents naming conflicts
- **Import clarity**: Clear package imports
- **Distribution ready**: Proper structure for package repositories

## When to Use This Template

Use the Enterprise MCP Template when you need:

- **Production deployment** on Kubernetes
- **Enterprise security** and compliance requirements
- **Comprehensive testing** and CI/CD pipelines
- **Modular architecture** for complex integrations
- **Multiple transport protocols** (HTTP, SSE, Streamable-HTTP)
- **Professional development workflow** with pre-commit hooks

## Architecture

{{< mermaid >}}
graph TD
    A[Client Request] --> B[FastAPI App]
    B --> C{Transport Protocol}
    C -->|HTTP| D[HTTP Handler]
    C -->|SSE| E[SSE Handler]

    D --> G[MCP Server]
    E --> G

    G --> H[Tools Registry]
    H --> I[CPU-Bound Tools]
    H --> J[I/O-Bound Tools]
    H --> K[Resource-as-Tools]

    I --> L["multiply()<br/>(def)"]
    J --> M["code_review()<br/>(async def)"]
    K --> N["get_redhat_logo()<br/>(async def)"]

    style G fill:#e1f5fe
    style H fill:#f3e5f5
{{< /mermaid >}}

## def vs async def: The Complete Guide

One of the most critical decisions in tool development is choosing between synchronous (`def`) and asynchronous (`async def`) functions.

### When to Use def (Synchronous)

**✅ CPU-Bound Operations:**
- Mathematical calculations
- Data processing and transformations
- Algorithms and sorting
- In-memory operations

```python
def multiply_numbers(a: float, b: float) -> dict:
    """CPU-bound calculation - uses def (synchronous)."""
    try:
        result = a * b
        return {"status": "success", "result": result}
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

### When to Use async def (Asynchronous)

**✅ I/O-Bound Operations:**
- Database queries
- HTTP API calls
- File uploads/downloads
- Network operations

```python
async def code_review_tool(code: str, language: str = "python") -> dict:
    """I/O-bound operation - uses async def for external API calls."""
    try:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "https://ai-review-service.example.com/analyze",
                json={"code": code, "language": language},
                timeout=30.0
            )
            analysis = response.json()

        return {"status": "success", "analysis": analysis}
    except httpx.TimeoutException:
        return {"status": "error", "message": "AI service timeout"}
```

### Decision Matrix

| Operation Type | Example | Use | Reason |
|---------------|---------|-----|---------|
| **Math/Logic** | Calculations, sorting | `def` | CPU-bound, no I/O benefit |
| **Database** | SQL queries | `async def` | I/O-bound, wait for network |
| **HTTP Calls** | API requests | `async def` | I/O-bound, network latency |
| **File I/O** | Reading files | `async def` | I/O-bound, disk operations |
| **External Services** | AI APIs, cloud services | `async def` | I/O-bound, service latency |

{{< tip >}}
**Rule of Thumb**: If your tool waits for something external (network, disk, database), use `async def`. If it only uses CPU and memory, use `def`.
{{< /tip >}}

## Testing Strategy

The template includes comprehensive tests across multiple categories:

### Test Categories

- **test_api.py**: FastAPI integration tests
- **test_container.py**: Kubernetes deployment tests  
- **test_mcp.py**: MCP server and tools-first architecture tests
- **test_tools.py**: Individual tool functionality tests
- **test_settings.py**: Configuration validation tests

### Example Test

```python
def test_container_startup_and_health(self):
    """Test that container starts and responds to HTTP requests."""
    build_cmd = ["podman", "build", "-t", image_name, "."]
    run_cmd = ["podman", "run", "-d", "--name", container_name,
               "-p", "5001:5001", image_name]

    # Build, run, test, cleanup
    assert build_result.returncode == 0
    assert health_response.status_code == 200
```

## Kubernetes Deployment

Production-ready Kubernetes configurations:

```yaml
# deployment.yaml - Core deployment
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: mcp-server
        image: "registry.example.com/mcp-server:latest"
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

**Production Features:**

- **High availability**: Multi-replica deployment
- **Resource management**: CPU/memory limits and requests
- **Health checks**: Liveness and readiness probes
- **Security**: Pod security contexts and service accounts

## Getting Started with the Template

{{< info >}}
**Ready to Start Building?**

For step-by-step setup instructions, transformation guide, and your first tool implementation, see the [Quick Start](quick-start) guide.

This Deep Dive focuses on understanding the **architecture** and **design decisions** behind the template.
{{< /info >}}

## Summary

This template represents a production-ready foundation for enterprise MCP servers, carefully designed around three core principles:

1. **Tools-First Architecture** - Universal client compatibility
2. **Enterprise Scalability** - FastAPI foundation with async/await patterns  
3. **Production Readiness** - Comprehensive testing and Kubernetes deployment

{{< tip >}}
**Ready to implement?** The [Quick Start](quick-start) guide provides step-by-step instructions, while [Taking Your MCP Server to Production](taking-mcp-server-to-production) covers deployment and operational considerations.
{{< /tip >}}

