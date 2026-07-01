---
description: Ensures complex multi-file tasks are delegated to kiro-cli instead of done inline
inclusion: auto
---

# Delegation Pattern

## Rule
When executing multi-file tasks (creating components, implementing features, building pages), ALWAYS delegate to kiro-cli instead of doing the work inline.

## Why
- Avoids "Too many requests" rate limit errors on sub-agents
- kiro-cli has its own execution context and rate limit handling
- Keeps the main conversation focused on orchestration, not implementation details
- Allows retry logic on failures

## When to use kiro-cli
- Creating new files with more than ~30 lines of code
- Tasks that touch 2+ files
- Building UI components/pages
- Implementing API routes with complex logic
- Any task from a spec's tasks.md
- Writing steering rules or config files with substantial content
- ANY work that involves more than 2-3 tool calls in sequence

## When to do directly
- Reading a single file (one readFile call)
- A single strReplace edit
- Running one build/test command
- Checking diagnostics
- Updating task status
- Asking the user a question

## Key Insight
The CLI has its own separate rate limit pool. It almost never hits rate limits. The main Kiro IDE session hits rate limits much faster because every tool call counts against it. When in doubt, route through the CLI.

## Command Pattern
```powershell
$env:PATH = [System.Environment]::GetEnvironmentVariable('PATH', 'Machine') + ';' + [System.Environment]::GetEnvironmentVariable('PATH', 'User')
kiro-cli chat --no-interactive --trust-all-tools "<clear instruction>"
```

- Always use `executePwsh` with `timeout: 300000` and `cwd` set to project root
- If output contains "Too many requests" or "rate limit", wait 15 seconds and retry up to 3 times
- Review CLI output and report results back to user
