---
inclusion: fileMatch
fileMatchPattern: '**/services/**,**/scripts/**,**/lib/db/migrations/**,**/scheduler*'
---
# Background Jobs, Scrapers & Data Migration Reference

## Critical Actions Before Optional Cleanup

In any long-running process (scrapers, background jobs, workers):
1. Record results to the database
2. Send notifications (Slack, email, webhooks)
3. THEN cleanup resources (close browsers, shutdown proxies, release connections)

Lost cleanup is harmless — the OS reclaims resources on process exit. Lost notifications or DB writes are permanent. Never place critical I/O after operations that could hang, crash, or terminate the process.

## Debugging Long-Running Processes

When validating or debugging async/background processes, prefer synchronous foreground execution with a timeout over background + log polling:

- **Good**: `ssh server "timeout 300 npx tsx scripts/test-scrape.ts"` (streams output live, fails visibly)
- **Bad**: `ssh server "nohup ... &"` then `sleep 180` then `grep log` (4 iterations to find one bug)

Every trigger/test script should work in foreground mode by default. Only background when running in production via the scheduler.

## Data Migrations

When fixing data in production:

1. **Dry-run first.** Before any bulk UPDATE on a constrained column, run a SELECT that computes target values and checks for unique constraint violations. Never apply a migration without surfacing conflicts first.

2. **Scope to system-created data.** When a fix is based on a business rule (e.g., "all GA foreclosures happen on first Tuesday"), include a temporal boundary (`WHERE created_at >= '2025-01-01'`) to limit to records created by your system. Don't modify historical data that predates your logic.

3. **Handle constraint conflicts explicitly.** If the fix would create duplicates (same address + same corrected date), decide upfront: delete the duplicate, or skip it? Don't let the database error tell you what to do mid-migration.
