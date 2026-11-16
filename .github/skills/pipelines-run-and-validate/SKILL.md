---
name: pipelines-run-and-validate
description: Queue Azure DevOps pipeline runs on a branch or PR, and check pipeline YAML changes without running them (previewRun). Use when passing template parameters, waiting for a run's result, cancelling or retrying a run, finding a definition by its YAML file, or on an Unexpected parameter error.
---

# Run pipelines and validate YAML

Queue a run only when asked; confirm definition, ref, and parameters first. Queuing one to prove YAML you changed: check the change against `pipelines-yaml-authoring` first.

# Tools

Use Azure DevOps MCP Server tools for all interactions with Azure DevOps.

- `pipelines_definition` (action `list`)
- `pipelines_write` (actions `run_pipeline`, `update_build_stage`)
- `pipelines_run` (action `get`)
- `pipelines_build` (action `get_status`)

# Rules

## 1. Ref

- `run_pipeline` has no branch argument. Set `resources: { repositories: { self: { refName: "refs/heads/<branch>" } } }`; without it the run uses the default branch and its YAML.
- PR merge commit: `refName: "refs/pull/<prId>/merge"`.
- Verify `resources.repositories.self.refName` and `version` in the response.

## 2. Parameters

- `templateParameters` values are strings, also for booleans: `{ "fullBuild": "true" }`.
- They are validated against the definition's YAML on that ref, even with `yamlOverride` (`Unexpected parameter '<name>'`).
- A parameter without `default` is required, also for previewRun (`A value for the '<name>' parameter must be provided`).
- The web UI's **Run new** on a build page inherits that run's parameters.

## 3. previewRun

- `run_pipeline` with `previewRun: true` returns `finalYaml` and queues nothing — the only pre-merge YAML check. Set the ref as in rule 1.
- `yamlOverride` (previewRun only) replaces the root file; templates still resolve from `refName`. Use it for a renamed root file or a minimal root around one template.
- Expansions are huge; search `finalYaml` for the changed value instead of reading it whole.
- previewRun doesn't check runtime behaviour (task inputs, `Cache@2` keys, secrets).

## 4. Cancel and retry

- Stages: `update_build_stage` (`Cancel`, `Retry`, `Run`).
- The MCP server can't cancel a whole build: cancel its running stages, or ask the user to cancel it in the web UI.
- Retrying a stage that already published an artifact fails with `Artifact … already exists`; queue a new run.

## 5. Find a definition

- `list` with `yamlFilename` needs `includeAllProperties: true` and the path as stored, without a leading `/` (`.azuredevops/pipeline.yml`). Otherwise it errors or returns `[]`.

## 6. Wait for a result

- `get_status` returns an HTML report (`Build Started`, `Build Succeeded`), not fields to compare. Poll `pipelines_run` (action `get`; `runId` is the build ID, plus `pipelineId`) instead.
- `pipelines_run`: `state` 1 inProgress, 4 completed; `result`, present once completed: 1 succeeded, 2 failed, 4 canceled.
- `pipelines_build` (action `list`) numbers differ: `status` 2 completed; `result` 2 succeeded, 4 partiallySucceeded, 8 failed, 32 canceled.
