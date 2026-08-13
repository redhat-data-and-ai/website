---
title: "Claude Desktop"
weight: 10
description: "Anthropic's Claude with Model Context Protocol support. Excellent for conversational workflows, analysis, and exploration."
---

Welcome to the guide for using Claude Desktop with MCP servers! This document shows you how to connect Claude Desktop to Model Context Protocol servers, enabling powerful AI-assisted workflows.

{{< info >}}
**Experimental**: Claude Desktop MCP integration is evolving. [Cursor IDE](cursor) is currently more mature for development workflows, but Claude Desktop excels at conversational exploration and analysis.
{{< /info >}}

## Overview

Claude Desktop is Anthropic's native desktop application that supports the Model Context Protocol (MCP), allowing Claude to interact with external systems. This integration provides:

- **MCP Server Access**: Connect to custom MCP tools and resources
- **Conversational Workflows**: Natural back-and-forth for complex tasks
- **Tool Integration**: Use specialized tools through MCP protocol
- **Data Exploration**: Query and analyze data using natural language

---

## Part 1: Installation

### Step 1: Get Claude Desktop

1. **Download Claude Desktop**
   - Visit [claude.ai/download](https://claude.ai/download)
   - Choose your operating system (macOS or Windows)
   - Download and install the application

2. **Sign In**
   - Launch Claude Desktop
   - Sign in with your Anthropic account
   - Claude Pro subscription recommended for heavy usage

3. **Verify Installation**
   - Open Claude Desktop
   - Test with a simple query: "Hello, what can you help me with?"

---

## Part 2: Configuring MCP Servers

### Locate Configuration File

**macOS:**
```bash
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

### Configure Your MCP Server

Edit the configuration file:

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "node",
      "args": ["/path/to/your/mcp-server/build/index.js"],
      "env": {
        "API_KEY": "your-api-key-here"
      }
    }
  }
}
```

**For HTTP-based MCP servers:**

```json
{
  "mcpServers": {
    "my-http-server": {
      "url": "http://localhost:5001/mcp",
      "transport": "http"
    }
  }
}
```

{{< tip >}}
**Multiple Servers**: You can connect to multiple MCP servers by adding additional entries in the `mcpServers` object. Claude will have access to tools from all connected servers.
{{< /tip >}}

### Restart Claude Desktop

After saving the configuration:

1. Completely quit Claude Desktop (Cmd+Q on Mac, Alt+F4 on Windows)
2. Restart the application
3. Claude will connect to your configured MCP servers on startup

---

## Part 3: Using Claude with MCP Servers

### Verify Connection

Ask Claude about available tools:

> "What tools do you have access to?"

Claude should list the tools from your connected MCP servers.

### Example Workflows

**Data Exploration:**
> "Can you query the database and show me the top 10 customers by revenue?"

**Code Generation:**
> "Help me write a Python function that processes this data using the available tools"

**Documentation:**
> "Explain how the authentication system works in this codebase"

### Best Practices

1. **Be Specific**: Clearly describe what you want Claude to do
2. **Reference Tools**: Mention specific tool names when you know them
3. **Iterate**: Start with simple requests, then refine
4. **Review Output**: Always verify Claude's responses and generated code

---

## Part 4: Troubleshooting

### MCP Server Not Connecting

**Symptoms**: No tools available, connection errors

**Solutions:**
- Verify MCP server is running (`curl http://localhost:5001/health`)
- Check configuration file path is correct
- Review server logs for connection attempts
- Restart Claude Desktop completely

### Tools Not Working

**Symptoms**: Claude can see tools but can't execute them

**Solutions:**
- Check MCP server logs for errors
- Verify authentication/API keys are correct
- Ensure server has proper permissions
- Test tools directly with MCP client

### Configuration Not Loading

**Symptoms**: Changes to config file don't take effect

**Solutions:**
- Verify JSON syntax is valid (use a JSON validator)
- Check file path is exactly correct for your OS
- Completely quit and restart Claude Desktop (not just close window)
- Check Claude Desktop logs for errors

---

## Part 5: Advanced Configuration

### Environment Variables

Pass environment variables to your MCP server:

```json
{
  "mcpServers": {
    "production-server": {
      "command": "node",
      "args": ["server.js"],
      "env": {
        "NODE_ENV": "production",
        "API_KEY": "${API_KEY}",
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Multiple Servers

Connect to multiple MCP servers simultaneously:

```json
{
  "mcpServers": {
    "database-server": {
      "url": "http://localhost:5001/mcp"
    },
    "file-server": {
      "url": "http://localhost:5002/mcp"
    },
    "api-server": {
      "url": "http://localhost:5003/mcp"
    }
  }
}
```

Claude will have access to tools from all servers!

---

## Comparison: Claude Desktop vs Cursor

| Feature | Claude Desktop | Cursor IDE |
|---------|---------------|------------|
| **Best For** | Conversational exploration | Active development |
| **Interface** | Chat-focused | Code editor |
| **MCP Support** | Native | Native |
| **Code Editing** | Copy/paste | Inline editing |
| **Codebase Context** | Limited | Full workspace |
| **Use Case** | Analysis, planning | Writing code |

{{< tip >}}
**Use Both**: Many developers use Cursor for coding and Claude Desktop for planning, analysis, and exploration. They complement each other well!
{{< /tip >}}

---

## Next Steps

- **Build an MCP server**: [MCP Server Template](/templates/mcp-server/)
- **Try Cursor too**: [Cursor Setup Guide](cursor)
- **Join the community**: [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions)

For questions or issues, visit [GitHub Issues](https://github.com/redhat-data-and-ai/website/discussions).

