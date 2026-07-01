# Requirements Document

## Introduction

Smart Lists extends the existing Lead Lists feature to support auto-populating lists based on saved filter criteria. Users define a set of filter conditions (county, notice type, lien range, date range, deal score minimum, equity potential minimum) when creating a list, and the system automatically adds matching foreclosure records — both existing and newly imported. This transforms lists from static organizational tools into "living" deal pipelines that grow as new data arrives from the scheduler scraper.

The implementation builds on the existing `lead_lists` and `lead_list_memberships` tables, the `SavedSearchFilters` pattern from `lib/savedSearches/`, and the notification checker cron pattern in the scheduler.

## Glossary

- **Smart_List**: A Lead_List with `list_type = 'smart'` that has associated filter criteria and auto-populates with matching foreclosure records
- **Manual_List**: A Lead_List with `list_type = 'manual'` — the existing behavior where users explicitly add and remove leads
- **Filter_Criteria**: A JSON object stored on a smart list defining the matching conditions (counties, notice types, lien range, date range, deal score minimum, equity potential minimum)
- **Auto_Population_Service**: The backend service that evaluates smart list filter criteria against foreclosure records and adds matching records to the list
- **Exclusion_Set**: The set of record IDs that a user has manually removed from a smart list, preventing re-addition by the Auto_Population_Service
- **Lead_List_Service**: The existing backend service responsible for CRUD operations on lead lists and their memberships (extended for smart lists)
- **Scheduler**: The separate PM2 process (`fortuneleo-scheduler`) that imports foreclosure records from scraped sources on a recurring schedule
- **MyListsPanel**: The existing sidebar panel on the Notices_Page that displays the user's lists with counts and management actions

## Requirements

### Requirement 1: Smart List Creation

**User Story:** As a real estate investor, I want to create a list with filter criteria that automatically fills with matching leads, so that I can set my investment strategy once and have the system surface new deals for me.

#### Acceptance Criteria

1. WHEN the user selects "Create Smart List" in the MyListsPanel, THE Lead_List_Service SHALL present a form collecting a list name and filter criteria (counties, notice types, minimum lien amount, maximum lien amount, auction date from, auction date to, minimum deal score, minimum equity potential)
2. WHEN the user submits a valid smart list creation form, THE Lead_List_Service SHALL create a new lead list record with `list_type = 'smart'` and store the filter criteria as a JSON column on the `lead_lists` table
3. THE Lead_List_Service SHALL apply the same name validation rules to smart lists as manual lists: trimmed length between 1 and 100 characters, case-insensitive uniqueness per user, maximum of 50 total lists per user (manual and smart combined)
4. WHEN a smart list is created, THE Auto_Population_Service SHALL immediately evaluate the filter criteria against all existing foreclosure records and add matching records to the list
5. IF no existing foreclosure records match the filter criteria on initial creation, THEN THE Lead_List_Service SHALL create the list successfully and display it with a lead count of zero
6. THE Lead_List_Service SHALL require at least one filter criterion to be specified (the filter criteria object must not be entirely empty)

### Requirement 2: Filter Criteria Schema

**User Story:** As a real estate investor, I want to filter by the same criteria I use for saved searches, so that my smart list criteria match my existing investment strategy parameters.

#### Acceptance Criteria

1. THE Filter_Criteria SHALL support the following optional fields: counties (array of county names), noticeTypes (array of notice type identifiers), minLien (minimum lien amount as number), maxLien (maximum lien amount as number), auctionDateFrom (ISO date string), auctionDateTo (ISO date string), minDealScore (number 0-100), minEquityPotential (number representing minimum equity potential percentage)
2. WHEN the Filter_Criteria specifies counties, THE Auto_Population_Service SHALL match only foreclosure records where the county column matches one of the specified county names (case-insensitive)
3. WHEN the Filter_Criteria specifies noticeTypes, THE Auto_Population_Service SHALL match only foreclosure records where the notice_type column matches one of the specified types
4. WHEN the Filter_Criteria specifies minLien or maxLien, THE Auto_Population_Service SHALL match only foreclosure records where the lien_amount falls within the specified range (inclusive)
5. WHEN the Filter_Criteria specifies auctionDateFrom or auctionDateTo, THE Auto_Population_Service SHALL match only foreclosure records where the auction_date falls within the specified date range (inclusive)
6. WHEN the Filter_Criteria specifies minDealScore, THE Auto_Population_Service SHALL match only foreclosure records where the deal_score is greater than or equal to the specified minimum
7. WHEN the Filter_Criteria specifies minEquityPotential, THE Auto_Population_Service SHALL match only foreclosure records where the equity_potential is greater than or equal to the specified minimum
8. WHEN multiple filter criteria are specified, THE Auto_Population_Service SHALL apply all criteria with AND logic — a record must satisfy every specified criterion to be added to the list

### Requirement 3: Auto-Population on New Data Import

**User Story:** As a real estate investor, I want my smart lists to automatically grow when the scraper imports new foreclosure records that match my criteria, so that I never miss a deal that fits my strategy.

#### Acceptance Criteria

1. WHEN the Scheduler completes a scrape run and new foreclosure records are inserted, THE Auto_Population_Service SHALL evaluate each new record against all smart lists' filter criteria
2. WHEN a new foreclosure record matches a smart list's filter criteria, THE Auto_Population_Service SHALL add that record to the smart list's memberships
3. WHEN a new foreclosure record matches multiple smart lists' filter criteria, THE Auto_Population_Service SHALL add that record to all matching smart lists
4. THE Auto_Population_Service SHALL NOT add a record to a smart list if that record exists in the smart list's Exclusion_Set (previously manually removed by the user)
5. THE Auto_Population_Service SHALL NOT add a record to a smart list if that record is already a member of that list (no duplicate memberships)
6. IF the Auto_Population_Service encounters an error while processing a single smart list, THEN THE Auto_Population_Service SHALL log the error and continue processing remaining smart lists without interruption
7. THE Auto_Population_Service SHALL complete processing of all smart lists within 60 seconds for a batch of up to 500 new records across up to 200 smart lists

### Requirement 4: Manual Removal and Exclusion

**User Story:** As a real estate investor, I want to manually remove leads from a smart list without them being re-added on the next update, so that I can curate my auto-populated results.

#### Acceptance Criteria

1. WHEN the user removes a lead from a smart list, THE Lead_List_Service SHALL remove the membership AND add the record ID to the smart list's Exclusion_Set
2. THE Auto_Population_Service SHALL check the Exclusion_Set before adding any record to a smart list and SHALL NOT add excluded records
3. WHILE a record ID exists in a smart list's Exclusion_Set, THE Auto_Population_Service SHALL never add that record to that smart list regardless of filter criteria match
4. THE Lead_List_Service SHALL NOT apply exclusion behavior to manual lists — removal from a manual list remains a simple membership delete with no exclusion tracking

### Requirement 5: Schema Extension

**User Story:** As a developer, I want the database schema to support smart lists alongside existing manual lists, so that both list types coexist without breaking existing functionality.

#### Acceptance Criteria

1. THE Lead_List_Service SHALL add a `list_type` column to the `lead_lists` table with allowed values 'manual' and 'smart', defaulting to 'manual'
2. THE Lead_List_Service SHALL add a `filter_criteria` JSON column to the `lead_lists` table that stores the smart list's filter configuration, nullable for manual lists
3. THE Lead_List_Service SHALL add an `exclusions` table (or JSON column) to track record IDs that have been manually removed from each smart list
4. THE schema migration SHALL set `list_type = 'manual'` for all existing lead list records to preserve backward compatibility
5. THE Lead_List_Service SHALL ensure all existing API endpoints for manual lists continue to function without modification for lists with `list_type = 'manual'`

### Requirement 6: Smart List Editing

**User Story:** As a real estate investor, I want to update my smart list's filter criteria after creation, so that I can refine my strategy as market conditions change.

#### Acceptance Criteria

1. WHEN the user edits a smart list's filter criteria, THE Lead_List_Service SHALL update the stored filter_criteria JSON and trigger a full re-evaluation of existing foreclosure records against the new criteria
2. WHEN a full re-evaluation occurs, THE Auto_Population_Service SHALL add records that now match the updated criteria (and are not in the Exclusion_Set) and SHALL remove records that no longer match the updated criteria (unless they were manually added — not applicable since smart lists don't support manual add)
3. WHEN the user edits a smart list's filter criteria, THE Lead_List_Service SHALL clear the Exclusion_Set, since the filter scope has fundamentally changed
4. THE Lead_List_Service SHALL allow renaming a smart list without triggering a re-evaluation of filter criteria
5. IF the updated filter criteria match zero existing records, THEN THE Lead_List_Service SHALL update the criteria successfully and set the list's lead count to zero

### Requirement 7: UI Integration

**User Story:** As a real estate investor, I want smart lists to appear alongside my manual lists with a clear visual distinction, so that I can easily identify which lists are auto-updating.

#### Acceptance Criteria

1. THE MyListsPanel SHALL display smart lists and manual lists in the same alphabetically sorted list
2. THE MyListsPanel SHALL display a visual indicator (⚡ icon) next to smart list names to distinguish them from manual lists
3. WHEN the user selects a smart list, THE Notices_Page SHALL display the list's members using the same Map_View and Table_View as manual lists, with the same sorting, pagination, and filtering behavior
4. WHILE viewing a smart list, THE Notices_Page SHALL display the active filter criteria as read-only summary tags below the list header
5. WHILE viewing a smart list, THE user SHALL be able to remove individual leads (triggering exclusion behavior per Requirement 4)
6. WHILE viewing a smart list, THE "Add to List" button on NoticeCards SHALL NOT appear for the currently viewed smart list (smart lists do not support manual addition)
7. WHEN the user opens the "Create List" flow, THE Lead_List_Service SHALL present a choice between "Manual List" and "Smart List" creation modes

### Requirement 8: Smart List Deletion

**User Story:** As a real estate investor, I want to delete a smart list when I no longer need it, so that I can keep my workspace organized.

#### Acceptance Criteria

1. WHEN the user requests to delete a smart list, THE Notices_Page SHALL display a confirmation prompt before proceeding with deletion
2. WHEN the user confirms smart list deletion, THE Lead_List_Service SHALL remove the list, all its memberships, and its Exclusion_Set without deleting the underlying foreclosure records
3. WHEN a smart list is deleted, THE Auto_Population_Service SHALL no longer evaluate new records against that list's filter criteria

### Requirement 9: Performance and Limits

**User Story:** As a real estate investor, I want smart lists to update quickly without impacting the rest of the application, so that my workflow remains smooth.

#### Acceptance Criteria

1. THE Auto_Population_Service SHALL process auto-population as a background operation that does not block the Scheduler's import pipeline
2. THE Lead_List_Service SHALL enforce the existing maximum of 1000 leads per list for smart lists — when a smart list reaches capacity, THE Auto_Population_Service SHALL stop adding new records and SHALL log a warning
3. WHEN a smart list reaches the 1000-lead capacity, THE MyListsPanel SHALL display a visual indicator that the list is at capacity
4. THE initial population of a smart list on creation SHALL complete within 10 seconds for filter criteria that match up to 1000 records
5. THE Lead_List_Service SHALL support up to 50 total lists per user (manual and smart combined), with the same limit enforcement as existing manual lists
