---
inclusion: manual
---
# Session Briefing

When this file is referenced, read and synthesize the following sources to understand the current state of the project:

## Current Work State
#[[file:.snapshots/active-tasks.md]]

## Pending Deploys
#[[file:.snapshots/deploy-pending.md]]

## Next Session Priorities
#[[file:.snapshots/next-session-tasks.md]]

## Recent Hook/Steering Changes
#[[file:.kiro/hooks/CHANGELOG.md]]

## How to Use This Briefing

After reading the above files, provide a structured summary:
1. **In flight:** What tasks are currently in progress from other sessions?
2. **Deploy queue:** Is there anything pending verification after a CI deploy?
3. **Priority work:** What's next in the queue?
4. **Conflicts to avoid:** Are other sessions touching files you might also need?

This prevents duplicate work, missed deploys, and conflicting edits across parallel sessions.

## Available Context (load with # when needed)

| Reference | Use when... |
|-----------|-------------|
| `#data-sources` | Working with APIs, property data, or scraper code |
| `#server-reference` | Deploying, SSH, mobile builds, DNS, Play Store |
| `#decision-log` | Proposing architecture or checking past decisions |
| `#product-overview` | Writing marketing copy, store listings, or feature descriptions |
| `#slack-alert-processing` | Processing error alerts from Slack |
| `#branch-strategy` | Working on multi-session features with PRs |
| `#dry-principles` | Refactoring or creating shared components |
| `#human-centered-design` | Building user-facing UI or forms |
| `#landing-page-framework` | Creating marketing/landing pages |
| `#writing-hooks-and-steering` | Modifying hooks or steering rules |
