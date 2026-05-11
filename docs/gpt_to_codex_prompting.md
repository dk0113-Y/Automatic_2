# GPT-to-Codex Prompting

## 1. Role

This document defines GPT-side prompt packaging for routine Codex tasks in `Automatic_2`. GPT uses it after selecting a Codex task, so the prompt is complete and directly executable.

Task details come from the concrete skill contract. `docs/training_system_manifest.md` may be included as passive engineering context.

## 2. Prompt Block Shape

Generated Codex prompts are one English prompt block with these sections:

- Task title
- Task type
- Skill invocation, when a concrete skill exists
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

Prompt blocks omit model settings and reasoning settings.

## 3. Global Packaging Rules

Routine prompts include execution scope, source inputs, allowed writes, validation, commit and push policy, and final report requirements. Generated prompts use the selected task interface and concrete skill.

Copy the Skill invocation exactly from the selected task interface when a concrete skill exists. Interfaces without a concrete Skill invocation remain reserved until an accepted local `SKILL.md` exists and the interface records the concrete invocation.

Training repository writes or source output writes require explicit task authorization. Archive and factual-analysis prompts do not copy checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts.

Tracked outputs use repository-relative paths or stable labels where possible. JSON outputs require JSON parse validation plus the schema and field checks required by the concrete skill.

Codex must commit and push after validation passes and only allowed files changed, unless the user explicitly disables commit/push for the task. Codex must not commit or push when validation fails, forbidden files changed, forbidden artifacts were copied, JSON outputs are invalid, the training repository was modified without authorization, source run outputs were modified, or a task-specific blocking condition is present.

Final reports include files changed, validation results, remaining risks, commit and push status, commit hash when available, push target branch when available, and skipped commit or push reasons when applicable.

## 4. Routine Task Interfaces

### 4.1 training_run_factual_analysis_task

Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)

Required input:
- `source_run_dir`
- `run_name`

Default output:
- `training_results/current/current_training_run_analysis.json`

Prompt requirements:
- Verify reproducible-launch evidence first.
- Treat train-side monitoring as primary factual evidence.
- Treat post-hoc selection as checkpoint-selection context.
- Treat `final_probe` as supplemental validation only.
- Write JSON-only factual output.
- Keep `tuning_recommendation_provided=false`.

Default allowed writes:
- `training_results/current/current_training_run_analysis.json`

Default forbidden scope:
- training repository
- source run outputs
- `DRL_automatic`
- Codex skills
- `README.md`
- history, unless archive writing is explicitly requested

Validation focus: JSON parse validation, schema completeness validation by skill contract, `factual_summary_status` validation, `tuning_recommendation_provided=false` validation, diff-scope validation, repository/source-output status checks, and forbidden-artifact checks.

### 4.2 single_run_analysis_archive_task

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Required input:
- `training_results/current/current_training_run_analysis.json`
- `tuning_review_payload` or `tuning_review_json_source`
- `archive_id` derived from `run_name` or supplied by the user

Default output concept:
- `training_results/history/single_runs/<archive_id>/training_run_analysis.json`
- `training_results/history/single_runs/<archive_id>/tuning_review.json`
- `training_results/history/history_index.json`

Prompt requirements:
- Preserve the source factual analysis as the Codex-owned factual record.
- Preserve the GPT tuning-review payload as the GPT-owned rationale record.
- Update `history_index.json` with one paired archive entry using repository-relative paths.
- Use `source_training_repo` as the portable working-directory label for external training repository command metadata.
- Use `<source_training_repo>` in recorded command templates.
- Use `recommended_next_command_summary.command_arguments` for concrete launcher parameters.

Validation focus: source and tuning-review JSON parse/report-type checks, archive id conflict check, history index single-entry check, allowed-file diff check, no `archive_manifest.json`, and no forbidden artifact copy.

### 4.3 multi_run_analysis_archive_task

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Required input:
- `training_results/current/current_multi_run_analysis.json`
- `tuning_review_payload` or `tuning_review_json_source`
- `archive_id` derived from `plan_id`, run group id, or supplied by the user

Default output concept:
- `training_results/history/multi_runs/<archive_id>/multi_run_analysis.json`
- `training_results/history/multi_runs/<archive_id>/tuning_review.json`
- `training_results/history/history_index.json`

Prompt requirements:
- Preserve the source multi-run factual analysis.
- Preserve the GPT tuning-review payload.
- Update `history_index.json` with one paired archive entry using repository-relative paths.
- Use `source_training_repo`, `<source_training_repo>`, and `command_arguments` for portable command metadata.

Validation focus: source and tuning-review JSON parse/report-type checks, declared multi-run report-type check, archive id conflict check, history index single-entry check, allowed-file diff check, no `archive_manifest.json`, and no forbidden artifact copy.

### 4.4 Reserved multi_run_script_generation_task

Skill identity:
- `$multi-run-script-generation`

Expected skill file:
- `.agents/skills/multi-run-script-generation/SKILL.md`

Required input:
- GPT-provided parameter plan
- training script target path or command output target
- execution mode constraints

Default output concept:
- generated multi-run script or launch-command file

Prompt requirements:
- Include explicit write authorization for any training-repository target.
- Keep execution and launch behavior explicit in the task prompt.
- Align task details with the concrete skill when this interface becomes callable.

### 4.5 Reserved multi_run_factual_analysis_task

Skill identity:
- `$multi-run-factual-analysis`

Expected skill file:
- `.agents/skills/multi-run-factual-analysis/SKILL.md`

Required input:
- list of `source_run_dir` entries
- run labels
- comparison scope

Default output:
- `training_results/current/current_multi_run_analysis.json`

Prompt requirements:
- Verify reproducible-launch evidence per run.
- Treat train-side monitoring as primary factual evidence per run.
- Provide cross-run factual comparison only.
- Treat `final_probe` as supplemental validation only.
- Keep write scope explicit and bounded to the requested current multi-run output.

Validation focus: JSON parse validation, schema completeness validation by future skill contract, per-run reproducibility checks, diff-scope validation, repository/source-output status checks, and forbidden-artifact checks.

## 5. Update Policy

When a reserved interface obtains an accepted `SKILL.md` file, replace its `Skill identity` and `Expected skill file` fields with the canonical concrete `Skill invocation` field. Keep the surrounding interface structure stable.

All callable skill invocations use the canonical local Markdown-link format.
