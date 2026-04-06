---
layout: post
title: Managing Freshservice with Claude using MCP
---

# The Problem

Freshservice doesn't currently have a publicly available MCP server.

I built a [Freshservice MCP server](https://github.com/matthewlboyd/freshservice-mcp) to solve this. It lets you manage Freshservice entirely through Claude, using plain English. No more hunting through menus or consulting API docs.

# What is MCP?

MCP (Model Context Protocol) is Anthropic's open standard for connecting AI models to external tools and data sources. Think of it as a plugin system for Claude. You run a local server that exposes a set of tools, point Claude at it, and suddenly Claude can take real actions on your behalf — creating tickets, updating records, searching assets — rather than just describing how to do it.

# What Can It Do?

The server exposes 62 tools across pretty much everything Freshservice offers:

- **Tickets**: create, view, list, update, delete, filter by assignee, team, status, urgency
- **Changes**: full request lifecycle, tasks, and notes
- **Knowledge Base**: create and search articles, manage categories and folders
- **Assets**: inventory management, filtering by type and department
- **People**: agents, requesters, and groups
- **Everything else**: problems, service catalog, contracts, purchase orders, departments, vendors, announcements

Instead of clicking through five screens to find all open P1 tickets assigned to your team, you just ask Claude to do it.

# Setup

## Prerequisites

- Python 3.10+
- `uv` package manager
- Claude Desktop or Claude Code
- A Freshservice account with API credentials (grab your API key from your profile settings)

## Claude Code

This is the simplest path if you're already using Claude Code:

```
claude mcp add freshservice --scope local \
  --env FRESHSERVICE_APIKEY=<your-api-key> \
  --env FRESHSERVICE_DOMAIN=<your-domain.freshservice.com>
```

Using `--scope local` keeps your credentials in a gitignored file, which is the right call.

## Claude Desktop

Clone the repo and run the install script on macOS:

```
git clone https://github.com/matthewlboyd/freshservice-mcp
cd freshservice-mcp
./install.sh
```

Or manually add the following to your Claude config file (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "freshservice": {
      "command": "uv",
      "args": [
        "--directory", "/path/to/freshservice-mcp",
        "run", "freshservice-mcp"
      ],
      "env": {
        "FRESHSERVICE_APIKEY": "<your-api-key>",
        "FRESHSERVICE_DOMAIN": "<your-domain.freshservice.com>"
      }
    }
  }
}
```

Restart Claude Desktop after editing the config and you should see the Freshservice tools available.

## Behind a Corporate Proxy?

If your organization does SSL inspection (see my [post on Zscaler](/2025/10/09/zscaler-ssl-inspection-engineering-heavy/)), you may need to point the server at your CA bundle:

```
SSL_CERT_FILE=/path/to/your/ca-bundle.crt
```

Add that as another `--env` argument or drop it into the JSON config.

# Using It

Once connected, you can just talk to Claude naturally. A few examples of what this unlocks:

- *"Show me all open tickets assigned to the infrastructure team with urgency high or above"*
- *"Analyze the last one hundred tickets and look for trends and common fixes"*
- *"Search the knowledge base for anything related to VPN setup"*
- *"List all laptops in the asset inventory assigned to the Seattle office"*
- *"Add a note to ticket 4821 saying the user has been contacted"*

It handles the API calls, pagination, and filtering under the hood. You just describe what you want.

# Wrapping Up

The full source and issue tracker are at [github.com/matthewlboyd/freshservice-mcp](https://github.com/matthewlboyd/freshservice-mcp). The project includes a test suite covering all 62 tools with mocked HTTP calls, so you can run tests without needing a real Freshservice account. If you run into issues or want to add support for an endpoint I missed, PRs are welcome.
