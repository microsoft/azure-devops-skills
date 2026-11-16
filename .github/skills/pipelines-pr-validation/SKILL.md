---
name: pipelines-pr-validation
description: Find and diagnose the validation build of an Azure DevOps pull request and check which branch policies apply. Use when an Azure DevOps pull request seems to have no validation build, a build listing is empty after a push, CI looks broken, or you need to know which branch policies gate a branch.
---

# Pull request validation builds

Never claim a trigger is broken — in chat or on the PR — before working through rule 3.

# Tools

- `repo_pull_request` (action `get`): `mergeStatus`, `lastMergeSourceCommit`, `targetRefName`.
- `pipelines_build` (action `get_status`).
- The MCP server has no policy tool: use `az repos pr policy list` and `az repos policy list`.

# Rules

## 1. PR builds run on the merge ref

- PR builds have `sourceBranch` `refs/pull/<prId>/merge`, never `refs/heads/<branch>`. Filter listings with `--branch refs/pull/<prId>/merge`.
- The merge ref is recomputed when the PR changes, not when the target moves. Compare a run's `sourceVersion` with the PR's current merge commit.

## 2. Find the build for a push

- `az repos pr policy list --id <prId>`: the Build policy's `context.buildId` shows the build immediately and whether it is required.
- Build listings lag the queue by up to minutes.
- Match the build's `sourceVersion` to the pushed commit.
- `az pipelines runs list --top 1` without a branch filter can return an unrelated run.

## 3. No build at all

Check in order:

1. `mergeStatus` `conflicts` (`2`): no build and no error. Resolve the conflict and push.
2. No build validation policy on the target branch (e.g. a stacked PR): no build; `pr:` triggers don't apply. Say in the PR that it is unvalidated.

## 4. Branch policies

- Check policies per branch; the YAML `trigger:` list says nothing about them: `az repos policy list --repository-id <id> --branch <branch> --query "[].{name:type.displayName, enabled:isEnabled, blocking:isBlocking}" -o json` (`-o table` drops the name).
- Don't diff two policy listings to claim a policy changed; they vary spuriously.
- `az devops invoke --area policy --resource configurations` needs `--api-version 7.1-preview`.
