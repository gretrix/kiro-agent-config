# Kiro Agent Configuration

This repository contains my AI agent configuration for the [Kiro IDE](https://kiro.dev) — hooks, steering rules, spec examples, and session coordination files developed over ~2 months of daily use on a production Next.js SaaS application.

## What This Is

Kiro is an AI-powered development environment. Its power comes from **persistent behavioral configuration** — files that shape how the AI agent thinks, verifies, and coordinates across sessions. This repo shows the system I've built:

- **Hooks** fire at specific trigger points (before every prompt, after every task) to enforce non-negotiable thinking patterns
- **Steering** provides domain knowledge and principles that load contextually based on what files are open
- **Specs** formalize feature development through structured requirements → design → implementation tasks
- **Snapshots** coordinate work across parallel AI sessions (preventing duplicate work, tracking deploy state)

## Why This Exists

Over hundreds of sessions, I iterated these files from scratch. The CHANGELOG in `.kiro/hooks/CHANGELOG.md` documents every evolution — what failed, what worked, and why. Key lessons:

1. **Hooks should encode universal thinking patterns**, not task-specific scripts. "Verify in real conditions" works for everything; "if deploying, check X" creates escape hatches.
2. **Steering should teach judgment**, not replace it. FileMatch-based loading means docs only appear when relevant.
3. **The agent needs to verify completeness, not just presence.** "The API returned data" is not enough — "the API returned the correct, complete data that the user will actually see" is the bar.

## Repository Structure

```
.kiro/
├── hooks/                    # Behavioral enforcement (fire at trigger points)
│   ├── understand-then-act.json   # 10-step pre-prompt thinking framework
│   ├── end-of-task.json           # Verification + debrief on task completion
│   ├── no-local-formatcurrency.json  # Code guard (PostFileSave)
│   └── CHANGELOG.md               # Full evolution history
├── steering/                 # Domain knowledge & principles
│   ├── README.md                  # Inventory with inclusion modes
│   ├── writing-hooks-and-steering.md  # Meta-guide for authoring these files
│   ├── verify-dont-ask.md         # Core principle: investigate, don't ask
│   ├── human-centered-design.md   # UX validation framework
│   ├── infrastructure-safety.md   # Blue-green deploy safety rules
│   ├── ... (33 files total)
│   └── local-dev.md              # Local environment config (always-on)
├── settings/
│   ├── mcp.json.example          # MCP server config template
│   ├── lsp.json                  # Language server settings
│   └── README.md                 # Setup instructions
├── specs/                    # Feature spec examples
│   ├── smart-lists/              # Full example: requirements → design → tasks
│   └── code-reusability-audit/   # Requirements-only example
└── DRY_QUICK_REFERENCE.md   # Reuse-first checklist

.snapshots/                   # Cross-session coordination
├── readme.md                 # How snapshots work
└── config.json               # Snapshot configuration

AGENTS.md                     # Root agent guidance (signal priority, key files)
```

## How It Works Together

```
User sends a prompt
        │
        ▼
┌─────────────────────────────┐
│  understand-then-act (hook) │  ← Fires BEFORE any work begins
│  10 checks: facts first,   │
│  reference docs, root cause,│
│  full scope, verify, etc.   │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Steering (contextual)      │  ← Loaded based on open files
│  FileMatch: deploy safety,  │
│  API docs, job reference    │
│  Manual: #decision-log, etc │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Agent does the work        │
│  (with principles in mind)  │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  end-of-task (hook)         │  ← Fires AFTER work completes
│  Confidence meter, update   │
│  tasks, debrief, surface    │
│  opportunities              │
└─────────────────────────────┘
```

## Key Design Principles

These emerged from trial and error (documented in the CHANGELOG):

| Principle | Why |
|-----------|-----|
| Universal hooks, conditional steering | Hooks fire every time — they must help every prompt type. Domain-specific guidance belongs in steering with fileMatch triggers. |
| Principles over scripts | "Observation beats inference" covers every case. "If user reports broken, use Playwright to visit..." creates loopholes. |
| No conditionals in hooks | "Skip for simple questions" defeats the purpose. If it fires, it must be useful. |
| Reference, don't embed | Hooks point to docs. Steering carries detail. Keeps hooks short and focused. |
| Verify completeness, not presence | "API returned 200" is not success. "User sees the correct data" is success. |
| Claim your work | Multi-session coordination via snapshots prevents duplicate effort. |

## Inclusion Modes (Steering)

| Mode | When Loaded | Use Case |
|------|-------------|----------|
| `always` | Every conversation | Universal rules (port config, environment) |
| `fileMatch` | When matching files are open | Domain docs (API reference when editing API code) |
| `manual` | User types `#filename` in chat | Large references loaded on demand |

## Sensitive Data

This is a sanitized export. Files that originally contained API tokens, SSH keys, IP addresses, or database credentials have been redacted with `<PLACEHOLDER>` values. The patterns and principles are preserved — only the specific secrets are removed.

Real credentials live in `.env.local` and gitignored config files, never in steering or hooks.

## Context

- **Project type:** Next.js 14 SaaS (App Router, TypeScript, MySQL, Tailwind)
- **Infrastructure:** Single EC2 with blue-green deploys, Cloudflare DNS, PM2 process management
- **Team size:** Solo developer + AI agent (me + Kiro)
- **Duration:** ~2 months of iteration on these agent files
- **Sessions:** Hundreds of AI sessions, each building on the last

## License

MIT — use any of this however you like.
