---
name: training-run-factual-analysis
description: Inspect one completed DRL-path-finding run for compact neutral factual JSON centered on train-side monitoring. Verify reproducible launch first. Treat posthoc/final_probe as conditional legacy or eval-enabled artifacts. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## Contract

| Field | Contract |
| --- | --- |
| Inputs | `source_run_dir`; `output_report_path`; `destination_directory`; `run_name` |
| Output | compact `training_run_factual_analysis` JSON |
| Primary evidence | `reproducible_launch`; `train_side_monitoring`; `configuration_runtime` |
| Conditional evidence | `posthoc_selection` and `supplemental_final_probe` only when enabled and present |
| Evidence storage | source artifacts remain full evidence; output JSON is a compact review-facing digest |
| Boundary | factual-only; `tuning_recommendation_provided=false` |
| Excluded decisions | tuning recommendations; next hyperparameters; baseline acceptance; stop/continue/branch; method or paper decisions |
| Input blocker | `blocked_insufficient_input` for missing, inaccessible, non-directory, or ambiguous `source_run_dir` |

Train-side-only contract:
- If `train_side_only_tuning=true`, posthoc/final_probe absence is intentional.
- Record each skipped object with `status=skipped_train_side_only_tuning`, `role=not_applicable_for_train_side_tuning`, and `missingness_treatment=not_missing`.
- Do not synthesize `winner_step`, `selected_candidate_steps`, checkpoint winners, candidate rankings, final_probe summaries, best-vs-last summaries, or test-side metrics.
- Skipped posthoc/final_probe must not lower `factual_summary_status` when reproducibility and core train-side monitoring are ready.

## Mode Detection

Required source artifacts:

| Source | Execution-useful fields |
| --- | --- |
| `logs/config_snapshot.json` | `train_side_only_tuning`; eval/posthoc/final_probe config; runtime/config; observed run contract |
| `logs/metric_snapshot.json` | train endpoint summaries; optional final_probe summaries; skipped/eval status |
| `logs/artifact_index.json` | artifact presence; `formal_final_object_role`; enabled/present/skipped/missing/malformed status |
| `logs/reproducibility_contract.json` | strict backend status; seed status; reproducibility verdict |
| launch/stdout/argv summaries when available | command identity; source branch/commit; observed run contract |

State table:

| Output state field | Allowed values |
| --- | --- |
| `train_side_only_tuning` | `true`; `false`; `unknown` |
| `test_side_evaluation_status` | `enabled`; `disabled`; `skipped_train_side_only_tuning`; `missing`; `malformed`; `unknown` |
| `skipped_test_side_evaluation.reason` | compact source value; `train_side_only_tuning`; `unknown` |
| `formal_final_object_role` | compact source role; `not_applicable_for_train_side_tuning`; `unknown` |
| `posthoc_selection.status` | `present`; `enabled_missing`; `skipped_train_side_only_tuning`; `missing`; `malformed`; `unknown` |
| `supplemental_final_probe.status` | `present`; `enabled_missing`; `skipped_train_side_only_tuning`; `missing`; `malformed`; `unknown` |
| `posthoc_selection.role` / `supplemental_final_probe.role` | `not_applicable_for_train_side_tuning`; compact source role; `unknown` |
| `missingness_treatment` | `not_missing`; `missing`; `malformed`; `unknown` |

## Extraction

Extraction order:
1. `reproducible_launch`
2. `train_side_mode_detection`
3. `train_side_monitoring`
4. `configuration_runtime`
5. `optional_test_side_artifacts`
6. `missingness_parseability`

`reproducible_launch`:
- Inspect `logs/reproducibility_contract.json`, `logs/config_snapshot.json`, `logs/artifact_index.json`, launch records, stdout summaries, and captured argv records.
- Extract `contract_verdict`, `strict_reproducibility`, deterministic algorithm status, `PYTHONHASHSEED`, `CUBLAS_WORKSPACE_CONFIG`, backend requested flags, backend runtime readbacks, train seed status, eval seed status when enabled, and source branch/commit when available.
- Set status to `reproducible_launch_confirmed`, `reproducible_launch_not_confirmed`, `reproducible_launch_contradictory`, `reproducibility_artifacts_missing`, or `reproducibility_unverified`.
- Verify from artifacts/launch records; do not infer from run name.

`train_side_monitoring`:

Inspect `logs/train_steps.csv`, `logs/train_episodes.csv`, and `logs/metric_snapshot.json`.

| Group | Required compact fields |
| --- | --- |
| `row_counts` | train step rows; train episode rows; empty CSV status; parsed row counts |
| `parseability` | readable files; malformed files; NaN/nonfinite handling |
| `endpoint_metrics` | `reward`; `coverage`; `success_rate`; `episode_length`; `repeat_visit_ratio` / `RVR`; `timeout` |
| `initial_to_endpoint_delta` | compact start-to-end train-side deltas |
| `late_stage_delta` | compact late-stage deltas or direction when safe |
| `key_behavior_diagnostics` | `zero_info`; `recent_revisit`; `stall`; `turn_ge_90`; `turn_135`; `turn_180`; `timeout_flag`; `turn_penalty_weight_sum` when present |
| `derived_exploration_efficiency` | `coverage_gain_per_step`; `weighted_info_gain_per_step`; `zero_info_rate`; `recent_revisit_rate`; `stall_rate`; `turn_burden_rate`; `timeout_rate`; `metric_role=diagnostic_only`; `evidence_role=not_standalone_superiority_evidence`; used to explain mechanism, not decide superiority alone |
| `reward_breakdown` | `info_reward_sum`; `step_penalty_sum`; `recent_revisit_penalty_sum`; `turn_penalty_sum`; `timeout_penalty_sum`; `terminal_bonus_sum` |
| `reward_event_summary` | `delta_empty_sum`; `delta_obstacle_sum`; `empty_info_gain_sum`; `obstacle_info_gain_sum`; `weighted_obstacle_info_gain_sum`; `weighted_info_gain_sum`; `obstacle_info_contribution_ratio`; `recent_revisit_trigger_count`; `stall_trigger_count`; `zero_info_step_count`; `turn_ge_90_count`; `turn_135_count`; `turn_180_count`; `timeout_flag` |
| `learner_replay_exploration_context` | `loss`; `q_mean`; `target_q_mean`; `td_abs_mean`; `grad_norm`; `replay_size`; `epsilon`; `batch_size`; `replay_capacity`; `n_step`; `gamma`; `learning_rate`; `target_update_interval`; `collect_steps_per_iter`; `learner_updates_per_iter`; `train_every_env_steps` |
| `missing_expected_columns` | required train-side columns absent from parsed artifacts |

`configuration_runtime`:
- Preserve `total_env_steps`, `budget_mode`, `total_train_episodes`, `final_greedy_episodes`, `epsilon_decay_steps`, `epsilon_end`, `min_replay_size`.
- Preserve replay/learner/update config values, reward config values needed to interpret reward breakdown, and map/eval config fields needed for compact comparison.
- Preserve `train_side_only_tuning`, `test_side_evaluation_status`, skipped reason, device, runtime duration, source git branch/commit, strict backend status, and seed status.

`optional_test_side_artifacts`:
- For `train_side_only_tuning=true`, set `posthoc_selection` and `supplemental_final_probe` to `status=skipped_train_side_only_tuning`, `role=not_applicable_for_train_side_tuning`, `missingness_treatment=not_missing`; include no synthetic test-side facts.
- For legacy/eval-enabled runs, extract posthoc/final_probe compactly only when enabled and present.
- Keep posthoc as checkpoint-selection context and final_probe as supplemental; neither may become primary evidence.

`missingness_parseability`:
- Report files found, missing artifacts, parse failures, malformed JSON, missing required columns, empty CSVs, unverified items, skipped train-side-only artifacts, scope-skipped artifacts, and factual summary sufficiency.
- Status enum: `factual_summary_ready`; `partial_factual_summary`; `missing_core_training_monitoring`; `reproducibility_unverified`; `blocked_insufficient_input`; `parse_failed`.

## JSON Output

Authorized write: one compact JSON report at `output_report_path`.

Top-level keys:
- `schema_version`; `report_type`; `generated_by`; `source_run_dir`; `run_name`
- `files_inspected`; `commands_run`; `reproducible_launch_status`; `reproducible_launch`
- `train_side_monitoring`; `posthoc_selection`; `supplemental_final_probe`; `configuration_and_runtime`
- `missing_artifacts`; `parse_failures`; `unverified_items`; `forbidden_artifact_findings`
- `factual_summary_status`; `tuning_recommendation_provided`

Constants:
- `schema_version=1.0`
- `report_type=training_run_factual_analysis`
- `generated_by=codex`
- `tuning_recommendation_provided=false`

Required `train_side_monitoring` substructure:
- `row_counts`; `parseability`; `endpoint_metrics`; `initial_to_endpoint_delta`; `late_stage_delta`
- `key_behavior_diagnostics`; `derived_exploration_efficiency`; `reward_breakdown`; `reward_event_summary`
- `learner_replay_exploration_context`; `missing_expected_columns`

Include:
- gate status and blocking missingness; identity and sanitized source path; mode detection
- compact configuration/runtime; endpoint train-side metrics; train-side deltas; key diagnostics
- derived exploration-efficiency metrics; reward breakdown; learner/replay/exploration context
- optional skipped/eval status for test-side artifacts; validation facts

Exclude:
- full CSV rows or headers; full logs or command traces; broad numeric dumps; full artifact inventories
- checkpoint payloads or model weights; plots; trajectories; raw outputs; binary artifacts
- private absolute source paths in tracked JSON; tuning decisions or next hyperparameters
- baseline acceptance decisions; stop/continue/branch decisions

## Validation

Required checks:
- `schema_and_constants`; `mode_detection_contract`; `train_side_monitoring_contract`; `derived_metrics_contract`
- `test_side_skipped_contract`; `factual_summary_status_contract`; `density_contract`; `path_sanitization`
- `write_scope`; `source_integrity`; `factual_only_boundary`

Factual summary rule:
- Use `factual_summary_ready` when reproducibility and core train-side monitoring are ready, even when posthoc/final_probe are skipped by `train_side_only_tuning`.

## Blockers

- `write_scope_violation`: writes outside `output_report_path`.
- `source_integrity_violation`: training code or source run outputs modified.
- `artifact_copy_violation`: checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts copied.
- `path_privacy_violation`: tracked JSON contains private absolute source paths.
- `decision_scope_violation`: tuning, next-parameter, baseline, stop/continue, branch, method, or paper decisions included.
- `evidence_role_violation`: posthoc/final_probe treated as primary evidence.
- `train_side_only_missingness_violation`: skipped posthoc/final_probe under `train_side_only_tuning` treated as missing.
- `output_density_violation`: JSON contains source-artifact dumps instead of compact summaries.
