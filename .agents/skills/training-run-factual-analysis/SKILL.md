---
name: training-run-factual-analysis
description: Train-side-only factual JSON.
---

# Training Run Factual Analysis

## Contract

| Field | Contract |
| --- | --- |
| Inputs | `source_run_dir`; `output_report_path`; `destination_directory`; `run_name` |
| Output | `training_run_factual_analysis` JSON |
| Primary evidence | `reproducible_launch`; `train_side_monitoring`; `configuration_runtime` |
| Test-side objects | `posthoc_selection`; `supplemental_final_probe`; status-only; never evidence |
| Evidence storage | source artifacts are full evidence; JSON is compact digest |
| Boundary | factual-only; `tuning_recommendation_provided=false` |
| Excluded decisions | no tuning recommendations, next hyperparameters, baseline acceptance, stop/continue/branch, method, paper decisions |
| Input blocker | `blocked_insufficient_input` for missing/inaccessible/non-directory/ambiguous `source_run_dir` |

| Mode case | Required handling |
| --- | --- |
| Normal | `train_side_only_tuning=true` |
| Skipped posthoc/final_probe | `status=skipped_train_side_only_tuning`; `role=not_applicable_for_train_side_tuning`; `missingness_treatment=not_missing`; intentional absence |
| Unexpected posthoc/final_probe | `status=unexpected_present`; `role=not_used_for_train_side_tuning`; `missingness_treatment=not_missing`; presence only; not evidence |
| False/unknown train-side-only | `status=unsupported_non_train_side_only_output` or `unknown_mode`; no legacy metric extraction; `factual_summary_status` may be `partial_factual_summary` unless core evidence is sufficient and mode issue non-blocking |
| Forbidden test-side facts | no `winner_step`, `selected_candidate_steps`, checkpoint winners, candidate rankings, final_probe summaries, best-vs-last summaries, test-side metrics |
| Summary rule | skipped posthoc/final_probe must not lower `factual_summary_status` when reproducibility and core train-side monitoring are ready |

## Mode Detection

| Source | Use |
| --- | --- |
| `logs/config_snapshot.json` | mode/config/runtime/run contract |
| `logs/metric_snapshot.json` | train endpoints; test-side status |
| `logs/artifact_index.json` | artifact presence; `formal_final_object_role`; missing/malformed |
| `logs/reproducibility_contract.json` | backend; seed; strict reproducibility; verdict |
| launch/stdout/argv summaries when available | command identity; source branch/commit; run contract |

| State field | Values |
| --- | --- |
| `train_side_only_tuning` | `true`; `false`; `unknown` |
| `test_side_evaluation_status` | `skipped_train_side_only_tuning`; `unsupported_non_train_side_only_output`; `unknown_mode`; `missing`; `malformed` |
| `skipped_test_side_evaluation.reason` | compact source value; `train_side_only_tuning`; `unknown` |
| `formal_final_object_role` | `not_applicable_for_train_side_tuning`; `not_used_for_train_side_tuning`; `unknown` |
| `posthoc_selection.status`; `supplemental_final_probe.status` | `skipped_train_side_only_tuning`; `unexpected_present`; `unsupported_non_train_side_only_output`; `unknown_mode`; `missing`; `malformed` |
| `posthoc_selection.role`; `supplemental_final_probe.role` | `not_applicable_for_train_side_tuning`; `not_used_for_train_side_tuning`; `unknown` |
| `missingness_treatment` | `not_missing`; `missing`; `malformed`; `unknown` |

## Extraction

Order: `reproducible_launch`; `train_side_mode_detection`; `train_side_monitoring`; `configuration_runtime`; `test_side_status_only`; `missingness_parseability`.

| Object | Extract |
| --- | --- |
| `reproducible_launch` | `contract_verdict`; `strict_reproducibility`; deterministic algorithm status; `PYTHONHASHSEED`; `CUBLAS_WORKSPACE_CONFIG`; backend requested flags; backend runtime readbacks; train seed status; eval seed status only as mode/status metadata if present; source branch/commit when available. Status: `reproducible_launch_confirmed`; `reproducible_launch_not_confirmed`; `reproducible_launch_contradictory`; `reproducibility_artifacts_missing`; `reproducibility_unverified`. Verify records; do not infer from run name. |
| `configuration_runtime` | `total_env_steps`; `budget_mode`; `total_train_episodes`; `final_greedy_episodes`; `epsilon_decay_steps`; `epsilon_end`; `min_replay_size`; replay/learner/update config; reward config for breakdown; map/eval config for compact comparison; mode/test-side status/skipped reason; device; duration; source git branch/commit; strict backend status; seed status. |
| `test_side_status_only` | Skipped: status/role/not-missing. Unexpected: presence status only. False/unknown mode: unsupported/unknown status. Do not extract posthoc/final_probe metrics, `winner_step`, rankings, summaries, held-out results, or evidence. |
| `missingness_parseability` | found/missing files; parse failures; malformed JSON; missing columns; empty CSVs; unverified items; skipped train-side-only artifacts; unexpected test-side artifacts; scope-skipped artifacts; sufficiency. Status: `factual_summary_ready`; `partial_factual_summary`; `missing_core_training_monitoring`; `reproducibility_unverified`; `blocked_insufficient_input`; `parse_failed`. |

`train_side_monitoring`: inspect `logs/train_steps.csv`, `logs/train_episodes.csv`, `logs/metric_snapshot.json`.

| Group | Fields |
| --- | --- |
| `row_counts` | train step rows; train episode rows; empty CSV status; parsed counts |
| `parseability` | readable/malformed files; NaN/nonfinite handling |
| `endpoint_metrics` | `reward`; `coverage`; `success_rate`; `episode_length`; `repeat_visit_ratio` / `RVR`; `timeout` |
| `initial_to_endpoint_delta` | compact start-to-end deltas |
| `late_stage_delta` | compact late-stage deltas/direction when safe |
| `key_behavior_diagnostics` | `zero_info`; `recent_revisit`; `stall`; `turn_ge_90`; `turn_135`; `turn_180`; `timeout_flag`; `turn_penalty_weight_sum` when present |
| `derived_exploration_efficiency` | `coverage_gain_per_step`; `weighted_info_gain_per_step`; `zero_info_rate`; `recent_revisit_rate`; `stall_rate`; `turn_burden_rate`; `timeout_rate`; `metric_role=diagnostic_only`; `evidence_role=not_standalone_superiority_evidence`; explain mechanism, not superiority alone |
| `reward_breakdown` | `info_reward_sum`; `step_penalty_sum`; `recent_revisit_penalty_sum`; `turn_penalty_sum`; `timeout_penalty_sum`; `terminal_bonus_sum` |
| `reward_event_summary` | `delta_empty_sum`; `delta_obstacle_sum`; `empty_info_gain_sum`; `obstacle_info_gain_sum`; `weighted_obstacle_info_gain_sum`; `weighted_info_gain_sum`; `obstacle_info_contribution_ratio`; `recent_revisit_trigger_count`; `stall_trigger_count`; `zero_info_step_count`; `turn_ge_90_count`; `turn_135_count`; `turn_180_count`; `timeout_flag` |
| `learner_replay_exploration_context` | `loss`; `q_mean`; `target_q_mean`; `td_abs_mean`; `grad_norm`; `replay_size`; `epsilon`; `batch_size`; `replay_capacity`; `n_step`; `gamma`; `learning_rate`; `target_update_interval`; `collect_steps_per_iter`; `learner_updates_per_iter`; `train_every_env_steps` |
| `missing_expected_columns` | absent required train-side columns |

## JSON Output

| Item | Requirement |
| --- | --- |
| Authorized write | compact JSON at `output_report_path` only |
| Constants | `schema_version=1.0`; `report_type=training_run_factual_analysis`; `generated_by=codex`; `tuning_recommendation_provided=false` |
| Required top-level keys | `schema_version`; `report_type`; `generated_by`; `source_run_dir`; `run_name`; `files_inspected`; `commands_run`; `reproducible_launch_status`; `reproducible_launch`; `train_side_monitoring`; `posthoc_selection`; `supplemental_final_probe`; `configuration_and_runtime`; `missing_artifacts`; `parse_failures`; `unverified_items`; `forbidden_artifact_findings`; `factual_summary_status`; `tuning_recommendation_provided` |
| Required `train_side_monitoring` keys | `row_counts`; `parseability`; `endpoint_metrics`; `initial_to_endpoint_delta`; `late_stage_delta`; `key_behavior_diagnostics`; `derived_exploration_efficiency`; `reward_breakdown`; `reward_event_summary`; `learner_replay_exploration_context`; `missing_expected_columns` |
| Include | gate status/blocking missingness; identity/sanitized source path; mode detection; compact configuration/runtime; endpoint train-side metrics; train-side deltas; diagnostics; derived efficiency metrics; reward breakdown; learner/replay/exploration context; skipped/unexpected test-side status only; validation facts |
| Exclude | full CSV rows/headers; full logs/command traces; broad numeric dumps; full artifact inventories; checkpoints/model weights; plots; trajectories; raw outputs; binary artifacts; private absolute source paths; tuning decisions; next hyperparameters; baseline acceptance; stop/continue/branch decisions; posthoc/final_probe metric extraction |

## Validation

| Check | Requirement |
| --- | --- |
| `schema_and_constants` | required keys and constants present |
| `mode_detection_contract` | state fields and allowed values present |
| `train_side_monitoring_contract` | required train-side groups and endpoint metrics present |
| `derived_metrics_contract` | exactly seven derived diagnostics plus `metric_role=diagnostic_only` and `evidence_role=not_standalone_superiority_evidence` |
| `test_side_status_only_contract` | posthoc/final_probe skipped or unexpected status only; no metrics, winners, rankings, summaries, or evidence use |
| `factual_summary_status_contract` | `factual_summary_ready` when reproducibility and core train-side monitoring are ready, even with skipped posthoc/final_probe |
| `density_contract` | compact digest; no source-artifact dumps |
| `path_sanitization` | repository-relative paths or stable labels; no private absolute source paths |
| `write_scope` | write only `output_report_path` |
| `source_integrity` | no source artifact modification or artifact copying |
| `factual_only_boundary` | no tuning recommendation or decision fields beyond `tuning_recommendation_provided=false` |

## Blockers

- `write_scope_violation`: write outside `output_report_path`.
- `source_integrity_violation`: training code or source run outputs modified.
- `artifact_copy_violation`: checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts copied.
- `path_privacy_violation`: tracked JSON contains private absolute source paths.
- `decision_scope_violation`: tuning, next-parameter, baseline, stop/continue, branch, method, or paper decisions included.
- `evidence_role_violation`: posthoc/final_probe used as evidence.
- `train_side_only_missingness_violation`: skipped posthoc/final_probe under `train_side_only_tuning` treated as missing.
- `output_density_violation`: JSON contains source-artifact dumps.
- `test_side_metric_extraction_violation`: posthoc/final_probe metrics, winners, rankings, or summaries extracted.
