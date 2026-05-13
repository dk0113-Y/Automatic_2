---
name: training-run-factual-analysis
description: Inspect completed DRL-path-finding run for compact train-side-only factual JSON. Record posthoc/final_probe status only. No tuning recommendations/decisions.
---

# Training Run Factual Analysis

## Contract

| Field | Contract |
| --- | --- |
| Inputs | `source_run_dir`; `output_report_path`; `destination_directory`; `run_name` |
| Output | compact `training_run_factual_analysis` JSON |
| Primary evidence | `reproducible_launch`; `train_side_monitoring`; `configuration_runtime` |
| Test-side objects | `posthoc_selection`; `supplemental_final_probe`; status-only; never evidence |
| Evidence storage | source artifacts full evidence; JSON compact review-facing digest |
| Boundary | factual-only; `tuning_recommendation_provided=false` |
| Excluded decisions | no tuning recommendations; next hyperparameters; baseline acceptance; stop/continue/branch; method/paper decisions |
| Input blocker | `blocked_insufficient_input` for missing/inaccessible/non-directory/ambiguous `source_run_dir` |

| Mode case | Required handling |
| --- | --- |
| Normal mode | `train_side_only_tuning=true` |
| Skipped posthoc/final_probe | intentional absence; `status=skipped_train_side_only_tuning`; `role=not_applicable_for_train_side_tuning`; `missingness_treatment=not_missing` |
| Unexpected posthoc/final_probe | `status=unexpected_present`; `role=not_used_for_train_side_tuning`; `missingness_treatment=not_missing`; not evidence |
| Non-train-side mode | `status=unsupported_non_train_side_only_output`; no fallback metric extraction |
| Unknown mode | `status=unknown_mode`; no fallback metric extraction |
| Forbidden facts | no `winner_step`; no `selected_candidate_steps`; no checkpoint winners, candidate rankings, final_probe summaries, best-vs-last summaries, or test-side metrics |
| Summary rule | skipped posthoc/final_probe must not lower `factual_summary_status` when reproducibility/core train-side monitoring are ready |

## Mode Detection

Required source artifacts:

| Source | Fields |
| --- | --- |
| `logs/config_snapshot.json` | `train_side_only_tuning`; test-side skip/config; runtime/config; run contract |
| `logs/metric_snapshot.json` | train endpoints; skipped/unexpected test-side status |
| `logs/artifact_index.json` | artifact presence; `formal_final_object_role`; skipped/unexpected/missing/malformed |
| `logs/reproducibility_contract.json` | backend status; seed status; reproducibility verdict |
| launch/stdout/argv summaries when available | command identity; source branch/commit; observed run contract |

State table:

| Output state field | Values |
| --- | --- |
| `train_side_only_tuning` | `true`; `false`; `unknown` |
| `test_side_evaluation_status` | `skipped_train_side_only_tuning`; `unsupported_non_train_side_only_output`; `unknown_mode`; `missing`; `malformed` |
| `skipped_test_side_evaluation.reason` | compact source value; `train_side_only_tuning`; `unknown` |
| `formal_final_object_role` | `not_applicable_for_train_side_tuning`; `not_used_for_train_side_tuning`; `unknown` |
| `posthoc_selection.status` | `skipped_train_side_only_tuning`; `unexpected_present`; `unsupported_non_train_side_only_output`; `unknown_mode`; `missing`; `malformed` |
| `supplemental_final_probe.status` | `skipped_train_side_only_tuning`; `unexpected_present`; `unsupported_non_train_side_only_output`; `unknown_mode`; `missing`; `malformed` |
| `posthoc_selection.role` / `supplemental_final_probe.role` | `not_applicable_for_train_side_tuning`; `not_used_for_train_side_tuning`; `unknown` |
| `missingness_treatment` | `not_missing`; `missing`; `malformed`; `unknown` |

## Extraction

Order: `reproducible_launch`; `train_side_mode_detection`; `train_side_monitoring`; `configuration_runtime`; `test_side_status_only`; `missingness_parseability`.

| Object | Required extraction |
| --- | --- |
| `reproducible_launch` | Inspect `logs/reproducibility_contract.json`, `logs/config_snapshot.json`, `logs/artifact_index.json`, launch/stdout/argv records. Extract `contract_verdict`, `strict_reproducibility`, deterministic algorithm status, `PYTHONHASHSEED`, `CUBLAS_WORKSPACE_CONFIG`, backend flags/readbacks, train seed status, eval seed status only as mode/status metadata if present, source branch/commit when available. Status: `reproducible_launch_confirmed`; `reproducible_launch_not_confirmed`; `reproducible_launch_contradictory`; `reproducibility_artifacts_missing`; `reproducibility_unverified`. Verify records; do not infer from run name. |
| `configuration_runtime` | Preserve budget, replay/learner/update, reward interpretation, map/eval comparison, mode, device, duration, git branch/commit, backend, seed fields: `total_env_steps`, `budget_mode`, `total_train_episodes`, `final_greedy_episodes`, `epsilon_decay_steps`, `epsilon_end`, `min_replay_size`, `train_side_only_tuning`, `test_side_evaluation_status`, skipped reason. |
| `test_side_status_only` | Skipped objects: skipped status/role/not-missing. Unexpected artifacts: compact presence status only. False/unknown `train_side_only_tuning`: unsupported/unknown status. Do not extract posthoc/final_probe metrics, winners, rankings, summaries, or held-out results. `factual_summary_status` may be `partial_factual_summary` unless core evidence is sufficient and mode issue non-blocking. |
| `missingness_parseability` | Report found/missing files, parse failures, malformed JSON, missing columns, empty CSVs, unverified items, skipped train-side-only artifacts, unexpected test-side artifacts, scope-skipped artifacts, sufficiency. Status: `factual_summary_ready`; `partial_factual_summary`; `missing_core_training_monitoring`; `reproducibility_unverified`; `blocked_insufficient_input`; `parse_failed`. |

`train_side_monitoring`: inspect `logs/train_steps.csv`, `logs/train_episodes.csv`, and `logs/metric_snapshot.json`.

| Group | Fields |
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
| `missing_expected_columns` | absent required train-side columns |

## JSON Output

Authorized write: compact JSON at `output_report_path`.

Top-level keys:
- `schema_version`; `report_type`; `generated_by`; `source_run_dir`; `run_name`
- `files_inspected`; `commands_run`; `reproducible_launch_status`; `reproducible_launch`
- `train_side_monitoring`; `posthoc_selection`; `supplemental_final_probe`; `configuration_and_runtime`
- `missing_artifacts`; `parse_failures`; `unverified_items`; `forbidden_artifact_findings`
- `factual_summary_status`; `tuning_recommendation_provided`

Constants: `schema_version=1.0`; `report_type=training_run_factual_analysis`; `generated_by=codex`; `tuning_recommendation_provided=false`.

`train_side_monitoring` substructure:
- `row_counts`; `parseability`; `endpoint_metrics`; `initial_to_endpoint_delta`; `late_stage_delta`
- `key_behavior_diagnostics`; `derived_exploration_efficiency`; `reward_breakdown`; `reward_event_summary`
- `learner_replay_exploration_context`; `missing_expected_columns`

Include:
- gate status/blocking missingness; identity/sanitized source path; mode detection
- compact configuration/runtime; endpoint train-side metrics; train-side deltas; key diagnostics
- derived exploration-efficiency metrics; reward breakdown; learner/replay/exploration context
- skipped/unexpected test-side status only; validation facts

Exclude:
- full CSV rows or headers; full logs or command traces; broad numeric dumps; full artifact inventories
- checkpoint payloads or model weights; plots; trajectories; raw outputs; binary artifacts
- private absolute source paths in tracked JSON; tuning decisions; next hyperparameters
- baseline acceptance decisions; stop/continue/branch decisions; posthoc/final_probe metric extraction

## Validation

Required checks:
- `schema_and_constants`; `mode_detection_contract`; `train_side_monitoring_contract`; `derived_metrics_contract`
- `test_side_status_only_contract`; `factual_summary_status_contract`; `density_contract`; `path_sanitization`
- `write_scope`; `source_integrity`; `factual_only_boundary`

Factual summary rule: use `factual_summary_ready` when reproducibility/core train-side monitoring are ready, even when posthoc/final_probe are skipped by `train_side_only_tuning`.

## Blockers

- `write_scope_violation`: write outside `output_report_path`.
- `source_integrity_violation`: training code/source run outputs modified.
- `artifact_copy_violation`: checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts copied.
- `path_privacy_violation`: tracked JSON contains private absolute source paths.
- `decision_scope_violation`: tuning, next-parameter, baseline, stop/continue, branch, method, or paper decisions included.
- `evidence_role_violation`: posthoc/final_probe used as evidence.
- `train_side_only_missingness_violation`: skipped posthoc/final_probe under `train_side_only_tuning` treated as missing.
- `output_density_violation`: JSON contains source-artifact dumps.
- `test_side_metric_extraction_violation`: posthoc/final_probe metrics, winners, rankings, or summaries extracted.
