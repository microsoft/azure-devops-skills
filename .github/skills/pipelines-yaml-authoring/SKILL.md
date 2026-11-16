---
name: pipelines-yaml-authoring
description: Azure Pipelines YAML mistakes that fail silently. Use when writing, reviewing, or debugging pipeline YAML, including steps that differ on Windows agents, empty variables or secrets, template parameters, conditions, Cache@2 misses, inflated or missing test results.
---

# Azure Pipelines YAML

These traps fail silently. After any change, expand the YAML with previewRun (`pipelines-run-and-validate`) and search for the changed value.

# Rules

## 1. Steps

- `script:` is `CmdLine@2`: cmd.exe on Windows agents. Use `bash:` or `pwsh:` for shell syntax.
- `$(x)` is substituted only if `x` is a defined variable; bash `$(cmd)` substitution is safe.
- Macros expand inside single quotes too: comparing a secret to its own macro always passes.

## 2. Variables and secrets

- `System.AccessToken` and secrets reach scripts only via `env:`.
- Map variable-group variables into `env:` explicitly; unmapped steps go empty once a variable is marked secret.
- A variable whose env name matches a secret's (`.` becomes `_`) empties the secret.
- Secrets can't be read back; share them through a variable group, not per-definition copies.
- No `CI` variable; Azure Pipelines sets `TF_BUILD`.
- `Build.SourceBranchName` is the last path segment only; branch filters need `Build.SourceBranch`.
- `[skip ci]` in the tip commit skips CI for the whole push.

## 3. Templates and parameters

- `${{ variables.x }}` in a template is empty for root-file variables; pass a template parameter.
- Boolean parameters reach scripts as `True`/`False`/`true`; compare case-insensitively or at template level.
- Template `eq()` is case-insensitive.
- `values:` without `default:` is not required: the Run dialog preselects the first value, the REST API rejects the missing value.
- To force a deliberate choice, drop `values:` and add a guard stage that fails on unknown values; otherwise a typo makes `${{ if }}` emit nothing and the run passes.

## 4. Conditions

- An explicit `condition:` replaces the implicit `succeeded()`: write `and(succeeded(), …)`.

## 5. Tasks

- Pin built-in tasks to the major only (`PublishTestResults@2`); a full pin silently runs whatever build of that major the agent has. Renovate expands them unless an `extractVersion` rule keeps the major.

## 6. Cache@2

- A key segment containing `/` is hashed as a file and fails if missing; quote literals: `"$(Agent.OS)"`.
- The key controls restore, not save; on reused agents the cached directory grows every build.
- An exact hit skips the save. For evolving caches end the key with `$(Build.SourceVersion)` and use `restoreKeys`.
- Keep cache paths outside `$(Build.SourcesDirectory)`; checkout cleans it.
- Scale-set pools: compare the agent name across runs before judging a cache change.

## 7. PublishTestResults@2

- It follows symlinks: pnpm/npm workspaces publish the same results many times. Narrow the pattern, or set `mergeTestResults: true` with a distinct `testRunTitle`.
- A regression shows as a drop in `TestResults To Publish <n>`.
