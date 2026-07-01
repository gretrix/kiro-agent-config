---
inclusion: manual
---
# Server & Infrastructure Reference

## EC2 / Staging / Production
- **Elastic IP**: <REDACTED>
- **Instance ID**: <REDACTED>
- **Instance Type**: t3.large (8GB RAM, 2 vCPU)
- **SSH**: `ssh -i ~/.ssh/deploy_key ec2-user@<EC2_IP>`
- **Staging**: PM2 process `myapp-staging`, port 3001
- **Production**: PM2 process `myapp`, port 3000, same EC2
- **Staging DB**: MariaDB on localhost:3306
- **Nginx routing**: myapp.com + whitelabel domains -> port 3000 (production). staging.myapp.com -> port 3001 (staging).
- **CRITICAL**: All whitelabel domains MUST be in the production nginx server_name. If a domain routes to staging by accident, users get stale code.

## Deploy Process (MANDATORY — Blue-Green)

**Primary method: Push to git.** CI auto-deploys on push to `main`.

The GitHub Action (`.github/workflows/deploy-app.yml`) handles:
1. TypeScript check
2. Regression tests
3. 5-minute cooldown (batches rapid pushes from multiple sessions)
4. SSH -> `deploy-blue-green.sh`
5. HTTP 200 verification (auto-rollback on failure)
6. Slack notification

**Manual deploy (urgent only):** `ssh server "cd /home/ec2-user/myapp && bash scripts/deploy-blue-green.sh"`

The deploy script deploys to the IDLE slot, health-checks it, then swaps nginx.
The live slot is NEVER touched during deploy. If anything fails, traffic
stays on the current live version. Zero-risk, zero-downtime.

Architecture:
- BLUE  = /home/ec2-user/myapp          (port 3000, PM2: myapp)
- GREEN = /home/ec2-user/myapp-staging   (port 3001, PM2: myapp-staging)
- Active slot tracked in /home/ec2-user/.active-slot ("blue" or "green")
- Nginx swaps which port gets production traffic after health check passes

**CRITICAL**: NEVER use `pm2 stop all` before building.
**CRITICAL**: NEVER run `git pull && npm run build && pm2 reload` on the LIVE slot.
**CRITICAL**: There is NO valid reason to skip the deploy script.
**CRITICAL**: Deploys kill the scheduler process — zombie cleanup handles aftermath.

**Auto-Migrations:** The deploy script automatically runs pending `.sql` files from `lib/db/migrations/`. Applied migrations are tracked in the `schema_migrations` table.

## Cloudflare DNS
- **API Token**: In deploy script (not committed to this repo)
- Use this token to manage DNS for all domains in the account

## Native Apps (Capacitor)
- WebView shell loads the live deployed site. Web deploys = instant app updates.
- Push notifications: Firebase (Android) / APNs (iOS). Tokens stored in `push_tokens` table.
- CI/CD: `.github/workflows/build-android.yml` — manual dispatch, outputs signed .aab

## Live Code References

When this file is loaded, these source files provide the current state of deployment:

### Deploy Script (source of truth for blue-green process)
#[[file:scripts/deploy-blue-green.sh]]

### PM2 Process Configuration
#[[file:ecosystem.config.js]]

### CI/CD Pipeline (auto-deploy on push)
#[[file:.github/workflows/deploy-app.yml]]
