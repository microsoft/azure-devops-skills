---
name: cool-tools-frozen-items
description: Find work items that are frozen (blocked from state changes) using the cool·tools Work Item Freeze or Status Freeze extensions (cool-tools.p2-work-item-freeze, cool-tools.p3-status-freeze).
---

# Find frozen work items

This skill identifies work items currently frozen by the [cool·tools Work Item Freeze](https://marketplace.visualstudio.com/items?itemName=cool-tools.p2-work-item-freeze) or [Status Freeze](https://marketplace.visualstudio.com/items?itemName=cool-tools.p3-status-freeze) extensions.

- If the user **specifies a project or area path**, scope the search there.
- If the user **does not specify a scope**, search the current project.
- Frozen items carry the tag `frozen` or a non-empty `Custom.FreezeReason` field.

# Tools

- `wit_run_wiql`: Query for items tagged `frozen` or with a non-empty `Custom.FreezeReason`.
- `wit_get_work_items_batch_by_ids`: Fetch details including freeze reason and state.
- `core_list_projects`: List projects if none specified.

# Steps

1. If no project specified, call `core_list_projects` and confirm with the user.
2. Call `wit_run_wiql` with:
   `SELECT [System.Id] FROM WorkItems WHERE [System.TeamProject] = '{project}' AND ([System.Tags] CONTAINS 'frozen' OR [Custom.FreezeReason] <> '') AND [System.State] <> 'Closed' ORDER BY [System.ChangedDate] DESC`
3. If no results with custom field, retry using only the tag filter.
4. Call `wit_get_work_items_batch_by_ids` for the returned IDs — fetch ID, title, state, type, assigned-to, freeze reason, and changed date.
5. Display results grouped by freeze type (tag vs. field).

# Display results

- Show a **summary**: "{N} frozen work items found in {project}"
- Table columns: **ID** (linked), **Title**, **Type**, **State**, **Assigned To**, **Freeze Reason**, **Frozen Since**
- Link each ID to `https://dev.azure.com/{org}/{project}/_workitems/edit/{ID}`
- If 0 results: confirm that no items are currently frozen — this is the expected healthy state.
- If many results: suggest reviewing the freeze policy with the team lead.
