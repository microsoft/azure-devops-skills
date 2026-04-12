---
name: cool-tools-iteration-issues
description: Detect iteration path inconsistencies in Azure DevOps — child items in different iterations than their parents — using the cool·tools Iteration Checker extension (cool-tools.p4-iteration-checker).
---

# Find iteration path issues

This skill surfaces hierarchy inconsistencies detected by the [cool·tools Iteration Checker](https://marketplace.visualstudio.com/items?itemName=cool-tools.p4-iteration-checker) — specifically child work items assigned to a different sprint or iteration than their parent.

- If the user **specifies an iteration or sprint**, scope the check there.
- If the user **does not specify**, check the current (active) iteration.
- Focus on parent–child pairs where the child's iteration path diverges from the parent's.

# Tools

- `wit_run_wiql`: Find parent work items in the target iteration.
- `wit_get_work_items_batch_by_ids`: Fetch parent details including child relation IDs.
- `wit_get_work_item`: Get individual child work item details to compare iteration paths.
- `boards_list_iterations`: List available iterations to resolve "current sprint" references.

# Steps

1. If no iteration specified, call `boards_list_iterations` and identify the current (active) iteration.
2. Call `wit_run_wiql` to find all Features and User Stories in the target iteration:
   `SELECT [System.Id], [System.IterationPath], [System.Title] FROM WorkItems WHERE [System.TeamProject] = '{project}' AND [System.IterationPath] UNDER '{iteration}' AND [System.WorkItemType] IN ('Feature', 'User Story') ORDER BY [System.WorkItemType]`
3. Call `wit_get_work_items_batch_by_ids` with `expand=relations` to get child IDs.
4. For each parent, collect child IDs from the `relations` array (link type `System.LinkTypes.Hierarchy-Forward`).
5. Call `wit_get_work_items_batch_by_ids` for child IDs in batches — compare each child's `System.IterationPath` to its parent's.
6. Collect all mismatches: parent iteration ≠ child iteration.

# Display results

- Show a **summary**: "{N} iteration mismatches found in {iteration}"
- Table columns: **Parent ID** (linked), **Parent Title**, **Parent Iteration**, **Child ID** (linked), **Child Title**, **Child Iteration**
- Link each ID to `https://dev.azure.com/{org}/{project}/_workitems/edit/{ID}`
- If 0 mismatches: confirm the iteration is consistent — no action needed.
- If mismatches exist: suggest using the cool·tools Iteration Checker extension to bulk-fix iteration paths.
