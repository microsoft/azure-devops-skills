---
name: security-alert-review
description: List and review Advanced Security alerts for an Azure DevOps repository. Shows dependency vulnerabilities, secret exposure, and code scanning findings with filtering by severity, state, and alert type.
---

# Security alert review

This skill works in the context of a **project** and a **repository**. Both are required to retrieve alerts.

## Project selection

- If the user **provides a project name** in their request (for example, "for Contoso"), use that project directly and **do not call** `core_list_projects`.
- If the user **does not provide a project name**, first ask the user once to provide the project name.
- If the project name is **still not provided after asking once**, call `core_list_projects` to return a list of projects the user can choose from.

## Repository selection

- If the user **provides a repository name**, use that repository directly.
- If the user **does not specify a repository**, ask the user once for the repository name.
- If the repository name is **still not provided after asking once**, call `repo_list_repos_by_project` to list available repositories for the user to choose from.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `core_list_projects`: Get a list of projects in the organization.
- `repo_list_repos_by_project`: Get a list of repositories for a project.
- `advsec_get_alerts`: Get Advanced Security alerts for a repository, with optional filters for severity, state, alert type, and confidence level.
- `advsec_get_alert_details`: Get detailed information about a specific alert by ID.

# Rules

## 1. List alerts for a repository

- When the user asks to **list alerts**, **show security alerts**, or **review alerts**, call `advsec_get_alerts` for the specified project and repository.
- Apply filters based on the user's request:
  - **Severity**: filter by `severities` (for example, "show critical alerts" → `["Critical"]`).
  - **State**: filter by `states` (for example, "show active alerts" → `["Active"]`).
  - **Alert type**: filter by `alertType` (for example, "show dependency alerts" → `"Dependency"`). Valid types are: `Dependency`, `Secret`, `Code`.
- Always include `confidenceLevels: ["High", "Other"]` on every call to `advsec_get_alerts` unless the user explicitly requests a specific confidence filter.
- If the user does not specify filters, show all active alerts on the default branch by default (use `onlyDefaultBranch: true`, `states: ["Active"]`, and `confidenceLevels: ["High", "Other"]`).
- Show the results in a table.
- If there are no alerts, explicitly state that there are no alerts matching the criteria for this repository.

### Example

- "show security alerts for repo MyApp in project Contoso"
- "list critical dependency alerts for repo MyApp"
- "show all active secret alerts in repo MyApp"

## 2. Get details for a specific alert

- When the user asks about a **specific alert** (for example, "alert 42" or "tell me about alert 42"), call `advsec_get_alert_details` with the alert ID, project, and repository.
- Show all available detail fields including the affected file, line number, description, remediation guidance, and rule information.

### Example

- "show details for alert 42 in repo MyApp, project Contoso"
- "what is alert 42 about?"

## 3. Summary view

- When the user asks for a **summary** or **overview** of alerts, call `advsec_get_alerts` (with no severity or type filter, `states: ["Active"]`, and `confidenceLevels: ["High", "Other"]`) and present a summary grouped by:
  1. **Alert type** (Dependency, Secret, Code) with count.
  2. **Severity** (Critical, High, Medium, Low, Other) with count per type.
- Show the summary as a compact table followed by the total count.
- Note: `advsec_get_alerts` returns up to 100 alerts by default. If the results include a continuation token, let the user know the summary is based on the first batch of alerts and that additional alerts exist.

### Example

- "give me a security overview for repo MyApp"
- "summarize the alerts in repo MyApp for project Contoso"

# Display results

When displaying alert lists, show in a table:

- **Alert ID**
- **Title** (the alert title or rule name)
- **Severity** with emoji: 🔴 Critical, 🟠 High, 🟡 Medium, 🟢 Low
- **State** (Active, Dismissed, Fixed, AutoDismissed)
- **Alert type** (Dependency, Secret, Code)
- **Rule** (the rule ID or name)
- **First seen** formatted as `MM/DD/YYYY`

When displaying alert details, show:

- All fields from the list view, plus:
- **Description** — full text of what the alert means.
- **File path** and **line number** (if applicable) — where the issue was found.
- **Remediation** — guidance on how to fix the issue (if available from the alert details).
- **Confidence** — High or Other (for secret alerts).
- **Validity** — Active, Inactive, or Unknown (for secret alerts).
- **Tool name** — the scanning tool that found the alert.

When displaying the summary view, show:

| Alert Type | 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low | Other | Total |
|------------|------------|---------|----------|--------|-------|-------|
| Dependency | count      | count   | count    | count  | count | count |
| Secret     | count      | count   | count    | count  | count | count |
| Code       | count      | count   | count    | count  | count | count |
| **Total**  | count      | count   | count    | count  | count | count |

The **Other** column includes any alerts with severity values outside Critical/High/Medium/Low (for example, Note, Warning, Error, or Undefined).
