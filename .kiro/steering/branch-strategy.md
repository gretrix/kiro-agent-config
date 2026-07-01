---
inclusion: manual
---

# Branch Strategy (Optional — Per-Session Isolation)

## When to Use Session Branches
Use a dedicated branch when:
- Working on a feature that will span multiple turns and you don't want it to auto-deploy mid-work
- Making risky changes you want CI to validate before they hit production
- Multiple sessions are doing conflicting work on the same files

## When to Push Directly to Working Branch
Push directly to `main` when:
- Quick fix that should ship immediately
- Single-session work that's complete and tested
- You want auto-deploy (push → CI → deploy)

## Branch Naming Convention
```
session/<topic>
```
Examples:
- `session/ai-site-builder`
- `session/fix-scraper-timeout`
- `session/crm-pipeline-v2`

## Workflow with Session Branches

1. **Start:** `git checkout -b session/<topic>`
2. **Work:** Commit as you go. Push to trigger CI (typecheck + tests run on PR).
3. **PR:** Open a PR against the working branch when ready.
4. **Auto-merge:** If CI passes, `session/*` PRs are automatically squash-merged and the branch is deleted. No manual merge step needed.
5. **Deploy:** The merge commit triggers auto-deploy via CI.

### Auto-Merge Rules
- Only `session/*` branches are auto-merged. Other PRs (feature branches, external contributors) require manual merge.
- The merge uses squash (single clean commit on target branch).
- The session branch is auto-deleted after merge.
- If you DON'T want auto-merge for a session PR (e.g., you want manual review), rename the branch to `draft/<topic>` instead.

## CI Behavior by Branch Type

| Branch | Typecheck | Tests | Auto-Merge | Deploy |
|--------|-----------|-------|------------|--------|
| `main` | ✅ | ✅ | — | ✅ Auto |
| `feature/*` | ✅ | ✅ | — | ✅ Auto |
| PR from `session/*` | ✅ | ✅ | ✅ Auto squash | On merge |
| PR from other | ✅ | ✅ | ❌ Manual | On merge |

## How This Helps Multi-Session Work
- Session A on `session/crm-v2` can push freely without triggering deploys
- Session B on the working branch still auto-deploys on push
- No risk of session A's half-finished code deploying because session B pushed
- When session A is done → merge PR → auto-deploy includes everything
- Branch protection ensures nothing merges without passing typecheck + regression

## Branch Protection (Live)
`main` has GitHub branch protection:
- **Required checks:** `typecheck` + `regression` must pass before merge
- **Force push:** disabled (prevents history rewriting on shared branches)
- **Admin bypass:** allowed (so you can force-merge if CI is broken/flaky)
- **PR reviews:** NOT required (auto-merge handles session branches without human review)

To reconfigure: `bash scripts/setup-branch-protection.sh`

## Integration with deploy-pending.md
- Sessions pushing to `main`/working branch: log to `deploy-pending.md` (CI auto-deploys)
- Sessions on session branches: no need to log until PR is merged (deploy happens on merge)
