---
inclusion: manual
---
# Steering File Inventory

Quick reference for what each steering file does and when it loads.

## Always-On (loaded every conversation)

| File | Purpose |
|------|---------|
| `local-dev.md` | Port 3002, test account, DB location, Slack channels, deploy summary, Cloudflare access |
| `d-drive-installations.md` | All installs/downloads on E: drive, never C: |

## FileMatch (loaded when matching files are open in editor)

| File | Trigger Pattern | Purpose |
|------|----------------|---------|
| `api-routing.md` | `lib/apivex/**, lib/rentcast/**, lib/data-sources/**, services/scraper/**, app/api/property*/**` | Which API to use for which data need |
| `background-jobs-reference.md` | `services/**, scripts/**, lib/db/migrations/**, scheduler*` | Cleanup ordering, foreground debugging, migration safety |
| `data-sources.md` | `lib/apivex/**, lib/rentcast/**, services/scraper/**, lib/data-sources/**` | Full API inventory, rate limits, response shapes |
| `external-api-testing.md` | `lib/apivex/**, lib/rentcast/**, scripts/test-*, scripts/spike-*` | Test raw response first, check for error-in-200 |
| `fix-means-deployed.md` | `scripts/deploy*, .github/workflows/**, .snapshots/deploy-pending*` | CI auto-deploy, batch mode, verification protocol |
| `hook-evolution-memory.md` | `hooks/**, steering/**` | Read CHANGELOG before modifying hooks/steering |
| `infrastructure-safety.md` | `scripts/deploy*, scripts/rollback*, ecosystem.config*, .github/workflows/**` | Blue-green architecture, crash recovery, memory constraints |

## Manual (load with `#filename` in chat)

| Reference | Purpose |
|-----------|---------|
| `#architecture-over-convenience` | Prefer event-driven over polling when data tells you when |
| `#automated-testing-workflow` | Validate by triggering real code paths |
| `#branch-strategy` | Session branches, auto-merge, CI behavior by branch type |
| `#cross-boundary-verification` | API <-> UI <-> email: same-field semantics, nullability |
| `#decision-log` | Approved architecture decisions |
| `#dry-principles` | DRY checklists for components, types, APIs, validation |
| `#human-centered-design` | Full UX validation framework, emotional design |
| `#landing-page-framework` | 9-block conversion structure for marketing pages |
| `#multi-session-safety` | Never `git add -A`, stage specific files, coordinate via snapshots |
| `#product-overview` | Full feature set for marketing copy and store listings |
| `#reuse-before-rebuild` | Search for existing solutions before creating new ones |
| `#server-reference` | EC2, Cloudflare, Capacitor, Play Store details |
| `#session-briefing` | Cross-session state overview |
| `#slack-alert-processing` | Full protocol for processing Slack error alerts |
| `#task-completion-priority` | Finish current work before switching, atomic edits |
| `#user-experience-first` | Form validation UX patterns and checklist |
| `#verify-dont-ask` | Investigate with tools, don't ask questions you can answer |
| `#writing-hooks-and-steering` | How to write hooks, Hook Quality Rules, choosing mechanism |

## Hooks (3 total)

| Hook | Trigger | Purpose |
|------|---------|---------|
| `understand-then-act` | UserPromptSubmit | 10-step universal thinking framework |
| `end-of-task` | Stop | Verify completeness, resolve alerts, update tasks, debrief, surface opportunities |
| `no-local-formatcurrency` | PostFileSave | Prevent regression of centralized utility pattern |
