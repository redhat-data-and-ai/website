---
title: "Development Best Practices"
weight: 30
description: "Comprehensive best practices for developing AI agents and MCP servers, covering code quality, security, performance, and maintainability."
---

This guide provides comprehensive best practices for developing production-ready AI agents and MCP servers. Whether you're building conversational agents or custom MCP servers, following these practices will help you create maintainable, secure, and high-performing solutions.

## Development Areas

- **[Agent Development Best Practices](#agent-development-best-practices)** - LangGraph workflows and MCP client integration
- **[MCP Server Development Best Practices](#mcp-server-development-best-practices)** - Framework-agnostic patterns

---

## Agent Development Best Practices

The [template-agent](/templates/agent/) Deep Agent template uses **config-as-code** and the Aegra LangGraph API — not a monolithic `ask_agent/` Python package. For agent setup, see:

- [Agent Quick Start](/templates/agent/quick-start/) — `make install`, `make local`, port **5002**
- [Agent Architecture](/templates/agent/architecture/) — `config/agent/` layout, MCP wiring, API reference
- [Config-as-code](/reference/config-as-code/) — prompts, subagents, skills, `mcp.json`

### Configuration (config-as-code)

Agent behavior lives in files under `config/agent/`, not in `src/agent/*.py`:

```text
config/agent/
├── PROMPT.md               # Orchestrator prompt + frontmatter
├── subagents/*.md          # Subagent definitions
├── skills/*/SKILL.md       # Agent Skills workflows
├── mcp.json                # MCP servers (default local URL: http://localhost:5001/mcp)
└── runtime/agent.yaml      # Middleware, memory, providers
```

Wire MCP servers in `mcp.json` and reference them from orchestrator/subagent frontmatter (`mcps:`). Secrets (Vertex AI, Postgres, Redis, Langfuse) stay in `.env`.

### LangGraph API testing

Test the running agent with the LangGraph HTTP API:

```bash
curl -X POST http://localhost:5002/threads -H "Content-Type: application/json" -d '{}'

curl -N -X POST "http://localhost:5002/threads/THREAD_ID/runs/stream" \
  -H "Content-Type: application/json" \
  -d '{"assistant_id": "agent", "input": {"messages": [{"role": "human", "content": "Hello"}]}, "stream_mode": "updates"}'
```

For workflow and middleware customization beyond config files, see `deep_agent/` in the template repository and [Agent Architecture](/templates/agent/architecture/).

---

## MCP Server Development Best Practices

{{< info >}}
**Architecture First**: For MCP server structure (FastAPI + FastMCP, nested package layout, async patterns), see the [Enterprise MCP Template](/templates/mcp-server/enterprise-mcp-template/) documentation.
{{< /info >}}

### 1. Configuration Management

```python
from pydantic_settings import BaseSettings
from typing import Optional

class ServerSettings(BaseSettings):
    """Type-safe server configuration."""

    # External service configuration
    EXTERNAL_API_URL: str
    EXTERNAL_API_KEY: str

    # Server configuration (see template .env.example)
    MCP_HOST: str = "localhost"
    MCP_PORT: int = 5001
    LOG_LEVEL: str = "INFO"

    # Performance settings
    MAX_CONNECTIONS: int = 100
    TIMEOUT_SECONDS: int = 30

    # SSL Configuration
    SSL_KEYFILE: Optional[str] = None
    SSL_CERTFILE: Optional[str] = None

    class Config:
        env_file = ".env"
        case_sensitive = False
```

### 2. Input Validation & Security

```python
# utils/validation.py
import re
from pydantic import BaseModel, validator

class QueryValidator(BaseModel):
    """Validate SQL queries for security."""

    query: str
    max_length: int = 10000

    @validator('query')
    def validate_query_safety(cls, v):
        if not v.strip():
            raise ValueError("Query cannot be empty")

        # Check for dangerous patterns
        dangerous_patterns = [
            r'\b(DELETE|DROP|TRUNCATE|ALTER)\b',
            r'\b(EXEC|EXECUTE)\b',
            r'\b(UNION.*SELECT)\b',
        ]

        for pattern in dangerous_patterns:
            if re.search(pattern, v, re.IGNORECASE):
                raise ValueError(f"Query contains dangerous pattern")

        return v
```

### 3. Error Handling & Resilience

```python
# utils/resilience.py
from enum import Enum
import time

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    """Circuit breaker pattern for external services."""

    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    async def call(self, func, *args, **kwargs):
        """Execute function with circuit breaker protection."""
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.timeout:
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = await func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

### 4. Performance Optimization

```python
# utils/caching.py
import hashlib
import json
from typing import Any, Optional, Dict

class CacheManager:
    """Async cache with TTL support."""

    def __init__(self, default_ttl: int = 300):
        self.cache: Dict[str, Dict[str, Any]] = {}
        self.default_ttl = default_ttl

    def cache_key(self, func_name: str, args: tuple, kwargs: dict) -> str:
        """Generate cache key from function and arguments."""
        key_data = {"func": func_name, "args": args, "kwargs": kwargs}
        key_str = json.dumps(key_data, sort_keys=True)
        return hashlib.md5(key_str.encode()).hexdigest()

    async def get(self, key: str) -> Optional[Any]:
        """Get value from cache."""
        if key in self.cache:
            entry = self.cache[key]
            if time.time() < entry["expires_at"]:
                return entry["value"]
            else:
                del self.cache[key]
        return None
```

### 5. Comprehensive Testing

```python
# tests/test_integration.py
import pytest
from unittest.mock import patch

@pytest.mark.asyncio
async def test_tool_integration(test_client):
    """Test tool integration with external services."""
    response = await test_client.post("/tools/query_data", json={
        "query": "SELECT * FROM test_table",
        "limit": 10
    })

    assert response.status_code == 200
    result = response.json()
    assert result["status"] == "success"
```

## Common Anti-Patterns to Avoid

### Agent Development

- **Hardcoded MCP server addresses** - Use configuration management
- **Lack of error handling** - Always implement comprehensive error handling
- **Infinite loops in workflows** - Set iteration limits and timeouts
- **Synchronous MCP calls** - Use async/await patterns
- **Poor state management** - Use proper TypedDict for state

### MCP Server Development

- **No input validation** - Always validate and sanitize inputs
- **Missing connection pooling** - Use connection pools for external services
- **No caching strategy** - Cache expensive operations
- **Lack of observability** - Implement comprehensive logging and monitoring
- **No circuit breakers** - Protect against cascading failures

## Resources

### Documentation

- **[Quick Start](quick-start)** - Hands-on implementation guide
- **[Taking to Production](taking-mcp-server-to-production)** - Deployment lifecycle

### External Resources

- **[LangGraph Documentation](https://langchain-ai.github.io/langgraph/)** - Advanced agent workflows
- **[FastAPI Documentation](https://fastapi.tiangolo.com/)** - Enterprise web framework
- **[Pydantic Settings](https://docs.pydantic.dev/usage/settings/)** - Configuration management

For questions and community support, visit [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions).

