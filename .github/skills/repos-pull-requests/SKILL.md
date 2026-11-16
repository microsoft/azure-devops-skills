---
name: repos-pull-requests
description: Change Azure DevOps pull requests (retarget, title, description, labels, draft, merge commit message, auto-complete, votes) and verify the change took effect. Use when a PR update seems ignored, or before pushing to a policy-protected branch (TF402455).
---

# Pull requests

After every write, read the PR back with `repo_pull_request` (action `get`) and compare. The update response can mix new and stale values.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `repo_pull_request` (actions `get` with `includeLabels`, `list`)
- `repo_pull_request_write` (actions `create`, `update`, `update_reviewers`, `vote`)
- `repo_pull_request_thread` / `repo_pull_request_thread_write`

# Rules

## 1. Updates (microsoft/azure-devops-mcp#1612)

- Every `update` publishes a draft PR unless you pass `isDraft: true` — labels, auto-complete, and text changes alike.
- An `update` with `targetRefName` applies only the retarget; title, description, and other fields are silently dropped. Retarget alone, then update the rest.
- Description max 4000 characters; longer rejects the whole call.

## 2. Labels

- `labels` replaces the whole set: read the current labels, edit the list, pass all of them. `[]` removes all.

## 3. Merge commit message and auto-complete

- `mergeCommitMessage` is stored separately from the description and is what lands on the target branch. Update it when the PR's scope changes; keep its first line equal to the title.
- `mergeCommitMessage` is only stored with `autoComplete: true`; otherwise it is silently ignored.
- Arming replaces all completion options: pass `mergeStrategy`, `deleteSourceBranch`, and `transitionWorkItems` every time, or they reset (`noFastForward`, not deleted, transition).
- `autoComplete: false` does not cancel auto-complete; the user cancels it with **Cancel auto-complete** on the pull request in the web UI.
- Completion options are capped at 4000 encoded characters.
- Armed means `autoCompleteSetBy` is not null.
- To change the merge message without arming, use the web **Complete pull request** dialog.

## 4. Votes

- Vote, approve, or complete only when asked.
- The reviewers list shows current votes only. History is in thread system comments: `<name> voted 10` (10 approve, 5 with suggestions, -5 waiting, -10 reject) and `Vote of <name> was reset`.
- A push can reset approvals. For an approved PR with an expired build, re-queue the policy build instead of pushing.

## 5. Stacked PRs with squash merges

- After the parent completes, retarget the child and merge the base branch into the child's source branch; otherwise the child re-shows the parent's changes.
- Merge the base up the stack; don't rebase (resets votes and comment anchors).

## 6. Direct push to a protected branch

- Any enabled branch policy rejects a direct push (`TF402455`). `git push --atomic` is unsupported.
- Use the per-user **Bypass policies when pushing** permission. Disabling policies releases every PR with auto-complete armed.
- Push the branch before tags.
