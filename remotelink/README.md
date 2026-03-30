# RemoteLink Plugin for Claude Code

Domain knowledge and MCP integration for [RemoteLink](https://www.unwiredremotelink.com/), a script execution and endpoint management platform.

## Features

- **RemoteLink Skill** — Domain glossary, naming traps, and workflow guidance that help Claude work effectively with RemoteLink's database and API
- **MCP Server Integration** — Pre-configured connection to the RemoteLink MCP server (search, get_doc, query_database, execute_script)
- **Setup Command** — `/remotelink:setup` guides environment configuration

## Prerequisites

- A running RemoteLink instance with the MCP endpoint enabled
- Claude Code CLI or Desktop app

## Installation

```bash
claude /plugin path/to/remotelink
```

## Configuration

Set the `REMOTELINK_URL` environment variable to your RemoteLink Admin base URL:

```bash
export REMOTELINK_URL="https://your-server.example.com"
```

Or run `/remotelink:setup` after installing the plugin for guided configuration.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `REMOTELINK_URL` | Yes | Base URL of your RemoteLink Admin site (e.g., `https://your-server.example.com`) |
