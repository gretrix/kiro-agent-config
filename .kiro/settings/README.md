# Kiro Settings

This directory contains Kiro IDE configuration. Some files are gitignored because they contain secrets.

## Files

| File | Tracked? | Purpose |
|------|----------|---------|
| `lsp.json` | ✅ Yes | Language server settings (shared across machines) |
| `mcp.json` | ❌ Gitignored | MCP server config with API tokens (machine-specific) |
| `mcp.json.example` | ✅ Yes | Template showing expected MCP config shape |

## Setting Up on a New Machine

1. Copy `mcp.json.example` to `mcp.json`
2. Fill in your tokens:
   - **Slack Bot Token** (`SLACK_BOT_TOKEN`): Get from the Slack app settings at https://api.slack.com/apps → OAuth & Permissions → Bot User OAuth Token
3. Restart Kiro or reconnect MCP servers (Command Palette → "MCP: Reconnect")

## What Each MCP Server Does

- **slack**: Gives Kiro read/write access to Slack channels for monitoring alerts, posting deploy notifications, and processing error reports. Used by the `slack-alert-processing.md` steering rule.

## Adding New MCP Servers

Add entries to `mcp.json` following the pattern in the `.example` file. Update the example file too so other devs know what to configure. See [MCP docs](https://modelcontextprotocol.io/) for available servers.
