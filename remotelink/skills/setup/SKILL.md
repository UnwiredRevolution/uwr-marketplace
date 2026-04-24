---
name: setup
description: >-
  This skill should be used when the user asks to "set up RemoteLink",
  "configure RemoteLink", "change the RemoteLink URL", or needs to point
  RemoteLink at a different instance in a specific project/worktree (e.g.,
  "point RemoteLink at my local dev instance in this worktree"). Also applies
  when the user reports "MCP server not connecting", "RemoteLink URL missing",
  or other RemoteLink MCP connection issues.
---

# RemoteLink Setup

Claude Code prompts for the RemoteLink Admin URL when the plugin is installed or enabled and stores it under `pluginConfigs["remotelink@uwr-marketplace"].options.remotelink_url` in `~/.claude/settings.json`. No shell variables are involved.

## Changing the URL

Run `/plugin`, disable the RemoteLink plugin, then re-enable it to trigger the prompt again. Or edit `~/.claude/settings.json` directly and restart Claude Code.

## Per-worktree override

`pluginConfigs` follows standard settings precedence (**Managed > CLI args > Local > Project > User**). To point at a different instance in a specific checkout — e.g., local RemoteLink for end-to-end testing — drop a `.claude/settings.local.json` at the worktree root:

```json
{
  "pluginConfigs": {
    "remotelink": {
      "options": {
        "remotelink_url": "https://localhost:44316"
      }
    }
  }
}
```

`settings.local.json` is gitignored by Claude Code convention. Use `settings.json` instead if the override should be committed and shared with the team. Restart Claude Code from the worktree after creating the file; verify with `/mcp`.

## Troubleshooting

- **MCP server won't connect**: `curl <remotelink_url>/mcp` — should respond, not timeout or 404.
- **URL missing or invalid**: re-prompt via `/plugin`, or hand-edit `pluginConfigs.remotelink.options.remotelink_url`.
- **HTTPS certificate errors**: ensure self-signed certs on internal servers are trusted by the OS.
- **Worktree override not taking effect**: confirm the file is at `<worktree-root>/.claude/settings.local.json` (not `~/.claude/`), and that Claude Code was restarted from that directory.
