# Implementation Plan: Smart Lists

## Overview

Extend the existing Lead Lists feature with a "smart" list type that auto-populates based on saved filter criteria. Implementation follows dependency order: database schema extension → filter matcher service → auto-population service → API route extensions → scheduler post-scrape hook → UI updates. All work builds on existing `lead_lists` infrastructure — no replacement, only extension.

## Tasks

- [x] 1. Database migration
  - [x] 1.1 Create smart lists migration file
    - Create `lib/db/migrations/smart-lists.sql`
    - ALTER `lead_lists`: add `list_type ENUM('manual','smart') NOT NULL DEFAULT 'manual'` after `name`
    - ALTER `lead_lists`: add `filter_criteria JSON DEFAULT NULL` after `list_type`
    - Backfill existing rows: `UPDATE lead_lists SET list_type = 'manual' WHERE list_type = 'manual'`
    - CREATE TABLE `smart_list_exclusions` with `id`, `list_id`, `record_id`, `excluded_at`, unique constraint `(list_id, record_id)`, FK cascades to `lead_lists` and `foreclosure_records`
    - ADD INDEX `idx_list_type` on `lead_lists(list_type)` for efficient smart list lookups
    - _Requirements: 5.1, 5.2, 5.3, 5.4_

- [x] 2. Filter Matcher service
  - [x] 2.1 Implement filterMatcher.ts
    - Create `lib/smartLists/filterMatcher.ts`
    - Define `SmartListFilterCriteria` interface (counties, noticeTypes, minLien, maxLien, auctionDateFrom, auctionDateTo, minDealScore, minEquityPotential)
    - Define `FilterQuery` interface (whereClause, params)
    - Implement `buildFilterQuery(criteria)` — builds parameterized SQL WHERE clause with AND logic across all specified fields, unspecified fields ignored
    - Implement `hasAtLeastOneCriterion(criteria)` — returns true if at least one field is non-empty
    - Implement `validateFilterCriteria(criteria)` — validates types, ranges, date formats; returns array of error messages
    - Counties match case-insensitive, lien range inclusive, dates inclusive, scores use >= comparison
    - _Requirements: 1.6, 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8_

  - [x]* 2.2 Write property test for filter matching AND logic
    - **Property 1: Filter Matching Correctness (AND Logic)**
    - Generate random `SmartListFilterCriteria` + random foreclosure record data → verify a record matches iff it satisfies every specified criterion
    - **Validates: Requirements 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8**

- [x] 3. Auto-Population service
  - [x] 3.1 Implement autoPopulationService.ts
    - Create `lib/smartLists/autoPopulationService.ts`
    - Implement `populateSmartList(listId, criteria)` — queries all matching foreclosure_records via `buildFilterQuery`, inserts into `lead_list_memberships` with INSERT IGNORE, respects 1000-member cap, returns `PopulationResult { listId, added, skipped, atCapacity }`
    - Implement `evaluateNewRecords(newRecordIds)` — loads all smart lists, for each list builds filter query scoped to new IDs, checks exclusion set + existing memberships + capacity, inserts matches; returns `{ listsProcessed, totalAdded, errors }`
    - Implement `repopulateSmartList(listId, criteria)` — deletes all existing memberships for the list, then runs fresh population (called after exclusions are cleared by caller)
    - Single transaction per smart list during batch evaluation (isolate failures per Requirement 3.6)
    - Log warnings for lists at capacity
    - _Requirements: 1.4, 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 9.2_

  - [x]* 3.2 Write property test for exclusion prevents addition
    - **Property 2: Exclusion Prevents Addition**
    - Generate matching records + exclusion sets → verify excluded records never added to membership
    - **Validates: Requirements 3.4, 4.2, 4.3**

  - [x]* 3.3 Write property test for auto-population idempotency
    - **Property 8: Auto-Population Idempotency**
    - Run population twice with same input → verify no duplicate memberships
    - **Validates: Requirements 3.5**

  - [x]* 3.4 Write property test for capacity enforcement
    - **Property 10: Capacity Enforcement**
    - Generate list at 1000 members + matching new records → verify no additions
    - **Validates: Requirements 9.2**

  - [x]* 3.5 Write property test for multi-list matching
    - **Property 11: Multi-List Matching**
    - Generate a record matching N smart lists → verify record added to all N
    - **Validates: Requirements 3.3**

- [x] 4. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Lead List query extensions
  - [x] 5.1 Extend leadListQueries.ts for smart lists
    - Extend `LeadListRow` interface to include `list_type` and `filter_criteria` fields
    - Update `getListsByUserId` query to SELECT `list_type` and `filter_criteria`
    - Implement `createSmartList(userId, name, criteria)` — validates name + criteria, inserts with `list_type='smart'`, calls `populateSmartList`, returns `{ id, name, populationResult }`
    - Implement `updateSmartListCriteria(listId, userId, criteria)` — validates criteria, updates `filter_criteria`, clears exclusions, deletes old memberships, calls `repopulateSmartList`
    - Implement `removeSmartListMember(listId, recordId, userId)` — deletes membership AND inserts into `smart_list_exclusions`
    - Implement `deleteSmartList(listId, userId)` — deletes list (CASCADE handles memberships + exclusions)
    - Implement `getAllSmartLists()` — returns all smart lists with filter_criteria (for scheduler hook)
    - Implement `getExclusions(listId)` — returns Set of excluded record IDs
    - Implement `clearExclusions(listId)` — deletes all exclusion rows for a list
    - Ensure existing `removeMember` is unaffected for manual lists (no exclusion side effect)
    - _Requirements: 1.2, 1.3, 1.5, 4.1, 4.4, 5.5, 6.1, 6.2, 6.3, 6.4, 8.2_

  - [x]* 5.2 Write property test for removal creates exclusion
    - **Property 3: Removal from Smart List Creates Exclusion**
    - Remove member from smart list → verify both membership deleted AND exclusion inserted
    - **Validates: Requirements 4.1**

  - [x]* 5.3 Write property test for manual list removal has no exclusion
    - **Property 4: Manual List Removal Has No Exclusion Side Effect**
    - Remove member from manual list → verify no exclusion row created
    - **Validates: Requirements 4.4**

  - [x]* 5.4 Write property test for re-evaluation produces correct membership
    - **Property 5: Re-Evaluation Produces Correct Membership Set**
    - Edit criteria → verify membership = exact match set of new criteria (no leftovers)
    - **Validates: Requirements 6.1, 6.2**

  - [x]* 5.5 Write property test for criteria edit clears exclusions
    - **Property 6: Criteria Edit Clears Exclusions**
    - Generate list with exclusions, edit criteria → verify exclusions empty
    - **Validates: Requirements 6.3**

  - [x]* 5.6 Write property test for rename does not trigger re-evaluation
    - **Property 7: Rename Does Not Trigger Re-Evaluation**
    - Rename smart list → verify memberships unchanged and exclusions unchanged
    - **Validates: Requirements 6.4**

  - [x]* 5.7 Write property test for cascade delete preserves records
    - **Property 9: Cascade Delete Preserves Foreclosure Records**
    - Delete smart list → verify foreclosure records still exist in DB
    - **Validates: Requirements 8.2**

- [x] 6. API route extensions
  - [x] 6.1 Extend POST /api/lead-lists for smart list creation
    - Modify `app/api/lead-lists/route.ts` POST handler to accept optional `listType: 'smart'` and `filterCriteria` fields
    - When `listType === 'smart'`: validate criteria via `validateFilterCriteria` + `hasAtLeastOneCriterion`, call `createSmartList`, return 201 with population result
    - When `listType` is omitted or `'manual'`: existing behavior unchanged
    - Return 422 with `INVALID_CRITERIA` error code for bad criteria
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6_

  - [x] 6.2 Extend GET /api/lead-lists to return smart list fields
    - Update `app/api/lead-lists/route.ts` GET handler to include `list_type` and `filter_criteria` in response
    - Existing consumers get new fields but remain backward-compatible (manual lists have `filter_criteria: null`)
    - _Requirements: 5.5, 7.1_

  - [x] 6.3 Extend PATCH /api/lead-lists/[id] for criteria updates
    - Modify `app/api/lead-lists/[id]/route.ts` PATCH handler to accept `filterCriteria` field
    - When list is smart and `filterCriteria` is provided: call `updateSmartListCriteria`, return population result
    - When only `name` is provided: rename only (no re-evaluation, per Requirement 6.4)
    - Reject criteria update on manual lists with 422
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 6.4 Extend DELETE /api/lead-lists/[id]/members for exclusion tracking
    - Modify the member removal handler in `app/api/lead-lists/[id]/members/route.ts`
    - Check list_type: if smart, call `removeSmartListMember` (adds exclusion); if manual, use existing `removeMember`
    - _Requirements: 4.1, 4.2, 4.3, 4.4_

- [x] 7. Scheduler post-scrape hook
  - [x] 7.1 Add smart list evaluation to post-scrape flow
    - Modify `services/scraper/scheduler.ts` — in `executeJob` (after `runScrape` completes), query new record IDs from the completed scrape run
    - Import and call `evaluateNewRecords(newRecordIds)` from `lib/smartLists/autoPopulationService`
    - Log results: lists processed, total records added, errors
    - Ensure hook runs as fire-and-forget (does not block scheduler state cleanup)
    - Handle errors gracefully — log and continue, never crash the scheduler
    - _Requirements: 3.1, 3.2, 3.6, 3.7, 8.3, 9.1_

- [x] 8. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 9. UI — Smart list creation flow
  - [x] 9.1 Add Smart List option to MyListsPanel create flow
    - Modify `app/components/notices/MyListsPanel.tsx` to show Manual vs Smart choice when user clicks "Create List"
    - For Smart: show form with name field + filter criteria fields (counties multi-select, notice types multi-select, min/max lien inputs, date from/to pickers, min deal score slider, min equity potential slider)
    - On submit: call POST `/api/lead-lists` with `listType: 'smart'` + `filterCriteria`
    - Show population result count after creation
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 7.7_

  - [x] 9.2 Add ⚡ icon and filter summary tags for smart lists
    - Update `MyListsPanel` list rendering to show ⚡ icon next to smart list names
    - When a smart list is selected, display filter criteria as read-only summary tags below the list header (e.g., "Fulton, DeKalb", "Lien: $50K–$500K", "Score ≥ 60")
    - Hide "Add to List" button on NoticeCards when the active view is a smart list
    - Show capacity indicator when list has 1000 members
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 9.3_

- [x] 10. UI — Smart list editing and deletion
  - [x] 10.1 Implement edit smart list filter criteria dialog
    - Add "Edit Filters" action to smart list context menu in MyListsPanel
    - Open dialog pre-populated with current filter criteria
    - On save: call PATCH `/api/lead-lists/[id]` with new `filterCriteria`
    - Show re-evaluation result (new member count)
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 10.2 Extend delete confirmation for smart lists
    - Ensure the existing delete confirmation dialog works for smart lists
    - Auto-population stops after deletion (no scheduler evaluation)
    - _Requirements: 8.1, 8.2, 8.3_

- [x] 11. Final checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties from the design document (fast-check)
- The migration file auto-runs on next deploy via the blue-green deploy script's migration step
- All API route extensions are backward-compatible — existing manual list behavior is unchanged
- The scheduler hook follows the same pattern as `checkSavedSearchNotifications` (fire-and-forget after scrape)
- `INSERT IGNORE` ensures idempotent membership operations across all population paths

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["2.1"] },
    { "id": 2, "tasks": ["2.2", "3.1"] },
    { "id": 3, "tasks": ["3.2", "3.3", "3.4", "3.5", "5.1"] },
    { "id": 4, "tasks": ["5.2", "5.3", "5.4", "5.5", "5.6", "5.7"] },
    { "id": 5, "tasks": ["6.1", "6.2", "6.3", "6.4"] },
    { "id": 6, "tasks": ["7.1"] },
    { "id": 7, "tasks": ["9.1"] },
    { "id": 8, "tasks": ["9.2", "10.1", "10.2"] }
  ]
}
```
