---
name: work-iterations
description: List, create, and assign iterations for Azure DevOps projects and teams.
---

# Skill Instructions

This skill always works in the context of a **project**. A **team** is optional and only required when assigning iterations or listing iterations for a specific team.

## Project selection

- If the user **provides a project name** in their request (for example, "for Contoso"), use that project directly and **do not call** `core_list_projects`.
- If the user **does not provide a project name**, first ask the user once to provide the project name.
- If the project name is **still not provided after asking once**, call `core_list_projects` to return a list of projects the user can choose from.

## Team selection

- If the **team is not provided**, only perform actions that require **project-only context** (list or create project-level iterations). Do **not** attempt to assign iterations.
- If the user asks to **assign iterations** and **no team is provided**, ask once for the team name.
- If the team name is **still not provided after asking once**, call `core_list_project_teams` for the selected project so the user can pick a team.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `core_list_projects`: Get a list of projects in the organization.
- `core_list_project_teams`: Get a list of teams for a project.
- `work_create_iterations`: Create iterations in the project.
- `work_assign_iterations`: Assign iterations to a team.
- `work_list_team_iterations`: List iterations currently assigned to a team.
- `work_list_iterations`: List iterations for the project.

# Rules

## 1. List iterations for a project

- When the user asks **only to list iterations for a project**, call `work_list_iterations` for that project.
- Do **not** call `work_create_iterations` or `work_assign_iterations` in this case.
- Show the results in a list, including iteration name and dates.
- If there are no iterations, explicitly state that there are no iterations for this project.

### Example

- "list iterations for project Contoso"

## 2. List iterations for a project and team

- When the user asks to **list iterations for a specific project and team**, call `work_list_team_iterations` for the given project and team.
- Do **not** call `work_create_iterations` or `work_assign_iterations` in this case.
- Show the results in a list, including iteration name and dates.
- If there are no iterations, explicitly state that there are no iterations for this team.

### Example

- "list iterations for project Contoso and team Contoso"

## 3. Create iterations for a project

- When the user asks to **create iterations** for a project, use the tool `work_create_iterations`.
- First, call `work_list_iterations` to get the full list of current iterations and determine the existing date cadence (start and finish dates, and iteration length).
- Using that cadence, create new iterations and pass them to `work_create_iterations`.
- After creation, show the new iterations in a list, including iteration name and dates.
- If no iterations are created, explicitly state that no iterations were created for this project.

### Example 1

- "create iterations after the last available iteration, using the same cadence, through 2026"

	- Call `work_list_iterations` to get the full list of current iterations and determine the existing date cadence.
	- Using that date cadence, create new iterations and pass them to `work_create_iterations`.

### Example 2

- "create iterations after the last available iteration, using the same cadence, through 2026. Then assign all of those new iterations to team 'Contoso Team'"

	- Call `work_list_iterations` to get the full list of current iterations and determine the existing date cadence.
	- Using that date cadence, create new iterations and pass them to `work_create_iterations`.
	- Take the iterations created and assign them to the team "Contoso Team" using `work_assign_iterations`.
	- Show the created iterations (and, if helpful, note that they are assigned to the specified team).

## 4. Assign iterations to a team

- When the user asks to **assign iterations to a team**, use `work_assign_iterations`.
- Make sure the project and team are selected using the rules above.
- Show the iterations that were assigned in a list, including iteration name and dates.
- If no iterations are assigned, explicitly state that there are no iterations assigned for this team.

### Example 1

- "assign iterations 'Iteration 1', 'Iteration 2', and 'Iteration 3' to team 'Contoso Team'"

	- Call `work_assign_iterations` with the list of iterations and team name to assign those iterations to the team.

### Example 2

- "assign all iterations in 2025 and 2026 to team 'Contoso Team'"

	- Call `work_list_iterations` to get the full list of iterations.
	- From the list, find the iterations that fall within 2025 and 2026.
	- Take the iterations that fall within 2025 and 2026 and assign them to the team "Contoso Team" using the tool `work_assign_iterations`.
	- Show the assigned iterations in a list.
