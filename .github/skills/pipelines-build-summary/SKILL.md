---
name: pipelines-build-summary
description: List, inspect, and troubleshoot Azure DevOps pipeline builds. Shows recent builds for a pipeline definition, drills into a specific build's status and result, displays logs for failed steps, and lists the changes associated with a build.
---

# Pipeline build summary

This skill works in the context of a **project**. A **pipeline definition** or **build ID** is optional and depends on what the user asks.

## Project selection

- If the user **provides a project name** in their request (for example, "for Contoso"), use that project directly and **do not call** `core_list_projects`.
- If the user **does not provide a project name**, first ask the user once to provide the project name.
- If the project name is **still not provided after asking once**, call `core_list_projects` to return a list of projects the user can choose from.

## Pipeline definition selection

- If the user **provides a pipeline or definition name**, use `pipelines_get_build_definitions` with a `name` filter to find it.
- If the user **does not specify a pipeline** and the request requires one (for example, "show recent builds"), ask the user once for the pipeline name.
- If the pipeline name is **still not provided after asking once**, call `pipelines_get_build_definitions` to list available definitions for the user to choose from.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `core_list_projects`: Get a list of projects in the organization.
- `pipelines_get_build_definitions`: Get a list of build/pipeline definitions for a project.
- `pipelines_get_builds`: Get a list of builds, optionally filtered by definition, branch, or status.
- `pipelines_get_build_status`: Get the status and result of a specific build by ID.
- `pipelines_get_build_log`: Get the list of logs for a specific build.
- `pipelines_get_build_log_by_id`: Get a specific log by log ID for a build (use to retrieve failed step logs).
- `pipelines_get_build_changes`: Get the commits/changes associated with a specific build.

# Rules

## 1. List recent builds for a pipeline

- When the user asks to **list builds** or **show recent builds** for a pipeline, call `pipelines_get_builds` filtered by the definition ID.
- Optionally filter by `branchName`, `statusFilter`, or `resultFilter` if the user specifies (for example, "failed builds on main").
- Do **not** fetch logs or changes unless the user asks.
- Show the results in a table.
- If there are no builds, explicitly state that there are no builds for this pipeline.

### Example

- "show recent builds for pipeline 'CI-Main' in project Contoso"

## 2. Get build status by ID

- When the user asks about a **specific build** (for example, "build 12345"), call `pipelines_get_build_status` with that build ID.
- Show the build number, status, result, source branch, start/finish time, requested by, and definition name.
- If the build failed, suggest viewing the logs.

### Example

- "what is the status of build 12345 in project Contoso?"

## 3. View logs for a failed build

- When the user asks to **view logs** or **why a build failed**, first call `pipelines_get_build_log` to get the list of all log entries for the build.
- Identify failed steps from the log list (look for steps that indicate errors or failures).
- Call `pipelines_get_build_log_by_id` for the relevant failed log entries to retrieve the actual log content.
- Present the log output in a code block so it is easy to read.
- Do **not** dump all logs. Focus on the failed steps and the last portion of each relevant log.

### Example

- "show me the logs for build 12345 in project Contoso"
- "why did build 12345 fail?"

## 4. List changes for a build

- When the user asks to **see what changed** or **list commits** for a build, call `pipelines_get_build_changes` with the build ID.
- Show the results in a table with commit ID, author, and message.
- If there are no changes, explicitly state that there are no changes associated with this build.

### Example

- "what changes were in build 12345?"
- "list commits for build 12345 in project Contoso"

# Display results

When displaying build lists, show the following in a table:

- **Build ID** as a clickable hyperlink in this format: `[{buildId}](https://dev.azure.com/{organization}/{project}/_build/results?buildId={buildId})`
- **Definition name**
- **Status** (for example: completed, inProgress, cancelling, notStarted)
- **Result** (for example: succeeded, failed, canceled, partiallySucceeded) — use emoji: ✅ succeeded, ❌ failed, ⚠️ partiallySucceeded, 🚫 canceled
- **Source branch**
- **Start time** formatted as `MM/DD/YYYY HH:MM`
- **Requested by** (display name only)

When displaying a single build's status, show all the above fields plus:
- **Finish time** formatted as `MM/DD/YYYY HH:MM`
- **Build number**
- **Duration** (calculated from start and finish time)

When displaying changes, show in a table:
- **Commit ID** (short hash, first 8 characters)
- **Author**
- **Message** (first line only)
- **Timestamp**
