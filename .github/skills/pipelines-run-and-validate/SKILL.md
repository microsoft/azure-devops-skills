---
name: pipelines-run-and-validate
description: Queue, wait for, cancel, and retry Azure DevOps pipeline runs, and expand pipeline YAML without running it. Use when queuing an Azure DevOps pipeline run on a branch or PR, passing template parameters, waiting for a run's result, cancelling or retrying a run, finding a definition by its YAML file, or checking pipeline YAML changes without running them (previewRun, Unexpected parameter).
---

# Run pipelines and validate YAML

Queue a run only when asked; confirm definition, ref, and parameters first. Queuing one to prove YAML you changed: check the change against `pipelines-yaml-authoring` first.

# Tools

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
- Expansions are huge; search `finalYaml`, don't read it whole. To filter with the CLI:
  `az devops invoke --area pipelines --resource runs --route-parameters project=<project> pipelineId=<id> --http-method POST --api-version 7.1-preview --in-file <file>` with `{"previewRun": true, "resources": {"repositories": {"self": {"refName": "refs/heads/<branch>"}}}}`. `7.1-preview.1` fails to parse; `--in-file` needs a real file.
- previewRun doesn't check runtime behaviour (task inputs, `Cache@2` keys, secrets).

## 4. Cancel and retry

- Stages: `update_build_stage` (`Cancel`, `Retry`, `Run`).
- Whole build via CLI: `az devops invoke --area build --resource builds --route-parameters project=<project> buildId=<id> --http-method PATCH --api-version 7.1 --in-file <file>` with `{"status": "cancelling"}`.
- Retrying a stage that already published an artifact fails with `Artifact … already exists`; queue a new run.

## 5. Find a definition

- `list` with `yamlFilename` needs `includeAllProperties: true` and the path as stored, without a leading `/` (`.azuredevops/pipeline.yml`). Otherwise it errors or returns `[]`.

## 6. Wait for a result

- `get_status` returns an HTML report (`Build Started`, `Build Succeeded`), not fields to compare. Poll `pipelines_run` (action `get`; `runId` is the build ID, plus `pipelineId`) instead.
- `pipelines_run`: `state` 1 inProgress, 4 completed; `result`, present once completed: 1 succeeded, 2 failed, 4 canceled.
- `pipelines_build` (action `list`) numbers differ: `status` 2 completed; `result` 2 succeeded, 4 partiallySucceeded, 8 failed, 32 canceled.
- Names instead of numbers: `az pipelines runs show --id <id> --query "[status,result]" -o tsv`.
