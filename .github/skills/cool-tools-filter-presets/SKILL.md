---
name: cool-tools-filter-presets
description: Apply, list, or manage saved filter presets for Azure DevOps boards and backlogs using the cool·tools ADO Filter extension (cool-tools.p1-ado-filter).
---

# Apply cool·tools filter presets

This skill helps users work with saved filter presets created by the [cool·tools ADO Filter](https://marketplace.visualstudio.com/items?itemName=cool-tools.p1-ado-filter) extension.

- If the user **specifies a filter name or criteria**, apply that filter directly.
- If the user **does not specify a filter**, list available presets and prompt once to choose.
- If no presets exist yet, guide the user to create one via the cool·tools ADO Filter extension.

# Tools

Use Azure DevOps MCP Server tools for all interactions.

- `wit_run_wiql`: Execute a WIQL query matching the selected filter preset criteria.
- `wit_get_work_items_batch_by_ids`: Retrieve full details for matched work item IDs.
- `core_list_projects`: List available projects when the user has not specified one.

# Steps

1. If no project is specified, call `core_list_projects` and ask the user to confirm the target project.
2. Identify the filter preset the user wants to apply (by name or description).
3. Call `wit_run_wiql` with the WIQL expression matching the preset logic (type, state, area path, tag filters).
4. Call `wit_get_work_items_batch_by_ids` for the returned IDs to fetch title, state, type, assigned-to, and iteration path.
5. Display results as a table grouped by work item type.

# Display results

- Show a **summary line**: "{N} work items matching filter '{preset name}'"
- Display a table with columns: **ID** (linked), **Title**, **Type**, **State**, **Assigned To**, **Iteration**
- Link each ID to `https://dev.azure.com/{org}/{project}/_workitems/edit/{ID}`
- If 0 results: suggest adjusting the filter criteria or checking the area path scope.
