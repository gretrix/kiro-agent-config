---
inclusion: manual
---
# Multi-Session Safety

## Problem
Multiple Kiro chat windows may be working on the same repo simultaneously. If one session does `git add -A && git commit`, it picks up files from other sessions that may be incomplete or untested.

## Rules

### 1. NEVER use `git add -A` or `git add .`
Always stage specific files that YOU modified in this session. Use `git add <file1> <file2> ...` with explicit paths.

### 2. Before committing, check for unexpected changes
Run `git status --short` and review the list. If you see files you didn't modify (e.g., `android/`, `ios/`, `capacitor.config.ts`, files in areas you didn't touch), those are from another session. Do NOT stage them.

### 3. Before deploying, verify git diff makes sense
Run `git diff --staged --stat` and confirm every file is something you changed. If the diff includes 50+ files when you only changed 5, something is wrong.

### 4. Use .snapshots/active-tasks.md as a coordination file
When starting work, check this file. If another session's tasks are listed as "in progress," be aware that files related to those tasks may have unstaged changes in the working directory.

### 5. If you're unsure, ask before committing
If `git status` shows changes you don't recognize, tell the user: "I see changes to [files] that I didn't make — these look like they're from another session. Should I include them in this commit or leave them for the other session?"

### 6. Prefer session branches for distinct work streams
When starting a new feature or fixing an unrelated issue, create a dedicated branch:
```
git checkout -b session/<short-name>
```
This isolates changes completely. Other sessions on other branches can't interfere. Push triggers CI checks (typecheck + tests) but NOT deployment. Merge via PR when the feature is complete and tested — the merge triggers auto-deploy.

Use `main` for quick fixes that should auto-deploy immediately. Use `session/*` branches for larger features or when you don't want half-finished work deploying.

See `.kiro/steering/branch-strategy.md` (manual inclusion) for the full branch strategy doc.

## The Key Insight
`git add -A` is convenient but dangerous in a multi-session workflow. It's the equivalent of "commit everything in the repo" — including half-finished work from other windows. Always be explicit about what you're staging.
