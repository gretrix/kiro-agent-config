---
inclusion: manual
---

# Decision Log

## Principle
When the user approves an approach, architecture decision, or feature direction during a conversation, it gets logged HERE immediately. Future sessions read this before proposing alternatives. If a decision is already logged, execute it — don't re-ask.

## Approved Decisions

### Data Source Architecture
- **Decision:** APIVex **Zillow is PRIMARY** for property details + rent estimates. APIVex **Realtor is PRIMARY for sold comps**. Realtor is SECONDARY for property details gap-fill (photos, descriptions).
- **Evidence:** Ran head-to-head comparison with 3 addresses (urban/suburban/rural) + sold comp searches.
- **Architecture:** `Zillow search_address` (1 call) -> property details + rent estimate. If sqft/beds missing -> `Realtor property/details` for gap-fill.
- **Rate Limit:** 2 req/sec SHARED across all APIVex endpoints. Enforced by `lib/apivex/rateLimiter.ts`.

### Credit Deduction Timing
- **Decision:** Credits deducted AFTER success only. Never charge for failed operations.
- **Applied to:** Property analysis, skip trace, copilot chat (all routes updated)

### Blue-Green Deployment
- **Decision:** Two directories alternate as live/idle. Deploy to idle, health check, swap nginx. Never touch live during build.
- **Scripts:** `deploy-blue-green.sh`, `rollback-blue-green.sh`
- **Active slot tracked in:** `/home/ec2-user/.active-slot`

### Address Not Found UX
- **Decision:** Return HTTP 200 with `success: false` + Google search link when address isn't found. Track failures in DB. No Slack alert on individual not-founds (user typos are expected behavior, not errors).

### Native Mobile App (Capacitor)
- **Decision:** Use Capacitor to wrap the existing Next.js web app as native iOS + Android apps. Single codebase — the native shell loads the live site in a WebView. Web deploys = instant app updates with no store review.

### CI Auto-Deploy + Batch Mode
- **Decision:** Deploys happen automatically via GitHub Actions on push. Sessions just commit+push — no manual SSH deploy needed. Multiple rapid pushes are batched via 5-minute cooldown.
- **Pipeline:** TypeScript check -> regression tests -> cooldown check -> SSH deploy -> HTTP 200 verify -> auto-rollback on failure -> Slack notify.

### Deploy Deduplication + Session Branch Strategy
- **Deploy dedup:** After cooldown wait, CI checks if the server's deployed commit already includes the triggering commit. If yes, skips redundant deploy.
- **Session branches:** Convention `session/<topic>` for multi-turn features. PRs to main trigger CI checks but NOT deployment. Merge triggers auto-deploy.

### Architecture Over Convenience
- **Decision:** When the data already tells you WHEN something should happen, don't poll — listen. Choose event-driven over polling when data provides the signal.
