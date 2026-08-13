---
title: "Quick Start"
weight: 5
description: "Get your first MCP server running in minutes using the Enterprise MCP Template. Clone, transform, add tools, and deploy quickly with tools-first architecture."
---

Get your MCP server up and running in **minutes**, not hours! This guide walks you through cloning the Enterprise MCP Template, transforming it for your domain, and adding your first tools.

{{< tip >}}
**Why Start Here?** The Enterprise MCP Template uses a **tools-first architecture** that works with ALL MCP clients (LangGraph, CrewAI, Claude Code, etc.) and provides a production-ready foundation from day one.
{{< /tip >}}

{{< info >}}
**What's New!** The Enterprise MCP Template is open-source and publicly available on GitHub at https://github.com/redhat-data-and-ai/template-mcp-server.
{{< /info >}}

## Prerequisites

Before you begin, ensure you have:

- **Python 3.12+** installed
- **UV package manager** ([installation guide](https://docs.astral.sh/uv/getting-started/installation/))
- **Git access** to clone the template repository
- **Basic understanding** of Python and API development
- **Make** — pre-installed on macOS/Linux; on Windows use `choco install make` or `scoop install make`

## Quick run (template as-is)

To run **template-mcp-server** without transforming it yet:

```bash
git clone https://github.com/redhat-data-and-ai/template-mcp-server.git
cd template-mcp-server
make install   # creates .venv and installs deps — required before make local
make local     # starts server on http://localhost:5001
```

Verify in another terminal:

```bash
curl http://localhost:5001/health
```

The MCP JSON-RPC endpoint is `http://localhost:5001/mcp`. When wiring [template-agent](/templates/agent/quick-start/), use that URL in `config/agent/mcp.json`.

To stop the server, press Ctrl+C in the terminal running `make local`.

{{< tip >}}
**Full stack:** MCP `:5001` → agent `:5002` → UI `:5173`. See the [Agentic AI Workshop](/guides/agentic-ai-workshop/) or [Stack Architecture](/reference/architecture/).
{{< /tip >}}

## Step 1: Prepare Your Environment

Get ready to create your new MCP server:

```bash
# Go to where you want to create your project
cd ~/dev  # or wherever you want to work

# Define your project naming (kebab-case - snake_case will be auto-generated)
export NEW_PROJECT_NAME="my-domain-mcp-server"  # Kebab-case for repo/project name
```

## Step 2: Clone the Template

Clone the enterprise MCP template from GitHub:

```bash
# Clone the template
git clone https://github.com/redhat-data-and-ai/template-mcp-server.git
```

## Step 3: Transform for Your Domain

After you have run the server once (see [Quick run](#quick-run-template-as-is)), rename the template for your project. The upstream repository documents a **[rename checklist](https://github.com/redhat-data-and-ai/template-mcp-server#rename-checklist)** — there is no `scripts/transform-template.sh` in the template.

{{< tip >}}
**Recommended:** Follow the rename checklist in the template README, then run `make install` and `make local` in your renamed project.
{{< /tip >}}

### Manual transformation

**Step 1: Create project structure**

```bash
# 1. Copy template to new project directory
cp -r template-mcp-server "$NEW_PROJECT_NAME"
cd "$NEW_PROJECT_NAME"

# 2. Rename the Python package directory (auto-generate snake_case from kebab-case)
NEW_SRC_NAME=$(echo "$NEW_PROJECT_NAME" | sed 's/-/_/g')
mv template_mcp_server "$NEW_SRC_NAME"
```

**Step 2: Update package references**

```bash
# 3. Update all Python imports and references
find . -name "*.py" -exec sed -i '' "s/template_mcp_server/$NEW_SRC_NAME/g" {} \;

# 4. Update configuration files
find . -name "*.toml" -exec sed -i '' "s/template_mcp_server/$NEW_SRC_NAME/g" {} \;
find . -name "*.toml" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;

# 5. Update YAML/container configs
find . -name "*.yaml" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;
find . -name "*.yml" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;
find . -name "Containerfile*" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;
```

On Linux, omit the `''` after `-i` in `sed -i` commands.

**Step 3: Update documentation**

```bash
# 6. Update documentation files
find . -name "*.md" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;
find . -name "*.md" -exec sed -i '' "s/template_mcp_server/$NEW_SRC_NAME/g" {} \;

# 7. Update any remaining references
find . -name "*.txt" -exec sed -i '' "s/template-mcp-server/$NEW_PROJECT_NAME/g" {} \;
```

**Step 4: Verify and install**

```bash
# 8. Check that all references were updated correctly
grep -r "template_mcp_server" . --exclude-dir=.git || echo "All Python refs updated"
grep -r "template-mcp-server" . --exclude-dir=.git || echo "All project refs updated"

# 9. Install and run
make install
make local
curl http://localhost:5001/health
```

### Why Two Naming Formats?

**🔤 Naming Convention Explained:**

| Format | Used For | Example | Rules |
|--------|----------|---------|-------|
| **Kebab-case** | Project/repo names, URLs, containers | `my-domain-mcp-server` | Lowercase, hyphens only |
| **Snake_case** | Python packages, imports, directories | `my_domain_mcp_server` | Auto-generated from kebab-case |

**Example Transformation:**

- **Input**: `"my-domain-mcp-server"` (kebab-case)
- **Auto-generated**: `"my_domain_mcp_server"` (snake_case for Python)

This ensures compatibility with:

- **Git/Container registries** (require kebab-case)
- **Python import system** (requires snake_case)
- **URL/DNS standards** (require kebab-case)

## Step 4: Understand the Tools-First Architecture

The template uses a **tools-first architecture** - everything your MCP server does is implemented as tools:

The template focuses on the **package structure** where your code lives:

{{< info >}}
**Note**: All references to `template_mcp_server` shown below will be different in your transformed project. After running the transformation script or manual steps, these will use your auto-generated Python package name (e.g., `my_domain_mcp_server`).
{{< /info >}}

```text
template_mcp_server/           # Your Python package (gets renamed)
└── src/
    ├── tools/                 # Your MCP functionality lives here
    │   ├── multiply_tool.py   # Mathematical operations
    │   ├── code_review_tool.py # Code analysis
    │   └── redhat_logo_tool.py # Asset access
    ├── assets/                # Static files accessed by tools
    ├── main.py                # FastAPI application entry point
    ├── mcp.py                 # MCP server implementation
    └── settings.py            # Configuration management
```

{{< tip >}}
**🗂️ Full Repository Structure**: For the complete directory layout including `examples/`, `tests/`, and deployment configs, see the repository README and documentation.
{{< /tip >}}

### Why Tools Are Everything

**Universal Compatibility**: Tools work with ANY MCP client - LangGraph, CrewAI, Claude Code, custom agents

**Consistent Interface**: All functionality accessed the same way - no confusion about resources vs prompts vs tools

**Easy Scaling**: Need new capability? Add a new tool. That's it.

**Simple Testing**: All features tested the same way with the same patterns

## Step 5: Add Your First Tool

Let's add a tool specific to your domain. Here's an example for creating new entity IDs:

```bash
# Create your first domain-specific tool
touch template_mcp_server/src/tools/entity_creation_tool.py
```

Add your tool implementation:

```python
# template_mcp_server/src/tools/entity_creation_tool.py
from typing import Dict, Any
import uuid
import time

async def create_new_entity_id(entity_type: str, source_system: str, additional_attributes: Dict = None) -> Dict[str, Any]:
    """
    TOOL_NAME=create_new_entity_id
    DISPLAY_NAME=Create New Entity ID
    USECASE=When you need to generate a new identifier for entities like customers, products, or vendors
    INSTRUCTIONS=Call with entity_type (customer/product/vendor) and source_system (CRM/ERP/etc). Returns new ID with metadata
    INPUT_DESCRIPTION=entity_type: string (customer/product/vendor), source_system: string (CRM/ERP/etc), additional_attributes: dict (optional metadata)
    OUTPUT_DESCRIPTION=Dict with status, entity_id, timestamps, and metadata
    EXAMPLES=create_new_entity_id("customer", "CRM_SYSTEM", {"company": "Acme Corp"})
    PREREQUISITES=Ensure you have entity type and source system identified
    RELATED_TOOLS=validate_entity_id, link_entities

    Create a new entity ID.

    Args:
        entity_type: Type of entity (customer, product, vendor, etc.)
        source_system: System requesting the ID (CRM, ERP, etc.)
        additional_attributes: Optional metadata for the entity

    Returns:
        Dictionary containing new ID and metadata
    """
    try:
        # Example implementation - replace with your logic
        new_entity_id = f"ENTITY-{entity_type.upper()}-{int(time.time())}-{str(uuid.uuid4())[:8]}"

        results = {
            "status": "success",
            "entity_id": new_entity_id,
            "entity_type": entity_type,
            "source_system": source_system,
            "created_timestamp": int(time.time()),
            "additional_attributes": additional_attributes or {},
            "message": f"Successfully created entity ID for {entity_type}"
        }

        # Add your actual async implementation here:
        # - await api.create_entity(entity_type)
        # - await database.insert(entity_record)  
        # - await validate_id_rules(new_entity_id)

        return results

    except Exception as e:
        return {
            "status": "error",
            "message": str(e),
            "entity_type": entity_type,
            "source_system": source_system
        }
```

{{< info >}}
**Agent-Friendly Tool Descriptions**

For AI agents to know **exactly when** to use your tool, follow the template's structured documentation format in your tool's docstring. The example above demonstrates this best practices description template.

**Why This Matters:**

- **Clear Tool Names**: Action-oriented (`create_new_entity_id` not `entity_id_creator`)
- **Specific Use Cases**: Agents know exactly when to call your tool
- **Concrete Examples**: Shows expected input format
- **Workflow Guidance**: Prerequisites and related tools help agents chain operations
- **Better Discovery**: Agents can find the right tool for their task

This structured format ensures agents understand **when**, **why**, and **how** to use your tools effectively!
{{< /info >}}

### Function Types: `def` vs `async def`

Choose the right function type for your tool's workload:

| Function Type | Use When | Examples | Performance |
|---------------|----------|----------|-------------|
| **`def`** | CPU-bound operations | Math calculations, string processing, algorithms, business logic | **Best for computation** |
| **`async def`** | I/O-bound operations | Database queries, API calls, file operations, network requests | **Best for I/O waiting** |

**Quick Decision Guide:**

```python
# ✅ Use def - CPU-bound (computation-heavy)
def calculate_risk_score(data: dict) -> float:
    # Complex mathematical operations
    return risk_algorithm(data)

# ✅ Use async def - I/O-bound (waiting for external systems)
async def create_new_entity_id(entity_type: str) -> Dict[str, Any]:
    # Database insert, API call, file write
    result = await api.create_entity(entity_type)
    return result
```

**For Enterprise FastAPI Scale:**

- FastAPI handles both `def` and `async def` tools efficiently
- Use `async def` when you need to call external systems (databases, APIs)
- Use `def` for pure business logic and calculations

## Step 6: Register Your Tool

Add your tool to the MCP server:

```python
# Edit template_mcp_server/src/mcp.py
# Add your import
from template_mcp_server.src.tools.entity_creation_tool import create_new_entity_id

# Register your tool in the _register_mcp_tools method
def _register_mcp_tools(self) -> None:
    """Register all MCP tools."""
    self.mcp.tool()(multiply_numbers)
    self.mcp.tool()(generate_code_review_prompt)
    self.mcp.tool()(get_redhat_logo)
    self.mcp.tool()(create_new_entity_id)  # Add your tool here
```

## Step 7: Test Locally

Run your MCP server and test your new tool:

```bash
make install   # if you haven't already
make local     # or: python -m template_mcp_server.src.main (with .venv active)
```

In another terminal:

```bash
curl http://localhost:5001/health
python examples/fastmcp_client.py
```

The server will be available at:

- **MCP endpoint**: `http://localhost:5001/mcp`
- **Health check**: `http://localhost:5001/health`

### Quick Test Your Tool

You can test your tool directly:

```python
# Quick test script
import asyncio
from template_mcp_server.src.tools.entity_creation_tool import create_new_entity_id

async def main():
    result = await create_new_entity_id("customer", "CRM_SYSTEM", {"company": "Acme Corp"})
    print(result)

asyncio.run(main())
```

## Step 8: Clean Up Template Example Tools

Now that you have your first domain-specific tool working, remove the template example tools you don't need:

### Remove Example Tool Files

```bash
# Remove template example tools
rm template_mcp_server/src/tools/multiply_tool.py
rm template_mcp_server/src/tools/code_review_tool.py
rm template_mcp_server/src/tools/redhat_logo_tool.py
```

### Remove Tool Registrations

Edit `template_mcp_server/src/mcp.py` and remove the imports and registrations:

```python
# Remove these imports from the top of the file
# from template_mcp_server.src.tools.multiply_tool import multiply_numbers
# from template_mcp_server.src.tools.code_review_tool import generate_code_review_prompt  
# from template_mcp_server.src.tools.redhat_logo_tool import get_redhat_logo

# In the _register_mcp_tools method, remove these lines:
def _register_mcp_tools(self) -> None:
    """Register all MCP tools."""
    # self.mcp.tool()(multiply_numbers)                    # ❌ Remove this
    # self.mcp.tool()(generate_code_review_prompt)         # ❌ Remove this  
    # self.mcp.tool()(get_redhat_logo)                     # ❌ Remove this
    self.mcp.tool()(create_new_entity_id)                  # ✅ Keep your tool
```

### Verify Clean Setup

Test that your server still works with only your domain tools:

```bash
make local

# Should only show your tools now ✅
```

{{< info >}}
**🧹 Clean Development**: Removing unused template tools keeps your MCP server focused on your domain and reduces maintenance overhead. Only include tools your domain actually needs!
{{< /info >}}

## Step 9: Iterate and Add More Tools

Now you're in the development flow! Add more tools as needed:

1. **Identify capability** your domain needs
2. **Create new tool** in `src/tools/`
3. **Register tool** in `mcp.py`
4. **Test locally** with examples
5. **Repeat** until you have all needed functionality

### Common Tool Patterns

**Entity Operations**:

```python
def validate_entity_id(entity_id: str) -> Dict[str, Any]:
    """Validate existing entity ID format and existence."""
```

**Entity Management**:

```python
def link_entities(parent_id: str, child_id: str) -> Dict[str, Any]:
    """Create relationships between entities."""
```

**Data Integration**:

```python
def sync_from_source_system(source: str, entity_data: Dict) -> Dict[str, Any]:
    """Synchronize entity data from source systems."""
```

**Audit Operations**:

```python
def get_entity_audit_trail(entity_id: str) -> Dict[str, Any]:
    """Retrieve complete audit history for an entity."""
```

## Step 10: Prepare for Production

When your tools work locally, prepare for production deployment:

```bash
# Run the full test suite
pytest

# Run pre-commit hooks (code quality)
pre-commit run --all-files

# Make sure everything passes ✅
```

## Step 11: Push to Your Repository

Create your MCP server repository:

```bash
# Create new repository on GitHub first, then:
git remote set-url origin https://github.com/your-org/$NEW_PROJECT_NAME.git

# Add all your changes
git add .
git commit -m "feat: Initial MCP server setup

- Transform template for domain-specific use case
- Add entity creation tools
- All tools tested locally and working
- Ready for production deployment"

# Push to your repository
git push -u origin main
```

## What's Next?

🎉 **Congratulations!** You now have a working MCP server with your domain-specific tools.

### Ready for Production?

When you're ready to deploy your MCP server to production:

👉 **[Taking Your MCP Server to Production](taking-mcp-server-to-production/)** - Complete deployment guide with Kubernetes, monitoring, and scaling

### Want to Learn More?

- **[Best Practices](best-practices/)** - Proven patterns for secure, maintainable MCP servers
- **[Enterprise MCP Template](enterprise-mcp-template/)** - Complete template documentation

### Need Help?

- **GitHub Discussions**: [Ask questions](https://github.com/redhat-data-and-ai/website/discussions)
- **Template Issues**: [Report issues](https://github.com/redhat-data-and-ai/template-mcp-server/issues)

## Why This Approach Works

### **Fast Time to Value**

- **5 minutes**: Clone and set up
- **15 minutes**: First tool working
- **1 hour**: Multiple tools, tested locally
- **Same day**: Deployed to production

### **Tools-First Benefits**

- **Universal compatibility** with all MCP clients
- **Consistent development** pattern for all features  
- **Easy testing** and validation
- **Future-proof** architecture

### 🏢 **Enterprise Ready**

- **Production deployment** configs included
- **Quality gates** with testing and pre-commit hooks
- **Security compliance** with best practices
- **Monitoring and observability** built-in

**Start building your MCP server today!**

