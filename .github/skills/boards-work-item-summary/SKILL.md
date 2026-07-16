---
name: boards-work-item-summary
description: Summarize a single Azure DevOps work item (and its links and comments) by ID.
---

# Get work item

This skill always works with **one specific work item ID**.

- If the user **provides an ID** (for example: "for 115770" or "work item 115770"), use that ID directly.
- If the user **does not provide an ID**, prompt the user once to provide the work item ID.
- If the ID is **still not provided after prompting**, stop and return a clear message stating that a work item ID is required and do not call any tools.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `wit_work_item` (action: `get`): Get a work item from Azure DevOps by its ID.
- `wit_work_item` (action: `list_comments`): Get the list of comments for a work item by its ID.
- `wit_work_item` (action: `get_batch`): Get work item details in batch by their IDs. Use this tool to get the details of linked work items, using the links returned from `wit_work_item` with action `get`.

# Steps

1. Call `wit_work_item` with action `get` and `expand=all` for the requested ID so that you retrieve fields, relations, and links in a single call.

2. If the work item has a parent (for example, `System.Parent` or a `System.LinkTypes.Hierarchy-Reverse` relation), include the parent ID in the set of linked IDs to summarize. You do **not** need a separate call; instead, rely on `wit_work_item` with action `get_batch` in the next step.

3. From the work item `relations` array, collect all linked work item IDs (parent, child, related, and any others). Call `wit_work_item` with action `get_batch` once with that list of IDs to get the details of all linked work items.

4. Call `wit_work_item` with action `list_comments` once to get the comments for the original work item.

# Display results

When displaying the results:

- Show the following fields for the main work item: **Id**, **Title**, **State**, **Assigned To** (just the display name, not the email), **Work Item Type**, **Created Date** formatted as `MM/DD/YYYY`, **Priority**, and **Tags**.

- Provide a **Links** table. For each linked work item, display:
	- **Id** as a clickable hyperlink in this exact format: `[{ID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID})`
	- **Link type** (for example: Parent, Child, Related, or the underlying relation name)
	- **Work item type**
	- **Title**
	- **State**

	Group and order the rows in this sequence: **Parent links first**, then **Child links**, then **Related links**, then any **other link types**.

- If there are pull requests linked to the work item (for example, via artifact links or specific PR relations), list them in a separate **Pull Requests** table with at least the PR ID, title, status, and a clickable link to the PR.

- Provide a short, plain-language **summary** of the work item’s description, system information, and repro steps (if those fields exist). Focus on the key intent, scope, and any critical technical or repro details.

- For **comments**:
	- If there are one or more comments, state the **total number of comments** and give a brief overview of the main themes or decisions mentioned.
	- If there are **no comments**, explicitly state that there are no comments for this work item.
