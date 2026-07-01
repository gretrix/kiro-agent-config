---
inclusion: always
---
# Local Dev Rules

## Domains (all route to port 3002)
- https://local.myapp.com
- https://local.whitelabel1.com (whitelabel)
- https://local.whitelabel2.com (whitelabel)

## Test Account (for authenticated Playwright testing)
- **Email:** In `.env.local` as ADMIN_LOGIN_EMAIL
- **Password:** In `.env.local` as ADMIN_PASSWORD
- **User ID:** 22
- When you need to test authenticated flows (report viewing, dashboard, saved analyses), log in via Playwright using these credentials. Do NOT say "I can't authenticate" — you have full access.

## Infrastructure
- MySQL runs in Docker container `myapp-db` on port 3308
- Check with: `docker ps | findstr myapp`
- DB credentials are in `.env.local` (DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME)
- NEVER ask the user where the database is — read `.env.local` and check `docker ps`
- **Production DB queries:** Use `ssh -i ~/.ssh/deploy_key ec2-user@<EC2_IP> "cd /home/ec2-user/myapp && bash scripts/ssh-query.sh 'YOUR SQL HERE'"` — this avoids PowerShell/SSH/bash quoting issues entirely. Never hand-roll mysql commands through SSH.

## Rules
1. Dev server MUST use port 3002: next dev -p 3002
2. NEVER modify hosts file or C:\caddy\Caddyfile
3. NEVER use ports 3000, 3001, 3003-3005, or 8080
4. Port registry: E:\Documents\PORT_REGISTRY.md
5. Playwright browsers path: E:\playwright-browsers
6. When diagnosing "page not loading" issues, ALWAYS check server logs first (not just HTTP status codes)

## Slack Channels
- **app-alerts**: <CHANNEL_ID> (errors, deploys, uptime alerts, regression failures — ALL automated notifications)
- **app-user-alerts**: <CHANNEL_ID> (user activity: signups, subscriptions, credit usage, analyses saved)
- **app-feedback**: <CHANNEL_ID> (copilot limitation reports, user feedback)
- **app-team**: <CHANNEL_ID> (team discussion ONLY — no bot posts should go here)

## Slack Access (FULL — DO NOT SAY YOU CAN'T ACCESS SLACK)
- You have FULL read/write access to all Slack channels
- **Bot token:** In `.kiro/settings/mcp.json` (gitignored)
- **DO NOT** rely on the Slack MCP server — it's often disconnected. Use the token directly:
  ```powershell
  $headers = @{ "Authorization" = "Bearer $env:SLACK_BOT_TOKEN" }
  Invoke-RestMethod -Uri "https://slack.com/api/conversations.history?channel=<CHANNEL_ID>&limit=10" -Headers $headers
  ```
- To post: `Invoke-RestMethod -Method Post -Uri "https://slack.com/api/chat.postMessage" -Headers $headers -Body (@{channel="<CHANNEL_ID>";text="message"} | ConvertTo-Json) -ContentType "application/json"`
- NEVER say "I don't have access to read Slack" — you DO. Use the API directly.

## Cloudflare Access (FULL — DO NOT SAY YOU CAN'T MANAGE DNS)
- You have FULL Cloudflare API access via token in `scripts/deploy-blue-green.sh`
- Token: In deploy script (gitignored from this repo)
- Permissions: DNS read/write, zone read/write, cache purge, zone settings read/write
- **Does NOT have:** Rulesets/Redirect Rules permission, SSL settings write, Workers
- Use Page Rules (legacy) for redirects — works with this token
- Use PowerShell `Invoke-RestMethod` with `$headers = @{ "Authorization" = "Bearer <token>"; "Content-Type" = "application/json" }`
- NEVER ask the user "do you have Cloudflare access?" — you DO. Use the API directly.

### Managed Zones
| Domain | Zone ID | Purpose |
|--------|---------|---------|
| myapp.com | <ZONE_ID> | Primary app |
| whitelabel1.com | <ZONE_ID> | Whitelabel |
| whitelabel2.com | <ZONE_ID> | Whitelabel |

## Deploy Process (summary)
**Primary method: Push to git.** CI auto-deploys on push to `main`.
**Manual deploy (urgent only):** `ssh -i ~/.ssh/deploy_key ec2-user@<EC2_IP> "cd /home/ec2-user/myapp && bash scripts/deploy-blue-green.sh"`
**NEVER hot-patch the live slot.** ALWAYS use the deploy script.

For full deploy architecture, server details, and Capacitor/mobile builds, see `#server-reference`.

## SSH from PowerShell (quoting rules)
PowerShell mangles quotes through SSH. Follow these patterns ALWAYS:
- **DB queries:** `ssh -i $HOME/.ssh/deploy_key ec2-user@<EC2_IP> "cd /home/ec2-user/myapp && bash scripts/ssh-query.sh 'SIMPLE SQL HERE'"`
- **Multi-line or complex commands:** Write a `.sh` script locally -> `scp` it to `/tmp/` -> `ssh ... "bash /tmp/script.sh"`
- **Simple commands** (ls, grep, cat, pm2 list): fine to inline directly
- **NEVER** use heredocs, nested quotes, or `DATE_FORMAT('%...')` inline — they ALWAYS break
