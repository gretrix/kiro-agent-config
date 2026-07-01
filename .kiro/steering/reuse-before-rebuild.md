---
inclusion: manual
---
# Reuse Before Rebuild

## Principle
Every new capability must be built ON TOP OF what exists, not alongside it. The codebase is the first resource. External tools are the last resort. New patterns that duplicate existing ones are bugs, not features.

## Before Writing Any New Code

Ask yourself ONE question: **"Does something that solves 80% of this already exist in this repo?"**

Search for it. Not a cursory grep — a genuine investigation:
- Search for the NOUNS in your proposal (email → search `email`, `sender`, `transporter`, `sendMail`)
- Search for the VERBS (scrape → search `playwright`, `proxy`, `socks`, `headless`)
- Search for the PATTERN (logging → search `audit`, `log`, `track`, `INSERT INTO`)
- Check `lib/`, `services/`, existing API routes, and migration files

If you find something that does 80% of what you need: EXTEND IT, don't rebuild.

## Before Proposing External Tools or Paid Services

Ask yourself: **"Can our existing infrastructure handle this if I just TRY it first?"**

- We have Playwright + SOCKS5 proxy on EC2 → TRY scraping before suggesting ScrapingBee
- We have MySQL + existing tables → CHECK if a table already tracks this before creating new ones
- We have nodemailer configured → CHECK if a centralized sender exists before making a new one
- We have existing API patterns → CHECK if an endpoint already exists before building one

The rule is: **PROBE before PROPOSE.** Make one test call, check one existing module, run one query. The cost of a 30-second verification is always less than building the wrong thing.

## Before Creating a New Pattern (abstraction, service, table)

Ask yourself: **"Is there an existing pattern that this should plug into?"**

- New email type → should use the centralized `sendTransactionalEmail()` from `lib/email/send.ts`
- New scraper → should use the existing `ProxyManager` + scheduler pattern from `services/scraper/`
- New notification → should use the existing Slack notification functions from `lib/slack-notifications.ts`
- New DB query module → should follow the pattern in `lib/db/` (pool from getPool, RowDataPacket types)
- New admin API → should use `withAuth('admin')` pattern from existing admin routes
- New cron job → should be a PM2 process following the `services/*/scheduler.ts` pattern

When you create a NEW pattern that already has a precedent, you create maintenance debt — two ways to do the same thing, and the next developer (or AI session) won't know which to use.

## At QA/Review Time

Before marking anything complete, ask: **"Did I just build something that already existed in another form?"**

If yes — refactor to use the existing one, even if it means undoing work. It's cheaper to delete 50 lines and import an existing module than to maintain two parallel systems forever.

## The Economics

Building new = more code to maintain + confusion about which to use + wasted credits
Reusing existing = less code + consistent patterns + faster builds + fewer bugs

This isn't about being lazy. It's about compound returns: every module that plugs into existing infrastructure makes the NEXT module faster to build. Every parallel system makes everything slower.
