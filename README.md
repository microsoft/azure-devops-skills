# Azure DevOps Skills (Examples)

This repository contains examples of reusable "skills" for working with Azure DevOps from AI coding assistants (for example, GitHub Copilot in VS Code). Each skill is a focused bundle of instructions and, optionally, references or scripts that teach the model how to perform a specific Azure DevOps task in a consistent, repeatable way.

Skills in this repo are designed to:
- Standardize how common Azure DevOps workflows are executed (iterations, boards, summaries, etc.)
- Reduce repeated explanation of organization/project-specific conventions
- Provide clear, tool-based workflows using the Azure DevOps MCP server

---

## 📁 Repository Layout

- `skills/` – Individual skills, one folder per skill
  - `.github/skills/<skill-name>/SKILL.md` – Primary instructions and metadata for each skill
- `template/` – A starter template for creating new skills
- `.github/` – GitHub configuration and skill metadata
- `.vscode/` – VS Code workspace and MCP configuration

---

## 🧩 Available Skills

> This project is very new and limited in the number of skills defined. We expect to build out more skills over time. 

This repo currently includes skills focused on Azure DevOps work item and iteration management:

- `boards-my-work` – Lists the user's active work across Azure DevOps boards
- `boards-work-item-summary` – Summarizes a single work item (plus links and comments)
- `work-iterations` – Lists, creates, and assigns iterations for projects and teams

Each skill has its own `SKILL.md` under `.github/skills/<skill-name>/` describing:
- When the skill should be used
- Which Azure DevOps MCP tools it can call
- The exact workflows and edge cases it should handle

---

## 💻 Using These Skills in VS Code

These skills are intended to be used with GitHub Copilot in VS Code and an Azure DevOps Model Context Protocol (MCP) server.

At a high level:
1. Ensure you have the Azure DevOps MCP server configured and authenticated. See https://github.com/microsoft/azure-devops-mcp for details.
2. Open this repository in VS Code.
3. Make sure your Copilot / MCP configuration includes this workspace so SKILL files can be loaded.
4. In chat with Copilot, describe what you want to do (for example, "list iterations for project Contoso" or "summarize work item 123 for project Foo").
5. Copilot will select the appropriate skill and call the underlying Azure DevOps tools according to the instructions in `SKILL.md`.

---

## 🛠️ Creating a New Skill

To add a new Azure DevOps-focused skill:

1. **Start from the template**  
   - Copy the contents of `template/SKILL.md` into a new file under `.github/skills/<your-skill-name>/SKILL.md`.
2. **Fill out the frontmatter**  
   - `name`: a short, unique identifier for the skill (lowercase, hyphen-separated).  
   - `description`: a clear explanation of what the skill does and when it should be used.
3. **Define when to use the skill**  
   - Describe the kinds of user requests that should trigger this skill (for example, "iteration planning", "board queries", "build summaries").
4. **Specify tools and workflows**  
   - List the Azure DevOps MCP tools the skill may call.  
   - For each common user task, describe which tool(s) to call, in what order, and how to present the results.  
   - Be explicit about what **not** to do (for example, "do not create iterations when the user only asked to list them").
5. **Keep instructions concise and unambiguous**  
   - Prefer numbered workflows and clear guards over prose.  
   - Call out edge cases (missing project/team, empty results, error conditions) and how the model should respond.

Use the existing skills in `.github/skills/` as concrete references for structure, tone, and level of detail.

---

## 🤝 Contributing

Contributions are welcome. When adding or updating a skill:
- Keep instructions focused on Azure DevOps workflows and the MCP tools actually available.
- Ensure behavior is deterministic and guarded (for example, ask once for missing project/team, then fall back to list APIs).
- Favor small, composable skills over large, monolithic ones.

Before submitting changes:
- Validate that SKILL instructions are internally consistent and do not conflict with existing tools or skills.
- Test the workflows manually in VS Code with Copilot and the Azure DevOps MCP server, where possible.

---

## 📜 License

See `LICENSE.md` for licensing terms.

If your organization plans to extend or adapt these skills, consider maintaining a separate, internal fork with any org-specific conventions (project naming, team structures, field customizations) captured as additional skills.
