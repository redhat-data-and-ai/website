---
title: "Taking Your MCP Server to Production"
weight: 40
description: "Complete workflow for deploying your MCP server to production, including Kubernetes deployment, monitoring, and scaling."
---

This document outlines the comprehensive lifecycle for developing, testing, and deploying Model Context Protocol (MCP) servers. Understanding this end-to-end journey helps developers coordinate effectively throughout the development process.

## Complete MCP Development Lifecycle

The MCP development lifecycle follows a three-phase approach that ensures reliable, secure, and high-performing deployments:

{{< mermaid >}}
graph LR
    A[Local Development] --> B[Pre-Production Testing]
    B --> C[Production Deployment]

    A1[Local Testing<br/>• IDE Integration<br/>• Local Development<br/>• Unit Testing] --> A
    B1[Staging Environment<br/>• Integration Testing<br/>• User Acceptance<br/>• Performance Testing] --> B
    C1[Production<br/>• Monitoring<br/>• Operations<br/>• Performance Optimization] --> C

    style A fill:#e1f5fe
    style B fill:#fff3e0
    style C fill:#e8f5e8
{{< /mermaid >}}

## Phase 1: Local Development and Testing

This phase focuses on rapid development and iteration in a controlled local environment.

### Core Development Activities

1. **MCP Server Development**
   - Implement MCP servers using FastMCP or FastAPI patterns
   - Configure authentication integration
   - Add comprehensive error handling and logging
   - Create reusable tools and resources

2. **Local Testing**
   - Test with Cursor agents for rapid feedback
   - Test with Claude Desktop for validation
   - Validate functionality in development environment

3. **Quality Assurance**
   - Pre-commit hooks for code quality
   - Unit tests for tool functionality
   - Documentation validation
   - Security compliance (credential management, input validation)

### Development Environment

**Required Setup**:

- Python development environment with FastMCP or FastAPI
- MCP client tools (Cursor, Claude Desktop)
- Testing framework (pytest)

**Key Technologies**:

- **FastAPI** for enterprise-scale MCP servers with async/await
- **MCP Protocol** for standardized AI agent integration
- **pytest** for comprehensive testing

{{< tip >}}
**Ready to Start Building?** For step-by-step implementation, see the [Quick Start Guide](quick-start).
{{< /tip >}}

## Phase 2: Pre-Production Testing

### Testing Checklist

Before moving to production:

- **Functionality**: All tools tested and working correctly
- **Documentation**: Complete setup, usage, and troubleshooting docs
- **Environment Config**: Ready for deployment (configs, dependencies)
- **Security**: Authentication tested, credentials properly managed
- **Quality**: Code quality checks passing, pre-commit hooks configured
- **Testing**: Both integration and unit tests completed

## Phase 3: Production Deployment

### Kubernetes Deployment

The MCP server template includes Kubernetes deployment configurations:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: mcp-server
        image: "your-registry/mcp-server:latest"
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5001
        readinessProbe:
          httpGet:
            path: /health
            port: 5001
```

### Production Features

- **High availability**: Multi-replica deployment
- **Resource management**: CPU/memory limits and requests
- **Health checks**: Liveness and readiness probes
- **Security**: Pod security contexts and service accounts
- **Networking**: Services and ingress configuration

### Monitoring & Observability

Implement comprehensive monitoring:

- **Health endpoints**: `/health` for Kubernetes probes
- **Metrics**: Expose Prometheus-compatible metrics
- **Logging**: Structured logging for debugging
- **Tracing**: Request tracing for performance analysis

### Deployment Workflow

1. **Build container image**
   ```bash
   podman build -t your-registry/mcp-server:v1.0 .
   podman push your-registry/mcp-server:v1.0
   ```

2. **Deploy to OpenShift** (manifests in `deployment/openshift/`)

   ```bash
   make deploy openshift NAMESPACE=your-project
   ```

   Or apply Kustomize directly:

   ```bash
   oc apply -k deployment/openshift/
   ```

3. **Verify deployment**
   ```bash
   kubectl get pods -n mcp-servers
   kubectl logs -f deployment/mcp-server -n mcp-servers
   ```

## Next Steps

### For New Developers

1. Start with the [Quick Start](quick-start) guide
2. Review [Best Practices](best-practices) for production-ready code
3. Test locally before deployment

### For Production Deployment

1. Prepare Kubernetes cluster
2. Configure secrets and environment variables
3. Deploy using provided manifests
4. Monitor and scale as needed

For questions or support, visit [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).

