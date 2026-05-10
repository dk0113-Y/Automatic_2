# GPT-to-Codex Prompting

## 1. Document Role

This document defines GPT-side prompt packaging for routine Codex tasks in `Automatic_2`. It describes how GPT should write executable Codex prompts after GPT has already determined that Codex execution is needed.

This document is not a user-request router, not a GPT decision workflow, not a Codex analysis skill, and not a training-system manifest. It does not define tuning decisions, result interpretation policy, accepted-baseline policy, next-hyperparameter policy, training-system engineering facts, or Codex data-analysis methods.

`docs/training_system_manifest.md` may be included in Codex prompt required reading when passive engineering context is relevant. `.agents/skills/training-run-factual-analysis/SKILL.md` is the concrete Codex skill for single-run factual analysis. `.agents/skills/training-analysis-archive/SKILL.md` is the concrete Codex skill for paired analysis archive records.

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

Current concrete invocations:

```text
Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)
```

```text
Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)
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
- Codex must commit and push after validation passes and only allowed files changed, unless the user explicitly disables commit/push for the task.
- Codex must not commit or push when validation fails, forbidden files changed, forbidden artifacts were copied, JSON outputs are invalid, the training repository was modified without authorization, source run outputs were modified, or any task-specific blocking condition is present.
- Codex must report files changed, validation commands, remaining risks, whether forbidden actions were avoided, whether commit and push were performed, commit hash when available, push target branch when available, and the reason if commit/push was not performed.

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
- required top-level key completeness check
- `factual_summary_status` presence check
- `factual_summary_status` allowed-value check
- `tuning_recommendation_provided=false` check
- `git diff` scope
- `git diff --check`
- training repository status check
- source run output modification check
- no forbidden artifact copy

Commit and push:

- Required after validation passes and only allowed files changed, unless the user explicitly disables commit/push for the task.
- Do not commit or push if validation fails or any forbidden change is detected.

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

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Purpose: archive a paired single-run record containing `current_training_run_analysis.json` and GPT-provided `tuning_review_payload` into history.

Required input:

- `training_results/current/current_training_run_analysis.json`
- `tuning_review_payload` or `tuning_review_json_source`
- `archive_id` derived from `run_name` or user-provided archive id

Default output concept:

- `training_results/history/single_runs/<archive_id>/training_run_analysis.json`
- `training_results/history/single_runs/<archive_id>/tuning_review.json`
- `training_results/history/history_index.json`

Required prompt behavior:

- Do not reanalyze training outputs.
- Preserve source report facts.
- Preserve GPT tuning rationale.
- Require `tuning_review_payload` to use repository-relative paths or path labels.
- Use `recommended_next_command_summary.working_directory: source_training_repo` when the external training repository is referenced.
- Use `recommended_next_command_summary.command_template` with the `<source_training_repo>` placeholder when a command template is recorded.
- Use `recommended_next_command_summary.command_arguments` for concrete launcher parameters.
- Do not create `archive_manifest.json`.
- Provide no tuning decision by Codex.

### 5.5 multi_run_analysis_archive_task

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Purpose: archive a paired multi-run record containing `current_multi_run_analysis.json` and GPT-provided `tuning_review_payload` into history.

Required input:

- `training_results/current/current_multi_run_analysis.json`
- `tuning_review_payload` or `tuning_review_json_source`
- `archive_id` derived from `plan_id`, run group id, or user-provided archive id

Default output concept:

- `training_results/history/multi_runs/<archive_id>/multi_run_analysis.json`
- `training_results/history/multi_runs/<archive_id>/tuning_review.json`
- `training_results/history/history_index.json`

Required prompt behavior:

- Do not reanalyze training outputs.
- Preserve source report facts.
- Preserve GPT tuning rationale.
- Require `tuning_review_payload` to use repository-relative paths or path labels.
- Use `recommended_next_command_summary.working_directory: source_training_repo` when the external training repository is referenced.
- Use `recommended_next_command_summary.command_template` with the `<source_training_repo>` placeholder when a command template is recorded.
- Use `recommended_next_command_summary.command_arguments` for concrete launcher parameters.
- Do not create `archive_manifest.json`.
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
    - python - <<'PY'
      import json
      from pathlib import Path

      path = Path(r"training_results\current\current_training_run_analysis.json")
      data = json.loads(path.read_text(encoding="utf-8"))

      required_keys = [
          "schema_version",
          "report_type",
          "generated_by",
          "source_run_dir",
          "run_name",
          "files_inspected",
          "commands_run",
          "reproducible_launch_status",
          "reproducible_launch",
          "train_side_monitoring",
          "posthoc_selection",
          "supplemental_final_probe",
          "configuration_and_runtime",
          "missing_artifacts",
          "parse_failures",
          "unverified_items",
          "forbidden_artifact_findings",
          "factual_summary_status",
          "tuning_recommendation_provided",
      ]

      missing = [key for key in required_keys if key not in data]
      if missing:
          raise SystemExit(f"Missing required top-level keys: {missing}")

      allowed_status = {
          "factual_summary_ready",
          "partial_factual_summary",
          "missing_core_training_monitoring",
          "reproducibility_unverified",
          "blocked_insufficient_input",
          "parse_failed",
      }

      if data.get("report_type") != "training_run_factual_analysis":
          raise SystemExit(f"Unexpected report_type: {data.get('report_type')!r}")

      if data.get("generated_by") != "codex":
          raise SystemExit(f"Unexpected generated_by: {data.get('generated_by')!r}")

      if data.get("factual_summary_status") not in allowed_status:
          raise SystemExit(f"Invalid factual_summary_status: {data.get('factual_summary_status')!r}")

      if data.get("tuning_recommendation_provided") is not False:
          raise SystemExit("tuning_recommendation_provided must be false")

      print("schema completeness check passed")
      PY
    - git status --short --branch
    - git diff -- <output_json_path>
    - git diff --check
    - git diff --cached --check
  Also verify that the training repository was not modified and that no forbidden artifacts were copied.

Commit and push policy:
  Commit and push after validation passes and only <output_json_path> changed, unless the user explicitly disables commit/push for the task. Do not commit or push if any forbidden file changed, the JSON is invalid, the training repository changed, source run outputs changed, or a forbidden artifact was copied.

Final report requirements:
  Report the output JSON path, files changed, validation commands and results, schema completeness validation result, factual_summary_status value, confirmation that tuning_recommendation_provided is false, whether the training repository was modified, whether forbidden artifacts were copied, whether commit and push were performed, commit hash when available, push target branch when available, the reason if commit/push was not performed, remaining risks, and confirmation that no tuning recommendation or decision was provided.
```

## 7. Update Policy

When a reserved interface obtains an accepted `SKILL.md` file, replace its `Skill identity` and `Expected skill file` fields with the canonical concrete `Skill invocation` field. Keep the surrounding interface structure unchanged.

Use stable interface wording. All callable skill invocations must use the same local Markdown-link format. Do not preserve obsolete prompt forms.
