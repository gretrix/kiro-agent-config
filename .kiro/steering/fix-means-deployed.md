---
inclusion: fileMatch
fileMatchPattern: '**/scripts/deploy*,.github/workflows/**,.snapshots/deploy-pending*'
---

# Fix Means Deployed

## Principle
A production bug fix is NOT complete until all three conditions are met:
1. The code change is committed and pushed
2. The deploy script has been run (`bash scripts/deploy-blue-green.sh`)
3. The fix is verified live (hit the endpoint, check the page, confirm the error no longer fires)

## Batch Deploy Mode (Multi-Session Workflow)

When multiple sessions are working in parallel, deploys can be batched to save time and credits:

- Each session commits and pushes its changes (condition 1 is met per-session)
- Deploys happen automatically via GitHub Actions on push (typecheck + regression tests must pass first)
- Sessions append their changes to `.snapshots/deploy-pending.md` for tracking what's shipping
- The GitHub Action deploys, verifies HTTP 200, and notifies Slack

### Auto-Detection: When Is Batch Mode Active?
A session is in batch mode when ANY of these are true:
- `.snapshots/deploy-pending.md` has entries under "## Pending Changes" (other sessions have uncommitted work)
- The user has explicitly said to hold/batch deploys
- `.snapshots/active-tasks.md` shows multiple sessions with in-progress work

When batch mode is detected, the session should:
1. Commit and push (this triggers the auto-deploy pipeline)
2. Log changes to `.snapshots/deploy-pending.md` (for tracking/verification)
3. NOT manually SSH and deploy (the CI pipeline handles it)
4. Post "Code fix committed — CI will auto-deploy" to Slack threads

### Auto-Deploy Pipeline (GitHub Actions)
Every push to `main` or the working branch triggers:
1. TypeScript check (fails deploy if app/lib has type errors)
2. Regression tests (fails deploy if tests break)
3. Cooldown check (if a deploy happened < 5min ago, waits for cooldown to expire)
4. Deduplication check (if server already has this commit via a later deploy, skips — posts ⏭️ to Slack). **Must compare the LIVE blue/green slot’s `git HEAD`** (see `deploy-app.yml`); comparing only the idle repo would wrongly skip while traffic still serves an old build.
5. SSH deploy via `deploy-blue-green.sh`
6. HTTP 200 verification (auto-rollback on failure)
7. Slack notification

PRs to `main` or the working branch trigger steps 1-2 only (validation, no deploy).

This means: **just push and the deploy happens automatically.** No manual SSH needed. Multiple pushes within 5 minutes are naturally batched because the concurrency queue ensures only one deploy runs at a time, and each deploy does `git pull` (pulling ALL commits pushed before that moment).

### Verification after push (same app, many URLs)

**Who deployed (CI vs manual SSH) does not close the loop.** After the deploy job succeeds — or Slack says **Deploy skipped (dedup)** — still verify the **exact** user path:

1. **Wait for CI** — `gh run watch` on the deploy workflow, or confirm success in GitHub Actions. Dedup means “server already had this commit”; it does **not** remove the need to confirm behavior in the browser.
2. **Use the hostname users use** — Production serves one Next build on `fortuneleo.com` and `dealsidekick.com` (see `server-reference.md`). For whitelabel-sensitive work, spot-check **both** if applicable.
3. **Hit the right route** — **`https://dealsidekick.com/`** (path `/`) is the **Deal Sidekick marketing** landing (`middleware` rewrite to `dealsidekick-landing`). **Lead funnels** are **`/f/{slug}`** on any host (e.g. `https://dealsidekick.com/f/lender-lead-gen-mq2z4rpg?preview=true`). If the user mentions “dealsidekick” in a funnel context, they usually mean **`/f/...`** — use their pasted URL. Verifying `/` alone does not validate `FunnelClient`.
4. **Rule out stale JS** — Hard refresh or incognito when checking layout/branding; Next client bundles cache aggressively.

Log concrete **Verifies** lines in `.snapshots/deploy-pending.md` (full URL + path), then clear entries after you confirm.

### When Immediate Manual Deploy is Still Required
- A production error is actively affecting users RIGHT NOW (500s, broken pages, data loss)
- Security fix (auth bypass, data exposure)
- User explicitly says "deploy this now"
- CI pipeline is broken/disabled

In these cases, SSH and run `deploy-blue-green.sh` manually — don't wait for CI.

### Session Completion in Batch Mode
A session's work is "complete" when:
1. Code is committed and pushed (CI will auto-deploy)
2. Changes are logged in `.snapshots/deploy-pending.md`
3. Slack errors get thread reply saying "Code fix committed — CI will auto-deploy"
4. ✅ reaction is NOT added (waits until CI confirms deploy + you verify live)

## Slack Thread Protocol
When posting a fix summary to a Slack error thread:
- **Immediate mode:** "Code fix ready — deploying now..." → deploy → "✅ Deployed and verified" → add ✅
- **Batch/auto mode:** "Code fix committed and pushed — CI auto-deploy in progress" → verify after CI completes → add ✅

## After CI Auto-Deploys
When the GitHub Action successfully deploys (you'll see the 🚀 notification in #fortune-leo-alerts):
1. Verify each entry in `.snapshots/deploy-pending.md` is working live
2. Add ✅ to resolved Slack errors
3. Clear the deploy-pending file

## Why This Matters
Every minute between "fix written" and "fix deployed" is a minute where users keep hitting the error. But when multiple sessions are running in parallel, deploying after every single commit means 3-4 separate deploy cycles that each take 2-3 minutes — and they can't overlap on the same server. The CI pipeline with a 5-minute cooldown naturally batches rapid pushes: if 3 sessions push within 5 minutes, only one deploy runs (with all 3 commits included).
