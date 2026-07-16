---
name: boards-team-active-work
description: Get active work items for a team showing dependencies, priorities, and sprint assignments. Displays items that are blocking others, high priority items first, and items due this sprint. Shows parent/child hierarchy with Title, State, Priority, ChangedDate, Iteration Name, and Iteration End Date.
---

# Get team active work items with dependencies

This skill retrieves the team's Requirements-level backlog, filters to **Active** items **assigned to the current user**, enriches them with parent/child details and dependency information, and displays them sorted by blocking status and priority with sprint information.

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
- `work` (action: `list_team_iterations`): Get the list of iterations for a team, including current sprint information.
- `wit_work_item` (action: `get_batch`): Get work item details in batch by their IDs. Use this to fetch fields and relations for backlog items, their parents, children, and dependencies in as few calls as possible.

# Steps

1. Resolve project and team using the rules above.

2. Call `work` with action `list_team_iterations` for the resolved project and team with `timeframe=current` to get the current sprint information. Store the current iteration path and end date for filtering and display purposes.

3. Call `wit_backlog` with action `list_work_items` for the resolved project and team, using backlog level `Microsoft.RequirementCategory` (the Requirements / Product Backlog Items level).

4. From the returned work items, **filter** to items that meet **all** of the following criteria:
   - `System.State` = `Active`

5. If no items match the filter, display a message stating there are no active backlog items for this team and stop.

6. Call `wit_work_item` with action `get_batch` for all filtered item IDs with `expand=relations` and the following fields:
   - `System.Id`
   - `System.Title`
   - `System.State`
   - `System.AssignedTo`
   - `System.WorkItemType`
   - `System.ChangedDate`
   - `System.IterationPath`
   - `Microsoft.VSTS.Common.Priority`
   - `System.Parent`

7. From the `relations` of each item, collect:
   - **Parent IDs** (`System.LinkTypes.Hierarchy-Reverse`)
   - **Child IDs** (`System.LinkTypes.Hierarchy-Forward`)
   - **Dependency IDs** - both items this work item depends on (`System.LinkTypes.Dependency-Forward`) and items that depend on this work item (`System.LinkTypes.Dependency-Reverse`)
   - **Related IDs** (`System.LinkTypes.Related`)

8. Call `wit_work_item` with action `get_batch` once with the combined set of all related IDs (excluding any IDs already fetched) to get their:
   - `System.Id`
   - `System.Title`
   - `System.State`
   - `System.IterationPath`

9. For each work item, determine:
   - **Is Blocking**: If there are any `System.LinkTypes.Dependency-Reverse` relations (other items depend on this one)
   - **Is in Current Sprint**: If the `System.IterationPath` matches the current iteration from step 2
   - **Priority**: The `Microsoft.VSTS.Common.Priority` value

10. Display the results as described in the **Display results** section below.

# Display results

**First, display the current sprint information:**

> **Current Sprint:** {Iteration Name} (ends {Iteration Finish Date formatted as MM/DD/YYYY})

---

**Then, sort and group the work items in the following order:**

1. **Items that are blocking others** (have `System.LinkTypes.Dependency-Reverse` relations) — sorted by Priority ascending (Priority 1 first)
2. **High priority items** (Priority 1 or 2) that are not blocking — sorted by Priority ascending
3. **Items due this sprint** (matching current iteration path) — sorted by Priority ascending
4. **All other items** — sorted by Priority ascending

For each item, display:

---

### [{ID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID}) — {Title}

{If blocking, show: **⚠️ BLOCKING OTHER WORK ITEMS**}

| Field | Value |
|---|---|
| **State** | {System.State} |
| **Priority** | {Microsoft.VSTS.Common.Priority} |
| **Changed Date** | {System.ChangedDate formatted as MM/DD/YYYY} |
| **Iteration** | {last segment of System.IterationPath} |
| **Iteration End Date** | {Iteration Finish Date from step 2 if in current sprint, otherwise look up from team iterations} |
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

**Dependencies:**

If this item is blocking others (has `System.LinkTypes.Dependency-Reverse` relations), show:

**⚠️ Blocking:**
- [{DependentID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{DependentID}) {Dependent Title} ({Dependent State})

If this item depends on others (has `System.LinkTypes.Dependency-Forward` relations), show:

**Depends on:**
- [{DependencyID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{DependencyID}) {Dependency Title} ({Dependency State})

If there are no dependencies, display: *(none)*

---

After all items are listed, display a summary:

> **Summary:**
> - **Total active items for the team:** {N}
> - **Items blocking others:** {M}
> - **High priority items (1-2):** {P}
> - **Items due this sprint:** {Q}
> 
> Team: **{team name}** | Project: **{project name}**
