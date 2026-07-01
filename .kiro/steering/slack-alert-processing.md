---
inclusion: manual
---

# Slack Alert Processing

## Philosophy
Every error in the alerts channel exists because something went wrong. There are only two valid resolutions:
1. **Fix the root cause** — so it never fires again
2. **Remove the alert** — if the condition being reported is actually expected behavior (not an error), then the code that fires the alert is wrong and should be removed

"Acknowledge" is never a resolution. "Known limitation" is not a resolution. If an error keeps firing, either fix what causes it or remove the alert that's incorrectly classifying normal behavior as an error.

## Channels

| Channel | ID | Purpose |
|---------|-----|---------|
| app-alerts | <CHANNEL_ID> | Errors, 404s, scraper status, deploys |
| app-user-alerts | <CHANNEL_ID> | User activity: signups, credit usage, analyses saved |
| app-feedback | <CHANNEL_ID> | User feedback submissions |

## How to Read Channels

```javascript
const token = '...'; // from .kiro/settings/mcp.json -> mcpServers.slack.env.SLACK_BOT_TOKEN
fetch(`https://slack.com/api/conversations.history?channel=${CHANNEL_ID}&limit=100`, {
  headers: { Authorization: `Bearer ${token}` }
})
```

## Processing Protocol

1. Read **ALL** messages (limit=100) from the alerts channel
2. Filter for **unprocessed** errors: messages WITHOUT a checkmark reaction
3. For EACH unprocessed error:
   - Investigate the root cause (check code, DB, logs, API)
   - Fix it so it can never fire again, OR remove the alert if it's reporting expected behavior
   - Post a thread reply explaining what caused it and what was fixed
   - Add checkmark reaction to mark it resolved
   - Remove error reaction if present
4. Report to user: what was found, what was fixed

**CRITICAL: Process ALL unprocessed messages. Never stop at 20 or "the recent ones." If there are 68 unprocessed, fix all 68.**

## Thread Reply Format (after fixing)

```
Fixed
- Root cause: [what went wrong]
- Fix: [what code change prevents recurrence]
- Deploy: [pending/deployed]
```

## Resolution Principles

- **Error fires for expected behavior** -> Remove the alert-firing code. Example: "address not found" is not an error — it's a user typing a bad address. Don't alert on it; track it silently in DB.
- **Error fires during deploy** -> Fix the root cause (stale client detection). The error-report API should detect buildId mismatch and not post to Slack.
- **Error fires due to actual bug** -> Fix the bug, deploy, verify it doesn't recur.
- **Error fires due to external API failure** -> Add retry logic, circuit breaker, or graceful degradation so the user never sees an error page.

## Bot Token Location

`.kiro/settings/mcp.json` -> `mcpServers.slack.env.SLACK_BOT_TOKEN`
