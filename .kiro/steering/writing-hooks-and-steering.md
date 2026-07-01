---
inclusion: manual
---

# How to Write Hooks and Steering Rules

## Principle
Hooks and steering teach HOW to think, not WHAT to do. They are principles, not if/then rules. They must work for every scenario, not just the one that triggered their creation.

## Before Writing Any Hook or Steering Rule

**Always propose changes to the user first.** Present the content you want to write and wait for explicit approval before creating or modifying any file in `.kiro/hooks/` or `.kiro/steering/`. These files control the agent's behavior — the human retains approval authority over them.

Ask:
1. What THINKING PATTERN am I trying to encourage? (not: what SPECIFIC ACTION in what SPECIFIC SCENARIO)
2. If a completely different situation came up tomorrow, would this rule still apply and still help?
3. Am I writing a principle or a checklist? (Principles scale. Checklists don't.)
4. Is this the simplest, most universal version of what I'm trying to say?

## Bad vs Good Examples

**Bad (rigid, scenario-specific):**
> "If the user says 'don't fix this', then don't fix it. If the user says 'X overlaps Y', then check all elements."

**Good (principle that works universally):**
> "Understand what the person actually wants before acting. The answer is in their words, not in assumptions."

**Bad:**
> "Test from fortuneleo.com, dealsidekick.com, and chratl.com. Test starter, pro, and elite plans."

**Good:**
> "Validate like a real user — different people use this differently. Test the variations that exist, not just one path."

## The Test

Before committing a hook or steering rule, ask:
- Does it read like wisdom or like a script?
- Would it help me handle a situation I haven't thought of yet?
- Is it teaching judgment or replacing it?

If it's replacing judgment with a script — rewrite it as a principle.

## Hook Quality Rules

- **Universal or nothing.** Every check in a hook must apply to ALL prompts — coding, design, screenshots, docs, conversations, everything. If a check only makes sense for one type of work, it belongs in conditional steering (fileMatch), not a hook.

- **No conditionals.** No "if this is a large task," no "if fixing something," no "skip for simple questions." A hook fires on every trigger. If the content shouldn't run every time, it shouldn't be in the hook.

- **Checks stack.** Order matters. Earlier checks provide the foundation later checks build on. Put foundational thinking (gather facts, don't guess) before derived thinking (define scope, plan solution).

- **Scope means empathy.** Understanding what the user wants means thinking like a human would — what would they notice? What's adjacent? What does "fully satisfied" look like from their perspective? Not a technical summary, but genuine comprehension of intent and context.

- **Verify universally.** Post-solution verification applies to everything — code, images, UI, docs, architecture. "Did this actually solve it in real conditions?" is never limited to one medium.

- **Reference, don't embed.** When certain task types need specific context, the hook should point to docs — not contain the instructions. Keep hooks short; let steering carry the detail.

- **Apply this guide as a filter.** When writing, merging, moving, or consolidating hook text, every line must pass these rules — including text that "already exists." If existing text wouldn't pass, rewrite it as a principle before proposing it.

## Choosing the Right Mechanism

Not everything belongs in the same place. Use this decision tree:

### Always-on Steering (`inclusion: always` or no frontmatter)
**Use for:** Universal behavioral principles that apply to EVERY session regardless of topic.
**Examples:** "verify before claiming," "complete tasks before moving on," "reuse before rebuilding"
**Rule of thumb:** If removing this would cause bad behavior in >50% of sessions, it's always-on.
**Keep it short.** Every word here is loaded into every conversation. If it's longer than ~30 lines, split it.

### Conditional Steering (`inclusion: fileMatch`)
**Use for:** Reference data that's only relevant when working in a specific domain.
**Examples:** API endpoint docs (when touching API clients), deploy architecture (when touching deploy scripts), data source inventory (when touching scraper code)
**Rule of thumb:** If this content is only needed when specific files are open, use fileMatch.
**Pattern:** `fileMatchPattern: '**/lib/apivex/**,**/services/scraper/**'`

### Manual Steering (`inclusion: manual`)
**Use for:** Large reference documents, infrequent workflows, or context that the user explicitly wants to load.
**Examples:** Decision log, product overview, server reference, landing page framework
**Rule of thumb:** If the user would say "I need this context for THIS task" but not for the next 5 tasks, it's manual.
**Access:** User types `#filename` (without .md) in chat to load it.

### Hooks (preToolUse, promptSubmit, agentStop, etc.)
**Use for:** Enforcement of specific behaviors at specific moments — things that MUST happen regardless of whether the agent "remembers" the steering.
**Examples:** "Block destructive SSH commands" (preToolUse), "check Slack alerts" (promptSubmit), "verify deployment" (agentStop)
**Rule of thumb:** If the behavior is critical enough that "forgetting" it causes outages or user frustration, enforce it with a hook. If it's guidance that helps but isn't catastrophic to miss, steering is sufficient.

### Key Differences

| Aspect | Steering | Hook |
|--------|----------|------|
| When loaded | Start of conversation | At trigger moment |
| Can be ignored | Yes (agent may deprioritize) | Harder to ignore (explicit prompt injection) |
| Cost | Constant context budget | Per-trigger context budget |
| Best for | Shaping judgment | Enforcing actions |
| Failure mode | Agent forgets under pressure | Agent gives rote response to frequent triggers |

### Anti-Patterns

- **Steering that duplicates a hook** — If you have always-on steering saying "always deploy" AND a hook enforcing it, the steering just wastes tokens. Let the hook enforce; use steering only for the WHY/context.
- **Hook with >20 lines** — Long hooks get partially ignored. If the instruction is complex, put the details in a steering file and have the hook reference it: "See #slack-alert-processing for the full protocol."
- **Always-on steering for rare workflows** — If it's only relevant 10% of the time, it's wasting 90% of sessions' context budget. Use fileMatch or manual.
- **Hook that fires too often** — A hook that triggers on every write/shell/read trains the agent to dismiss it reflexively. Narrow the trigger (regex patterns, specific toolTypes) so it only fires when genuinely relevant.

## Worked example: encoding a *mindset* (not a task)

When friction comes from **misaligned meaning across layers** (e.g. HTTP 200 + `success: true` but `results: null`), the durable fix is usually **judgment**, not a funnel-specific hook. That is what **manual steering** is for: short principles that apply to every similar case. See **`#cross-boundary-verification`** → `.kiro/steering/cross-boundary-verification.md`. The `understand-then-act` hook's scope/verify checks surface this habit on every prompt without loading the full file every time.
