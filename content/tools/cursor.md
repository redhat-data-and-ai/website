---
title: "Cursor IDE"
weight: 5
description: "AI-first IDE with deep integration via MCP servers. Primary tool for AI-assisted development with codebase awareness and inline editing."
---

Welcome to the complete guide for using Cursor IDE with AI Templates! This document will walk you through installation, MCP server configuration, and leveraging AI to boost your productivity.

## Overview

Cursor IDE is an AI-powered development environment that integrates with MCP servers to provide intelligent assistance for:

- **Application Development**: AI pair programming and code generation
- **MCP Server Development**: Building and testing MCP tools
- **Agent Development**: Creating LangGraph workflows
- **Documentation**: Writing and improving docs

---

## Part 1: Getting Started with Cursor

### Step 1: Install Cursor IDE

#### Download and Install

1. Visit [cursor.sh](https://cursor.sh) and download for your operating system
2. Install the application:
   - **macOS**: Drag to Applications folder
   - **Windows**: Run the installer
   - **Linux**: Install the package

3. Launch Cursor and sign in with your Cursor account
4. Verify your email address via the confirmation link

**Time estimate**: 5-10 minutes

{{< info >}}
**New to Cursor?** Cursor is an AI-powered IDE built on VSCode that enhances developer productivity through intelligent code assistance and natural language interactions.
{{< /info >}}

### Step 2: Basic Cursor Configuration

#### Configure AI Model

1. Open **Settings** (Cmd+, on Mac, Ctrl+, on Windows)
2. Navigate to **Cursor** section
3. Choose your preferred AI model:
   - **GPT-4** - Best for complex reasoning
   - **Claude** - Great for coding tasks
   - **GPT-3.5** - Faster for simple tasks

#### Trust External Websites (Optional)

If you want Cursor to access documentation sites:

1. Go to **Settings** → **Cursor** → **External Websites**
2. Add trusted domains (e.g., `github.com`, your documentation site)

---

## Part 2: MCP Server Integration

### What is MCP Integration?

Model Context Protocol (MCP) allows Cursor to connect to specialized servers that provide additional capabilities:

- **Custom tools** for domain-specific operations
- **Data access** from databases and APIs
- **Context** about your systems and workflows

### Configure MCP Servers in Cursor

{{< info >}}
**Prerequisites**: You need an MCP server running locally or accessible via network. See [MCP Server Template](/templates/mcp-server/) to build your own.
{{< /info >}}

#### Step 1: Navigate to MCP Settings

1. Open Cursor **Settings**
2. Search for "MCP" or navigate to **Cursor** → **Model Context Protocol**
3. Click **"MCP Integrations"**

#### Step 2: Add Custom MCP Server

1. Click **"Add Custom MCP Server"**
2. Configure your server:

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "node",
      "args": ["/path/to/mcp-server/build/index.js"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

**For HTTP-based MCP servers:**

```json
{
  "mcpServers": {
    "my-http-mcp-server": {
      "url": "http://localhost:5001/mcp",
      "transport": "http"
    }
  }
}
```

#### Step 3: Verify Connection

1. After saving, Cursor will attempt to connect
2. Check the MCP status indicator
3. You should see your server's tools available in the agent

{{< tip >}}
**Connection Issues?** Make sure your MCP server is running and accessible. Check the server logs for connection attempts.
{{< /tip >}}

---

## Part 3: Using Cursor with AI Templates

### Chat with Your Agent

1. **Open a new chat** (Cmd+L on Mac, Ctrl+L on Windows)
2. **Select Agent mode** from the dropdown
3. **Ask questions** or request help:
   - "Help me build a new MCP tool for database queries"
   - "Create a LangGraph workflow for this task"
   - "Explain how this code works"

### Inline AI Editing

1. **Select code** in your editor
2. **Press Cmd+K** (Mac) or **Ctrl+K** (Windows)
3. **Describe changes** in natural language:
   - "Add error handling to this function"
   - "Refactor this to be async"
   - "Add type hints and docstrings"

### Codebase Questions

1. **Press Cmd+Shift+L** (or Ctrl+Shift+L)
2. **Ask about your codebase**:
   - "Where is the authentication logic?"
   - "How does the MCP server register tools?"
   - "Show me examples of async tools"

---

## Part 4: MCP-Powered Workflows

### Example: Building an MCP Tool

**Ask Cursor:**
> "I need to create a new MCP tool that queries a database. Can you help me build it following the template pattern?"

Cursor will:
1. Understand your MCP server structure
2. Generate the tool following your patterns
3. Add proper type hints and documentation
4. Suggest testing approaches

### Example: Testing MCP Tools

**Ask Cursor:**
> "Help me write tests for the database query tool we just created"

Cursor will:
1. Create pytest test cases
2. Mock external dependencies
3. Test both success and error cases
4. Follow your testing patterns

---

## Tips for Effective Use

### Best Practices

- **Be specific**: "Add error handling for network timeouts" vs "improve this code"
- **Provide context**: Reference your MCP server patterns and conventions
- **Iterate**: Start simple, then refine with follow-up prompts
- **Review AI suggestions**: Always review and understand generated code

### Common Workflows

1. **New Feature Development**
   - Describe the feature to Cursor
   - Review generated code
   - Test with your MCP client
   - Iterate based on results

2. **Debugging**
   - Share error messages with Cursor
   - Ask for explanation and fixes
   - Test the suggested solutions

3. **Code Review**
   - Select code and ask "Can this be improved?"
   - Request security review
   - Ask for performance optimization suggestions

---

## Troubleshooting

### MCP Server Not Connecting

**Symptoms**: Tools don't appear, connection errors

**Solutions**:
- Verify MCP server is running (`curl http://localhost:5001/health`)
- Check Cursor MCP settings for correct URL/command
- Review MCP server logs for errors
- Restart Cursor after configuration changes

### Agent Not Using MCP Tools

**Symptoms**: Agent doesn't call your custom tools

**Solutions**:
- Ensure tools have clear, descriptive docstrings
- Explicitly mention the tool in your prompt
- Check that server connection is active (green indicator)
- Try rephrasing your request to match tool use cases

### Performance Issues

**Symptoms**: Slow responses, timeouts

**Solutions**:
- Use lighter AI models for simple tasks (GPT-3.5 vs GPT-4)
- Reduce context window size in settings
- Close unnecessary chat sessions
- Check MCP server performance

---

## Next Steps

- **Build your first MCP server**: [MCP Server Template](/templates/mcp-server/)
- **Create an AI agent**: [Agent Template](https://github.com/redhat-data-and-ai/template-agent)
- **Join the community**: [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions)

{{< tip >}}
**Pro Tip**: Create a `.cursorrules` file in your project root to give Cursor context about your MCP server patterns, coding standards, and best practices!
{{< /tip >}}

