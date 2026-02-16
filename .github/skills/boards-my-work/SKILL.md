---
name: boards-my-work
description: This skill retrieves and displays work items assigned to the user in Azure DevOps, organized by type and sorted by recently changed. It prompts for a project name or lists available projects if not provided, then fetches work item details and displays them in a formatted table with clickable links.
---

# Get my work items

Before getting work items, you need to know which project to use.

- If the user **provides a project name** in their request (for example, "for Contoso"), **use that project directly and do not call** the `core_list_projects` tool.
- If a project name is **not** provided, first return a prompt asking the user for the project name.
- If the project name is still not provided after prompting the user, then call the MCP Server tool `core_list_projects` to get the list of projects the user can choose from.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `core_list_projects`: Get the list of projects the user can choose from.
- `wit_my_work_items`: Get work items from Azure DevOps that are assigned to the user
- `wit_get_work_items_batch_by_ids`: Get work item details in batch by their IDs. Use this tool to get detailed information about the work items retrieved by `wit_my_work_items`.

# Fields

When using the tool `wit_get_work_items_batch_by_ids`, if the user does not provide a list of fields to retrieve, use the following default fields:

- System.Id
- System.Title
- System.State
- System.AssignedTo
- System.WorkItemType
- System.CreatedDate
- System.ChangedDate
- System.Tags
- Microsoft.VSTS.Common.Priority

# Display results

When displaying the results...

- show the work item ID (make this clickable hyperlink to open the work item in a web browser. Format should be like this: [{ID}](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID})),
- title
- state, 
- priority

in a table format. Show the most recently changed work items first and group the work items by thier work item type (System.WorkItemType). For example, group the work items into the following categories based on their work item type:

Epics
Features
Stories
Bugs
Tasks
Test Cases
other work item types