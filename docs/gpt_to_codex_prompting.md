# GPT-to-Codex Prompting

## 1. Document Role

This document defines GPT-side prompt packaging for routine Codex tasks in `Automatic_2`. It describes how GPT should write executable Codex prompts after GPT has already determined that Codex execution is needed.

This document is not a user-request router, not a GPT decision workflow, not a Codex analysis skill, and not a training-system manifest. It does not define tuning decisions, result interpretation policy, accepted-baseline policy, next-hyperparameter policy, training-system engineering facts, or Codex data-analysis methods.

`docs/training_system_manifest.md` may be included in Codex prompt required reading when passive engineering context is relevant. `.agents/skills/training-run-factual-analysis/SKILL.md` is the currently available concrete Codex skill for single-run factual analysis.

## 2. Prompt Language And Shape

Codex prompts generated from this document should be one complete English prompt block. The prompt block should be directly executable by Codex without requiring cross-document orchestration.

Use this prompt shape:

- Task title
- Task type
- Skill invocation, when the task interface has a concrete skill path
- Working context
- Source inputs
- Output destination
- Goal
- Required reading
- Allowed writes
- Forbidden writes
- Required procedure
- Validation
- Commit and push policy
- Final report requirements

Do not mention model settings or reasoning settings inside Codex prompt blocks.

## 3. Canonical Skill Invocation Format

Use this exact local Markdown-link format when a task interface has a concrete local skill file:

```text
Skill invocation:
  Use [$<skill-name>](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\<skill-name>\SKILL.md)
```

A task interface is callable only when it contains this concrete local `SKILL.md` path. Task interfaces without a concrete `Skill invocation` field define reserved interface contracts but are not executable skill calls.

Current concrete invocation:

```text
Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)
```

## 4. General Prompt Boundary Rules

All Codex prompts generated from these interfaces must follow these rules:

- Codex must only perform the requested bounded task.
- Codex must not provide GPT tuning decisions unless explicitly assigned a decision role; routine Codex tasks do not have a decision role.
- Codex must not provide tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, or paper-level conclusions.
- Codex must not modify the training repository unless the specific task explicitly authorizes training-repository writes.
- Codex must not copy checkpoints, model weights, full logs, full CSVs, raw outputs, plots, or trajectories.
- Codex must sanitize private local paths in tracked outputs.
- Codex must validate JSON outputs when JSON is written.
- Codex may commit and push only when validation passes and only allowed files changed.
- Codex must report files changed, validation commands, remaining risks, and whether forbidden actions were avoided.

## 5. Routine Codex Task Interfaces

### 5.1 training_run_factual_analysis_task

Purpose: analyze one completed training run into the current structured JSON factual analysis.

Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)

Required input:

- `source_run_dir`
- `run_name`

Default output:

- `training_results/current/current_training_run_analysis.json`

Required prompt behavior:

- Verify reproducible-launch evidence first.
- Treat train-side monitoring as primary factual evidence.
- Treat post-hoc selection as checkpoint-selection context.
- Treat `final_probe` as supplemental only.
- Write JSON-only analysis output.
- Provide no tuning decision.

Default allowed writes:

- `training_results/current/current_training_run_analysis.json`

Default forbidden writes:

- training repository
- source run outputs
- `DRL_automatic`
- skills
- manifest
- `README.md`
- history unless explicitly requested

Required validation:

- JSON parse check
- `git diff` scope
- `git diff --check`
- training repository status check
- no forbidden artifact copy

Commit and push:

- Allowed only if validation passes and only allowed files changed.

### 5.2 multi_run_script_generation_task

This interface is reserved as a native routine task interface.

Skill identity:

- `$multi-run-script-generation`

Expected skill file:

- `.agents/skills/multi-run-script-generation/SKILL.md`

Purpose: generate executable multi-run training scripts or launch-command sets from a GPT-provided parameter plan.

Required input:

- GPT-provided parameter plan
- training script target path or command output target
- execution mode constraints

Default output concept:

- generated multi-run script or launch-command file

Important boundary:

- Training-repository writes require explicit task authorization.
- Codex provides no tuning decision.
- Codex does not automatically execute training unless explicitly requested.

### 5.3 multi_run_factual_analysis_task

This interface is reserved as a native routine task interface.

Skill identity:

- `$multi-run-factual-analysis`

Expected skill file:

- `.agents/skills/multi-run-factual-analysis/SKILL.md`

Purpose: analyze multiple completed training run directories into a structured current multi-run factual comparison.

Required input:

- list of `source_run_dir` entries
- run labels
- comparison scope

Default output:

- `training_results/current/current_multi_run_analysis.json`

Required prompt behavior:

- Verify reproducible-launch evidence per run.
- Treat train-side monitoring as primary factual evidence per run.
- Provide cross-run factual comparison only.
- Treat `final_probe` as supplemental only.
- Provide no tuning recommendation.

Default allowed writes:

- `training_results/current/current_multi_run_analysis.json`

Default forbidden writes:

- training repository
- source run outputs
- current single-run analysis unless explicitly requested

### 5.4 single_run_analysis_archive_task

This interface is reserved as a native routine task interface.

Skill identity:

- `$training-analysis-archive`

Expected skill file:

- `.agents/skills/training-analysis-archive/SKILL.md`

Purpose: archive the current single-run factual analysis JSON into history after GPT has produced a new tuning decision or after the user authorizes archival.

Required input:

- `training_results/current/current_training_run_analysis.json`
- archive identity derived from `run_name` or user-provided archive id

Default output concept:

- `training_results/history/single_runs/<archive_id>/training_run_analysis.json`

Required prompt behavior:

- Do not reanalyze training outputs.
- Do not modify current JSON unless explicitly requested.
- Validate JSON before archiving.
- Preserve source report facts.
- Provide no tuning decision by Codex.

### 5.5 multi_run_analysis_archive_task

This interface is reserved as a native routine task interface.

Skill identity:

- `$training-analysis-archive`

Expected skill file:

- `.agents/skills/training-analysis-archive/SKILL.md`

Purpose: archive the current multi-run factual analysis JSON into history after GPT/user authorization.

Required input:

- `training_results/current/current_multi_run_analysis.json`
- archive identity derived from `plan_id`, run group id, or user-provided archive id

Default output concept:

- `training_results/history/multi_runs/<archive_id>/multi_run_analysis.json`

Required prompt behavior:

- Do not reanalyze training outputs.
- Do not modify current JSON unless explicitly requested.
- Validate JSON before archiving.
- Preserve source report facts.
- Provide no tuning decision by Codex.

## 6. Template: Single-Run Factual Analysis Prompt

```text
Task title:
  Analyze one completed training run into the current factual JSON report

Task type:
  training_run_factual_analysis_task

Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)

Working context:
  Target repository:
    C:\Users\Dk\Desktop\SCI\Automatic_2

Source inputs:
  source_run_dir:
    <source_run_dir>
  run_name:
    <run_name>

Output destination:
  <output_json_path>

  Default output path:
    C:\Users\Dk\Desktop\SCI\Automatic_2\training_results\current\current_training_run_analysis.json

Goal:
  Inspect the completed source run and write one structured JSON factual analysis report. The generated analysis output must be JSON only. Do not provide tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, or paper-level conclusions.

Required reading:
  - C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md
  - C:\Users\Dk\Desktop\SCI\Automatic_2\docs\training_system_manifest.md when passive engineering context is needed
  - Source run logs and source run metadata under <source_run_dir>

Allowed writes:
  - <output_json_path>

Forbidden writes:
  - Do not modify the training repository.
  - Do not modify source run outputs.
  - Do not modify DRL_automatic.
  - Do not modify existing Codex skills.
  - Do not modify C:\Users\Dk\Desktop\SCI\Automatic_2\docs\training_system_manifest.md.
  - Do not modify C:\Users\Dk\Desktop\SCI\Automatic_2\README.md.
  - Do not copy checkpoints, model weights, full logs, full CSVs, raw output directories, plots, or trajectories.

Required procedure:
  - Verify reproducible-launch evidence before summarizing metrics.
  - Use train-side monitoring as primary factual evidence.
  - Treat post-hoc selection as checkpoint-selection context.
  - Treat final_probe as supplemental validation evidence only.
  - Sanitize private absolute paths in the JSON report.
  - Write valid JSON to <output_json_path>.
  - Keep tuning_recommendation_provided set to false.

Validation:
  Run from C:\Users\Dk\Desktop\SCI\Automatic_2:
    - python -m json.tool <output_json_path>
    - git status --short --branch
    - git diff -- <output_json_path>
    - git diff --check
    - git diff --cached --check
  Also verify that the training repository was not modified and that no forbidden artifacts were copied.

Commit and push policy:
  Commit and push only if validation passes and only <output_json_path> changed. Do not commit or push if any forbidden file changed, the JSON is invalid, the training repository changed, or a forbidden artifact was copied.

Final report requirements:
  Report the output JSON path, files changed, validation commands and results, whether the training repository was modified, whether forbidden artifacts were copied, remaining risks, and confirmation that no tuning recommendation or decision was provided.
```

## 7. Update Policy

When a reserved interface obtains an accepted `SKILL.md` file, replace its `Skill identity` and `Expected skill file` fields with the canonical concrete `Skill invocation` field. Keep the surrounding interface structure unchanged.

Do not add temporary notes, patch notes, construction history, or wording that marks content as recently introduced. All callable skill invocations must use the same local Markdown-link format. Do not preserve obsolete prompt forms.
