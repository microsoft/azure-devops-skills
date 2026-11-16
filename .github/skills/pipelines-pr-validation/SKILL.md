---
name: pipelines-pr-validation
description: Find the validation build of an Azure DevOps pull request, or why it has none. Use when a PR seems to have no validation build, a build listing is empty after a push, or CI looks broken.
---

# Pull request validation builds

Work through rule 3 before saying a trigger is broken, in chat or on the PR.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `repo_pull_request` (action `get`): `mergeStatus`, `lastMergeCommit`, `lastMergeSourceCommit`, `targetRefName`.
- `pipelines_build` (actions `list`, `get_status`).
- No MCP tool reads branch policies, and the YAML `trigger:` list says nothing about them: ask the user to check the target branch's **Branch policies** in the web UI.

# Rules

## 1. PR builds run on the merge ref

- PR builds have `sourceBranch` `refs/pull/<prId>/merge`, never `refs/heads/<branch>`.
- The merge ref is recomputed when the PR changes, not when the target moves.

## 2. Find the build for a push

- List with `branchName: refs/pull/<prId>/merge` and `queryOrder: QueueTimeDescending`; without the branch filter the newest build can be an unrelated run.
- Build listings lag the queue by up to minutes.
- The build for the push has `sourceVersion` equal to the PR's `lastMergeCommit`, and `system.pullRequest.sourceCommitId` in its `parameters` equal to `lastMergeSourceCommit`, the pushed commit.

## 3. No build at all

Check in order:

1. `mergeStatus` `conflicts` (`2`): no build and no error. Resolve the conflict and push.
2. No build validation policy on the target branch (e.g. a stacked PR): no build; `pr:` triggers don't apply. Say in the PR that it is unvalidated.

