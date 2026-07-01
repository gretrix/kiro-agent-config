---
inclusion: fileMatch
fileMatchPattern: '**/scripts/deploy*,**/scripts/rollback*,**/ecosystem.config*,.github/workflows/**,**/services/scraper/**'
---

# Infrastructure Safety Rules

## CRITICAL: Production IP Protection

The production Elastic IP is a PROTECTED RESOURCE. No code may ever:
- Disassociate it
- Release it
- Rotate it
- Modify it in any way

The scraper uses a SEPARATE secondary network interface with its own EIP for rotation.

## Before ANY Infrastructure Change

1. Review incident log for similar past issues
2. Verify no running processes use old code that could conflict
3. Kill old processes BEFORE deploying new code that changes behavior
4. Never deploy infrastructure changes while background processes are running with old code
5. Always have external monitoring confirm the site is up after changes

## Elastic IP Architecture

- **Primary ENI (ens5)**: Production traffic. EIP is PERMANENT and PROTECTED.
- **Secondary ENI (ens6)**: Scraper traffic only. EIP can be rotated freely.
- The ProxyManager MUST only reference the secondary ENI ID.
- A hardcoded `PROTECTED_EIP_ALLOCATION_ID` prevents accidental rotation of production.

## Deployment Safety

**Blue-Green Deployment (PRIMARY — use this for all deploys):**

Preferred: `git push` -> CI auto-deploys.
Manual (urgent only): `ssh server "cd /home/ec2-user/myapp && bash scripts/deploy-blue-green.sh"`

Architecture:
- BLUE = /home/ec2-user/myapp (port 3000)
- GREEN = /home/ec2-user/myapp-staging (port 3001)
- Active slot tracked in /home/ec2-user/.active-slot
- Deploys always go to the IDLE slot. Live traffic is never at risk.
- Health check must pass before nginx swaps. If it fails, nothing changes.
- Instant rollback: `bash scripts/rollback-blue-green.sh`
- 5-minute cooldown: if a deploy happened recently, CI waits then deploys (never skips)

**NEVER deploy by manually running `npm run build` on the live slot.**
**NEVER use `pm2 stop all` before building.**
**NEVER delete `.next` on the live slot.**

**Auto-Migrations:** The deploy script automatically runs pending `.sql` files from `lib/db/migrations/`. Applied migrations are tracked in the `schema_migrations` table.

**Scraper IP Isolation:** The secondary ENI (scraper traffic) is completely independent of the blue-green swap.

## Crash Loop Recovery

If PM2 processes are crash-looping, the server can become unresponsive.

**Recovery order:**
1. Stop the instance via AWS API: `aws ec2 stop-instances --instance-ids <INSTANCE_ID>`
2. Wait for "stopped" state
3. Start it: `aws ec2 start-instances --instance-ids <INSTANCE_ID>`
4. SSH in immediately and `pm2 stop all && pm2 delete all` before the crash loop restarts
5. Fix the root cause (rebuild `.next`, fix env vars, etc.)
6. Start fresh: `pm2 start ecosystem.config.js`

**A reboot is NOT sufficient** when PM2 auto-starts crash-looping processes — they just resume crashing immediately.

**CRITICAL: Deploys kill the scheduler process.** Any actively running scrape will be terminated mid-run. Check `scrape_runs WHERE status = 'running'` first if needed.

## EC2 Memory Constraints

The production EC2 has limited RAM. Builds require ~1.5GB. NEVER attempt a build while PM2 processes are running — it will OOM kill.

Safe build sequence:
1. Stop processes (frees memory)
2. `NODE_OPTIONS='--max-old-space-size=1536' npm run build`
3. Start processes

NEVER `rm -rf .next` on the live slot while the server is running.

## Validation Rule (MANDATORY)

For ANY infrastructure fix:
1. Write the validation test FIRST (a script that replicates the failure scenario)
2. Run it to confirm the bug exists
3. Implement the fix
4. Run the test again to confirm it PASSES
5. Only THEN declare the fix complete

**A fix is NOT validated until the exact failure scenario has been replicated and survived.**
