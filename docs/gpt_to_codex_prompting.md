# GPT-to-Codex Prompting

Use this file after a Codex task is selected. Generate one executable English Codex prompt from the selected task interface; task details come from the concrete skill.

## 1. Codex Prompt Requirements

Generated prompts are one English prompt block with: Task title; Task type; Skill invocation copied exactly from the selected task interface when concrete; Working context; Source inputs; Output destination; Goal; Required reading; Allowed writes; Forbidden writes; Required procedure; Validation; Commit and push policy; Final report requirements.

Rules:
- Interfaces without concrete Skill invocation remain reserved until an accepted local `SKILL.md` exists and the interface records the concrete invocation.
- Training repository writes or source output writes require explicit authorization; tracked outputs use repository-relative paths or stable labels where possible.
- Forbidden writes cover checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, and binary artifacts.
- JSON outputs require JSON parse validation and concrete-skill schema checks.
- Codex must commit and push after validation passes and only allowed files changed, unless the user explicitly disables commit/push.
- Codex must not commit or push when validation fails, forbidden files changed, forbidden artifacts were copied, JSON is invalid, the training repository was modified without authorization, source run outputs were modified, or task-specific blockers exist.
- Final reports include files changed, validation results, remaining risks, commit/push status, commit hash when available, and push target branch when available.

## 2. Task Interfaces

### 2.1 training_run_factual_analysis_task

Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)

Inputs: `source_run_dir`; `run_name`

Outputs: `training_results/current/current_training_run_analysis.json`

Prompt: reproducible_launch_first; train_side_monitoring_primary; posthoc_selection_context; final_probe_supplemental; json_only_factual_output; `tuning_recommendation_provided=false`

Writes: `training_results/current/current_training_run_analysis.json`

Forbidden: training_repository; source_run_outputs; `DRL_automatic`; Codex_skills; `README.md`; history_without_archive_request; forbidden_training_artifacts

Validation: json_parse; skill_schema_checks; factual_summary_status; tuning_recommendation_provided_false; diff_scope; repository_and_source_output_status; forbidden_artifacts

### 2.2 single_run_analysis_archive_task

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Inputs: `training_results/current/current_training_run_analysis.json`; `tuning_review_payload` or `tuning_review_json_source`; `archive_id`

Outputs: `training_results/history/single_runs/<archive_id>/training_run_analysis.json`; `training_results/history/single_runs/<archive_id>/tuning_review.json`; `training_results/history/history_index.json`

Prompt: preserve_source_factual_analysis; preserve_gpt_tuning_review_payload; one_paired_history_index_entry; repository_relative_paths; portable_command_metadata: `source_training_repo`, `<source_training_repo>`, `command_arguments`

Validation: source_json_parse_and_report_type; tuning_review_json_parse_and_report_type; archive_id_conflict; history_index_single_entry; allowed_file_diff; no_archive_manifest; no_forbidden_artifacts

### 2.3 multi_run_analysis_archive_task

Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)

Inputs: `training_results/current/current_multi_run_analysis.json`; `tuning_review_payload` or `tuning_review_json_source`; `archive_id`

Outputs: `training_results/history/multi_runs/<archive_id>/multi_run_analysis.json`; `training_results/history/multi_runs/<archive_id>/tuning_review.json`; `training_results/history/history_index.json`

Prompt: preserve_source_multi_run_factual_analysis; preserve_gpt_tuning_review_payload; one_paired_history_index_entry; repository_relative_paths; portable_command_metadata: `source_training_repo`, `<source_training_repo>`, `command_arguments`

Validation: source_json_parse_and_report_type; tuning_review_json_parse_and_report_type; declared_multi_run_report_type; archive_id_conflict; history_index_single_entry; allowed_file_diff; no_archive_manifest; no_forbidden_artifacts

### 2.4 Reserved multi_run_script_generation_task

Skill identity: `$multi-run-script-generation`

Expected skill file: `.agents/skills/multi-run-script-generation/SKILL.md`

Inputs: GPT-provided parameter plan; training script target path or command output target; execution mode constraints

Outputs: generated multi-run script or launch-command file

Prompt: explicit_training_repository_write_authorization; explicit_execution_and_launch_behavior; align_with_concrete_skill_when_callable

### 2.5 Reserved multi_run_factual_analysis_task

Skill identity: `$multi-run-factual-analysis`

Expected skill file: `.agents/skills/multi-run-factual-analysis/SKILL.md`

Inputs: list of `source_run_dir` entries; run labels; comparison scope

Outputs: `training_results/current/current_multi_run_analysis.json`

Prompt: reproducible_launch_per_run; train_side_monitoring_primary_per_run; cross_run_factual_comparison_only; final_probe_supplemental; bounded_current_multi_run_write_scope

Validation: json_parse; future_skill_schema_checks; per_run_reproducibility; diff_scope; repository_and_source_output_status; forbidden_artifacts

## 3. Update Policy

- Reserved interfaces become callable only after an accepted local `SKILL.md` exists.
- Replace `Skill identity` and `Expected skill file` with concrete `Skill invocation`.
- Keep the surrounding interface structure stable.
- Callable skill invocations use the exact local Markdown-link format recorded in the task interface.
