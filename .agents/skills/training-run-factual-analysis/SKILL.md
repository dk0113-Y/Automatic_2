---
name: training-run-factual-analysis
description: Inspect one completed DRL-path-finding run for neutral factual analysis centered on train-side monitoring. Verify reproducible launch first. Treat final_probe as supplemental only. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## 1. Contract

| Label | Contract |
| --- | --- |
| Input | `source_run_dir`; `output_report_path`; `destination_directory`; `run_name` |
| Output | `neutral_factual_extraction`; `missingness_parseability`; `artifact_metadata`; `configuration_runtime_context`; `schema_defined_json`; `validation_record` |
| Evidence | `reproducible_launch_first`; `train_side_monitoring_primary`; `posthoc_checkpoint_selection_context`; `final_probe_supplemental`; `configuration_runtime_context` |
| Limits | `final_probe_role=supplemental_only`; `held_out_sample_limit=preserved`; `deterministic_reproducibility_limit=finite_sample_limit_preserved`; `train_final_consistency=factual_when_provided`; `metric_movement=factual_not_causal` |
| Input blocker | `blocked_insufficient_input` for missing/inaccessible/non-directory/ambiguous `source_run_dir` |

## 2. Extraction

Order: `train_side_monitoring` -> `posthoc_selection` -> `supplemental_final_probe` -> `configuration_runtime`.

### 2.1 reproducible_launch

| Action | Fields |
| --- | --- |
| Inspect | `logs/reproducibility_contract.json`; `logs/config_snapshot.json`; `logs/benchmark_summary.json`; `logs/artifact_index.json`; launch command records; stdout summaries; captured argv records |
| Extract | `contract_verdict`; `strict_reproducibility`; `deterministic_algorithms_enabled`; `deterministic_algorithms_warn_only`; `PYTHONHASHSEED`; `CUBLAS_WORKSPACE_CONFIG`; backend requested flags: TF32, cuDNN benchmark, AMP, inference AMP, `torch.compile`, channels-last; backend runtime readbacks; fixed train episode seed settings; fixed final-probe seed settings |
| Report | missing/malformed/incomplete/contradictory artifacts; neutral reproducible-launch status |
| Status | `reproducible_launch_confirmed`; `reproducible_launch_not_confirmed`; `reproducible_launch_contradictory`; `reproducibility_artifacts_missing`; `reproducibility_unverified` |
| Rule | verify artifacts/launch records; do not infer from run name |

### 2.2 train_side_monitoring

| Action | Fields |
| --- | --- |
| Inspect | `logs/train_steps.csv`; `logs/train_episodes.csv`; `logs/metric_snapshot.json` |
| Extract | reward; coverage; success_rate; episode_length; repeat_visit_ratio / RVR; timeout; stall; zero_info; recent_revisit; turns; semantic monitoring fields; value truncation / cap-hit diagnostics; learner / replay / exploration fields; reward breakdown; reward events |
| Semantic fields | `accessible_block_count`; `total_accessible_unknown_area`; `total_frontier_cluster_count`; `mean_block_area`; `local_frontier_coverage`; `local_frontier_block_area_mean` |
| Learner fields | `loss`; `q_mean`; `target_q_mean`; `td_abs_mean`; `grad_norm`; `replay_size`; `epsilon` |
| Report | row_counts; parseability; headers; missing_expected_columns; first_last_values_when_safe; min_max_best_recent_window_when_clear; late_stage_direction_or_slope_when_safe; empty_NaN_nonfinite_missing_unparseable_not_inspected |

### 2.3 posthoc_selection

| Action | Fields |
| --- | --- |
| Inspect | `logs/posthoc_candidate_scores.csv`; `logs/posthoc_selection_summary.json`; `logs/formal_selection_manifest.json`; `logs/best_vs_last_gap_summary.json`; checkpoint metadata only |
| Extract | `protocol_name`; `candidate_start_step`; `candidate_end_step`; `checkpoint_interval`; `selection_window_env_steps`; `selection_weights`; `candidate_count`; `valid_candidate_count`; `selected_candidate_steps`; `winner_step`; `best.pt` / `last.pt` metadata; selected candidate checkpoint metadata; winner-vs-last facts; best-vs-last facts; whether `last.pt` received `final_probe` row; candidate score row count; parseability |
| Report | `checkpoint_selection_facts_only` |

### 2.4 supplemental_final_probe

| Action | Fields |
| --- | --- |
| Inspect | `logs/final_probe.csv`; `logs/final_probe_summary.json`; `metric_snapshot` `final_probe` fields |
| Extract | final-probe episode_count; seed base; winner row; candidate rows; reward; coverage; success_rate; episode_length; repeat_visit_ratio / RVR; timeout; stall; zero-info; recent-revisit; turn diagnostics; ranking order; row count; parseability; missing expected fields |
| Report | supplemental held-out outcome after train-side and post-hoc facts |

### 2.5 configuration_runtime

| Action | Fields |
| --- | --- |
| Inspect | `logs/config_snapshot.json`; `logs/benchmark_summary.json`; `logs/artifact_index.json` |
| Extract | budget fields; exploration/replay/learner fields; reward config; map/eval config; post-hoc candidate and selection fields; runtime duration; device; run mode; performance switches; artifact inventory; git branch and commit metadata; observed run contract fields |
| Budget fields | `total_env_steps`; `budget_mode`; `total_train_episodes` |
| Exploration/replay/learner | `epsilon_decay_steps`; `epsilon_end`; `min_replay_size`; `batch_size` |
| Reward config | `reward_info_scale`; `reward_revisit_penalty`; `reward_turn_penalty_scale` |
| Map/eval config | `scan_radius`; `rows`; `cols`; `final_greedy_episodes` |

### 2.6 missingness_parseability

| Action | Fields |
| --- | --- |
| Report | files found; files missing; parse failures; malformed JSON; missing required columns; empty CSVs; unverified items; artifacts not inspected due to scope; factual summary sufficiency |
| Status | `factual_summary_ready`; `partial_factual_summary`; `missing_core_training_monitoring`; `reproducibility_unverified`; `blocked_insufficient_input`; `parse_failed` |

## 3. JSON

Authorized write: JSON only, with this object:

```json
{
  "schema_version": "1.0",
  "report_type": "training_run_factual_analysis",
  "generated_by": "codex",
  "source_run_dir": "<sanitized or relative>",
  "run_name": "<name or null>",
  "files_inspected": [],
  "commands_run": [],
  "reproducible_launch_status": "<status>",
  "reproducible_launch": {},
  "train_side_monitoring": {},
  "posthoc_selection": {},
  "supplemental_final_probe": {},
  "configuration_and_runtime": {},
  "missing_artifacts": [],
  "parse_failures": [],
  "unverified_items": [],
  "forbidden_artifact_findings": [],
  "factual_summary_status": "<status>",
  "tuning_recommendation_provided": false
}
```

Validate:
- `json_parse`
- `required_top_level_keys`
- constants: `schema_version="1.0"`; `report_type="training_run_factual_analysis"`; `generated_by="codex"`; `tuning_recommendation_provided=false`
- `factual_summary_status_neutral`
- `path_sanitization`
- `excludes_full_csv_checkpoint_payloads_weights_private_paths_tuning_decisions`

## 4. Validation

Checklist:
- `commands_run_recorded`
- `files_inspected_recorded`
- `json_parse`
- `top_level_key_completeness`
- `report_type`
- `generated_by`
- `factual_summary_status`
- `tuning_recommendation_provided_false`
- `path_sanitization`
- `write_scope`
- `forbidden_artifact_status`
- `no_tuning_decision_status`

## 5. Blockers

Block:
- `write_scope_violation`: only the authorized report path may be written.
- `source_integrity_violation`: training code and source run outputs are read-only.
- `artifact_copy_violation`: checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, and binary artifacts are metadata-only.
- `path_privacy_violation`: tracked outputs use sanitized paths, repository-relative paths, or stable labels.
- `decision_scope_violation`: factual-only output; excludes tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, paper-level conclusions, routing rules, project-baseline instructions, and workflow decisions.
- `evidence_role_violation`: `final_probe` is supplemental only; deterministic reproducibility keeps finite-sample limits.
