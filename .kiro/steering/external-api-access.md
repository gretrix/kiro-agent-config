---
inclusion: manual
---
# External API Access

Direct API access available to the agent. When a task involves any of these services, use them — do NOT claim "I don't have access" without checking here first.

## Available

| Service | Method | Scope |
|---------|--------|-------|
| Slack | REST API via Bot token (in local-dev rules) | Full read/write all channels |
| Cloudflare | REST API via token (in deploy script) | DNS, zones, cache, bot management |
| GitHub | `gh` CLI (authenticated) | Repos, workflows, secrets, PRs |
| EC2 Production | SSH via deploy key | Server management, DB queries, deploys |
| Google Play (upload) | Via GitHub Actions workflow | Build + upload AAB to production track |
| Gmail/IMAP (inbox read) | Node.js `imap` package + SMTP credentials from `.env.local` | Read inbox |

## Not Available (requires user action or new MCP setup)

| Service | Why | Workaround |
|---------|-----|------------|
| Google Play Console (review status) | Service account key only in GitHub Secrets | Trigger workflow or check inbox via IMAP |
| Stripe | MCP configured but disabled | Enable in `~/.kiro/settings/mcp.json` |

## Credentials Location

All credentials live in the local-dev rules (`.kiro/steering/local-dev.md`) or `.env.local`. Never hardcode new credentials — add them to the appropriate config and reference from there.
