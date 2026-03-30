---
name: setup
description: >-
  This skill should be used when the user asks to "set up RemoteLink",
  "configure RemoteLink", "connect to RemoteLink", "install RemoteLink plugin",
  "get started with RemoteLink", or needs help setting the REMOTELINK_URL
  environment variable. Also applies when the user reports "MCP server not
  connecting", "REMOTELINK_URL not found", "fix RemoteLink connection", or
  other RemoteLink MCP connection issues.
---

# RemoteLink Setup

Configure the `REMOTELINK_URL` environment variable required by the RemoteLink MCP server.

## Prerequisites

- A running RemoteLink instance with the MCP endpoint enabled
- The base URL of the RemoteLink Admin site (e.g., `https://your-server.example.com`)

## Setup Steps

### 1. Determine the RemoteLink URL

Ask the user for their RemoteLink Admin URL. This is the base URL without any path — for example, `https://your-server.example.com` (not `https://your-server.example.com/mcp`). The plugin appends `/mcp` automatically.

Before setting the variable, validate the URL: strip any trailing slash or path segments (e.g., `/mcp`, `/Admin`). The value should be just the scheme and host like `https://your-server.example.com`.

### 2. Set the Environment Variable

The `REMOTELINK_URL` environment variable must be available to Claude Code at startup.

**Option A: Claude Code settings (recommended)**

Add to the user's Claude Code settings via the `env` field in `~/.claude/settings.json`:

```json
{
  "env": {
    "REMOTELINK_URL": "https://your-server.example.com"
  }
}
```

**Option B: Shell profile**

For bash/zsh, append to the appropriate profile file (`~/.bashrc`, `~/.zshrc`, or `~/.bash_profile`):

```bash
export REMOTELINK_URL="https://your-server.example.com"
```

For PowerShell, add to `$PROFILE`:

```powershell
$env:REMOTELINK_URL = "https://your-server.example.com"
```

After setting via shell profile, restart the terminal and Claude Code.

### 3. Verify Connection

After setting the environment variable and restarting Claude Code, verify the MCP server connects by running `/mcp` and checking that the `remotelink` server shows as connected with its tools available (search, get_doc, query_database, execute_script).

## Troubleshooting

- **MCP server not connecting**: Verify the URL is reachable from the machine running Claude Code. Test with `curl <REMOTELINK_URL>/mcp` — it should respond (not timeout or 404).
- **Environment variable not found**: Ensure Claude Code was restarted after setting the variable. Check with `echo $REMOTELINK_URL` in the Claude Code terminal.
- **HTTPS certificate errors**: For self-signed certificates on internal servers, ensure the certificate is trusted by the operating system.
