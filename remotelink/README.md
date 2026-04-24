# RemoteLink Plugin for Claude Code

Domain knowledge and MCP integration for [RemoteLink](https://www.unwiredremotelink.com/), a script execution and endpoint management platform.

## Features

- **RemoteLink Skill** — Domain glossary, naming traps, and workflow guidance that help Claude work effectively with RemoteLink's database and API
- **MCP Server Integration** — Pre-configured connection to the RemoteLink MCP server (search, get_doc, query_database, execute_script)
- **Setup skill** — `/remotelink:setup` covers per-project overrides and connection troubleshooting

## Prerequisites

- A running RemoteLink instance with the MCP endpoint enabled
- Claude Code CLI or Desktop app

## Installation

```bash
claude /plugin path/to/remotelink
```

## Configuration

Claude Code prompts for your RemoteLink Admin base URL (`remotelink_url`) when you install or enable the plugin and stores it under `pluginConfigs.remotelink.options` in `~/.claude/settings.json`. Use the base URL only — no trailing slash, no `/mcp` (the plugin appends `/mcp`).

To change it later, run `/plugin` and re-enable the plugin, or edit settings directly.

### Per-project override

To point this plugin at a different RemoteLink instance in a specific worktree (e.g., a local dev server) without changing your global default, drop a `.claude/settings.local.json` into that project. See `/remotelink:setup` for the exact snippet.
