---
inclusion: manual
---
# Verify, Don't Ask

## Principle
When the user provides evidence of a problem — a URL, an error message, a screenshot, a domain name — use your tools to investigate directly. Don't ask the user to clarify what environment they're in, what they see, or whether something is deployed. If they gave you a prod URL, check prod. If they gave you a local URL, check local.

This principle applies equally to YOUR OWN assumptions. Before writing code, before choosing an approach, before claiming something will or won't work — verify it with the smallest possible real-world probe. Make one API call. Check one DB row. Load one page. Read one response. The cost of a 30-second verification is always less than the cost of building on a wrong assumption.

You have SSH access, database access, API access, and web fetch. Use them. The user should never have to answer a question you could answer yourself with a tool call. And YOU should never write 200 lines of code based on something you haven't confirmed with a 1-line test.

## Before proposing new features or architecture
Before suggesting "we need to build X," grep the codebase for X. It probably already exists. This project has partner signup flows, CRM systems, credit billing, notification engines, and marketing tools already built. SEARCH FIRST — then extend what exists rather than proposing from scratch.

Specifically: before designing any new system, run `file_search` and `grep_search` for the nouns in your proposal. "Partner signup" → search for `partner/signup`. "Credit billing" → search for `creditService`. "Email notifications" → search for `email` in lib/. The answer to "do we have this?" is always a tool call, never a guess.

## No Speculation Loops
If you're uncertain whether something works — STOP REASONING and GO TEST IT. One Playwright run, one curl, one DB query is worth more than ten paragraphs of "maybe it's X." The pattern to avoid:

- ❌ "Maybe the issue is..." → think → "Or maybe..." → think → "Let me check if..." → read more code → "Actually maybe..."
- ✅ "I'm not sure if the report renders. Let me submit the funnel with Playwright and see."

The rule: **one round of hypothesis is allowed. If you're still uncertain after that, execute the user flow and observe the result.** Never speculate twice about the same thing.

## Verify COMPLETENESS, Not Just Presence (UNIVERSAL RULE)

**The error to avoid:** Seeing SOME evidence of success and declaring victory without checking if the outcome is COMPLETE.

This is the single most dangerous agent error pattern. It applies to EVERYTHING:

| Situation | Presence (WRONG) | Completeness (RIGHT) |
|-----------|-------------------|---------------------|
| Scraper | "Found 50 notices" | "Found 50 of 1000. Only page 1 of 20 processed." |
| Deploy | "CI says success" | "CI succeeded AND the live URL serves the new code" |
| API fix | "Returns 200" | "Returns 200 with the correct response body shape" |
| Feature | "No TypeScript errors" | "No errors AND the user flow produces the expected result" |
| DB migration | "Ran without error" | "Table exists AND has the expected columns AND the app reads it correctly" |
| Bug fix | "Error is gone" | "Error is gone AND the original user flow now works end-to-end" |

**The question to ALWAYS ask before declaring anything "done" or "working":**

> "What does COMPLETE look like for this specific thing — and have I verified THAT, not just that something happened?"

**Operationalized rules:**
1. When checking a batch/paginated system: ask "how many TOTAL items exist?" and compare to what was processed
2. When checking a deploy: visit the actual URL and verify the specific change is visible
3. When checking a fix: reproduce the ORIGINAL failure scenario and confirm it now succeeds
4. When checking a feature: exercise the full happy path, not just one step
5. When a result contains a count: ask "is this count reasonable?" (50 notices in 159 counties = ~0.3 per county = suspicious)
6. When a status says "partial" or "in progress": that is NOT a success — investigate WHY it's partial

**The anti-pattern this prevents:**
- ❌ "The API returned data" → success!
- ✅ "The API returned data. Is this ALL the data? Is the shape correct? Does the UI render it? Does the user see what they expect?"

This rule overrides the desire to move fast. Taking 30 seconds to check completeness saves hours of debugging later.

## Before writing code that crosses execution boundaries
When code you write will execute in a DIFFERENT context than where you're thinking about it — serialization (JSON.stringify), different process (webhook/cron), deferred execution (setTimeout/Promise), different runtime (browser vs server) — write a 1-line probe FIRST that confirms the value arrives in the expected shape. Async values don't survive JSON.stringify. Server-only code doesn't run in browsers. Environment variables don't exist in all contexts. Verify the boundary, then build on it.


## Fix-Then-Verify Discipline (for ANY fix or feature)

Before writing ANY fix or implementing ANY feature, state these THREE things explicitly in your response:

1. **Success criteria** (one sentence): What specific, measurable outcome proves this works?
2. **Verification command** (one line): The exact DB query, curl, or Playwright command that will confirm it.
3. **Failure response** (one sentence): If verification fails, what's the alternative approach?

Then — and ONLY then — write the fix. After deploying, run the verification command. If it passes, you're done. If it fails, execute the failure response. Do NOT add incremental patches.

**Example:**
- Success: "Scrape run reaches page 3+ with new_records > 0"
- Verify: `SELECT last_page_reached, new_records FROM scrape_runs WHERE id = (SELECT MAX(id) FROM scrape_runs)`
- If fail: "Switch to direct connection permanently, abandon proxy"

This prevents the pattern of: fix → partial signal → declare success → user finds it's broken → another fix → repeat.
