---
name: boards-backlog-summary
description: Get the team's Requirements-level backlog (Product Backlog Items and Bugs) filtered to Active items assigned to the current user. Shows parent/child hierarchy and sorts by priority. Use this skill when the user wants to see their backlog items, sprint assignments, or work they own on a team board.
---

# Get team backlog summary

This skill retrieves the team's Requirements-level backlog, filters to **Active** items **assigned to the current user**, enriches them with parent and child work item details, and displays them sorted by priority.

## Project selection

- If the user **provides a project name** in their request (for example, "for Contoso"), use that project directly and **do not call** `core_list_projects`.
- If the project name is **not provided**, ask the user once to provide it.
- If the project name is **still not provided after asking once**, call `core_list_projects` to return a list of projects the user can choose from.
- Do not continue if no project has been provided or selected.

## Team selection

- If the user **provides a team name**, use it directly and **do not call** `core_list_project_teams`.
- If the team name is **not provided**, ask the user once to provide it.
- If the team name is **still not provided after asking once**, call `core_list_project_teams` for the selected project so the user can pick a team.
- Do not continue if no team has been provided or selected.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `core_list_projects`: Get the list of projects the user can choose from.
- `core_list_project_teams`: Get the list of teams for a project.
- `wit_backlog` (action: `list_work_items`): Get the work items on the team's backlog at a specific backlog level.
- `wit_work_item` (action: `get_batch`): Get work item details in batch by their IDs. Use this to fetch fields and relations for backlog items, their parents, and their children in as few calls as possible.

# Steps

1. Resolve project and team using the rules above.

2. Call `wit_backlog` with action `list_work_items` for the resolved project and team, using backlog level `Microsoft.RequirementCategory` (the Requirements / Product Backlog Items level).

3. From the returned work items, **filter** to items that meet **all** of the following criteria:
   - `System.State` = `Active`
   - `System.AssignedTo` matches the current user
   - `System.WorkItemType` is one of: `Product Backlog Item`, `Bug`

4. If no items match the filter, display a message stating there are no active backlog items assigned to the current user for this team and stop.

5. Call `wit_work_item` with action `get_batch` for all filtered item IDs with `expand=relations` and the following fields:
   - `System.Id`
   - `System.Title`
   - `System.State`
   - `System.AssignedTo`
   - `System.WorkItemType`
   - `System.ChangedDate`
   - `System.IterationPath`
   - `Microsoft.VSTS.Common.Priority`
   - `System.Parent`

6. From the `relations` of each item, collect all **parent IDs** (`System.LinkTypes.Hierarchy-Reverse`) and **child IDs** (`System.LinkTypes.Hierarchy-Forward`). Call `wit_work_item` with action `get_batch` once with the combined set of those IDs (excluding any IDs already fetched) to get their `System.Id`, `System.Title`, and `System.State`.

7. Display the results as described in the **Display results** section below.

# Display results

Sort all items by `Microsoft.VSTS.Common.Priority` ascending (Priority 1 first). For each item, display:

---

### [{ID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID}) — {Title}

| Field | Value |
|---|---|
| **State** | {System.State} |
| **Priority** | {Microsoft.VSTS.Common.Priority} |
| **Changed Date** | {System.ChangedDate formatted as MM/DD/YYYY} |
| **Iteration** | {last segment of System.IterationPath} |
| **Work Item Type** | {System.WorkItemType} |

**Parent:**
- If a parent exists: [`{ParentID}`](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ParentID}) {Parent Title}
- If no parent exists: *(none)*

**Child Items:**

If there are child items, display them in a table:

| ID | Title | State |
|----|-------|-------|
| [{ChildID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ChildID}) | {Child Title} | {Child State} |

If there are no child items, display: *(none)*

---

After all items are listed, display a summary line:

> Showing **{N}** active backlog item(s) assigned to you on team **{team name}** in project **{project name}**.
