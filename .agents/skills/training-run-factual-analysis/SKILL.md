---
name: training-run-factual-analysis
description: Inspect one completed DRL-path-finding run for compact neutral factual JSON centered on train-side monitoring. Verify reproducible launch first. Treat posthoc/final_probe as conditional legacy or eval-enabled artifacts. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## 1. Contract

| Label | Contract |
| --- | --- |
| Input | `source_run_dir`; `output_report_path`; `destination_directory`; `run_name` |
| Output | compact `training_run_factual_analysis` JSON for GPT tuning review |
| Primary evidence | `reproducible_launch`; `train_side_monitoring`; `configuration_runtime` |
| Conditional evidence | `posthoc_selection` only when enabled and present; `supplemental_final_probe` only when enabled and present |
| Evidence role | train-side facts guide objective tuning review; derived metrics explain mechanism, not superiority |
| Boundary | factual-only; `tuning_recommendation_provided=false`; no tuning decisions, next hyperparameters, acceptance decisions, stop/continue decisions, or branch decisions |
| Density | review-facing compact digest; source artifacts remain full evidence |
| Input blocker | `blocked_insufficient_input` for missing, inaccessible, non-directory, or ambiguous `source_run_dir` |

Train-side-only rule:
- If `train_side_only_tuning=true`, posthoc/final_probe absence is intentional.
- Record `status=skipped_train_side_only_tuning`; `role=not_applicable_for_train_side_tuning`; `missingness_treatment=not_missing`.
- Do not synthesize `winner_step`, `selected_candidate_steps`, checkpoint winners, candidate rankings, final_probe summaries, best-vs-last summaries, or test-side metrics.
- Skipped posthoc/final_probe must not lower `factual_summary_status` when reproducibility and core train-side monitoring are ready.

## 2. Mode Detection

Inspect compact artifacts:

| Artifact | Fields |
| --- | --- |
| `logs/config_snapshot.json` | `train_side_only_tuning`; eval/posthoc/final_probe config; runtime/config fields; observed run contract fields |
| `logs/metric_snapshot.json` | train endpoint summaries; optional final_probe summary fields; skipped/eval status fields |
| `logs/artifact_index.json` | artifact presence; formal final object role; enabled/present/skipped/missing/malformed status |
| `logs/reproducibility_contract.json` | strict backend status; seed status; reproducibility verdict |
| launch/stdout/argv summaries | command identity; source git metadata when available; observed run contract fields |

Report:

| Field | Values |
| --- | --- |
| `train_side_only_tuning` | `true`; `false`; `unknown` |
| `test_side_evaluation_status` | `enabled`; `disabled`; `skipped_train_side_only_tuning`; `missing`; `malformed`; `unknown` |
| `skipped_test_side_evaluation.reason` | compact source value or `train_side_only_tuning` |
| `formal_final_object_role` | compact role when present |
| `posthoc_selection.status` | `present`; `enabled_missing`; `skipped_train_side_only_tuning`; `not_applicable_for_train_side_tuning`; `malformed`; `unknown` |
| `supplemental_final_probe.status` | `present`; `enabled_missing`; `skipped_train_side_only_tuning`; `not_applicable_for_train_side_tuning`; `malformed`; `unknown` |
| `missingness_treatment` | `not_missing` for train-side-only skips; otherwise factual missingness state |

## 3. Extraction

Order:
1. `reproducible_launch`
2. `train_side_mode_detection`
3. `train_side_monitoring`
4. `configuration_runtime`
5. `optional_test_side_artifacts`
6. `missingness_parseability`

### 3.1 reproducible_launch

| Action | Fields |
| --- | --- |
| Inspect | `logs/reproducibility_contract.json`; `logs/config_snapshot.json`; `logs/benchmark_summary.json`; `logs/artifact_index.json`; launch command records; stdout summaries; captured argv records |
| Extract | `contract_verdict`; `strict_reproducibility`; deterministic algorithms; `PYTHONHASHSEED`; `CUBLAS_WORKSPACE_CONFIG`; backend requested flags; backend runtime readbacks; train seed status; eval seed status when enabled; source branch/commit when available |
| Status | `reproducible_launch_confirmed`; `reproducible_launch_not_confirmed`; `reproducible_launch_contradictory`; `reproducibility_artifacts_missing`; `reproducibility_unverified` |
| Rule | verify artifacts/launch records; do not infer from run name |

### 3.2 train_side_monitoring

Inspect: `logs/train_steps.csv`; `logs/train_episodes.csv`; `logs/metric_snapshot.json`.

| Group | Compact fields |
| --- | --- |
| Row counts | train step rows; train episode rows; empty CSV status; parsed row counts |
| Parseability | readable files; malformed files; missing expected columns; NaN/nonfinite handling |
| Endpoint metrics | `reward`; `coverage`; `success_rate`; `episode_length`; `repeat_visit_ratio` / `RVR`; `timeout` |
| Deltas | initial-to-endpoint delta; late-stage delta or direction when safe |
| Key behavior diagnostics | `zero_info`; `recent_revisit`; `stall`; `turn_ge_90`; `turn_135`; `turn_180`; `timeout_flag`; `turn_penalty_weight_sum` when present |
| Derived exploration-efficiency diagnostics | `coverage_gain_per_step`; `weighted_info_gain_per_step`; `zero_info_rate`; `recent_revisit_rate`; `stall_rate`; `turn_burden_rate`; `timeout_rate` |
| Derived metric role | `diagnostic_only`; `not_standalone_superiority_evidence`; used to explain mechanism, not decide superiority alone |
| Reward breakdown | `info_reward_sum`; `step_penalty_sum`; `recent_revisit_penalty_sum`; `turn_penalty_sum`; `timeout_penalty_sum`; `terminal_bonus_sum` |
| Reward event summary | `delta_empty_sum`; `delta_obstacle_sum`; `empty_info_gain_sum`; `obstacle_info_gain_sum`; `weighted_obstacle_info_gain_sum`; `weighted_info_gain_sum`; `obstacle_info_contribution_ratio`; `recent_revisit_trigger_count`; `stall_trigger_count`; `zero_info_step_count`; `turn_ge_90_count`; `turn_135_count`; `turn_180_count`; `timeout_flag` |
| Learner/replay/exploration | `loss`; `q_mean`; `target_q_mean`; `td_abs_mean`; `grad_norm`; `replay_size`; `epsilon`; `batch_size`; `replay_capacity`; `n_step`; `gamma`; `learning_rate`; `target_update_interval`; `collect_steps_per_iter`; `learner_updates_per_iter`; `train_every_env_steps` |

### 3.3 configuration_runtime

| Group | Compact fields |
| --- | --- |
| Budget | `total_env_steps`; `budget_mode`; `total_train_episodes`; `final_greedy_episodes` |
| Exploration | `epsilon_decay_steps`; `epsilon_end`; `min_replay_size` |
| Replay/learner/update | `batch_size`; `replay_capacity`; `n_step`; `gamma`; `learning_rate`; `target_update_interval`; `collect_steps_per_iter`; `learner_updates_per_iter`; `train_every_env_steps` |
| Reward config | reward config values needed to interpret reward breakdown |
| Comparability | map/eval config fields needed for compact comparison |
| Mode | `train_side_only_tuning`; `test_side_evaluation_status`; skipped reason |
| Runtime | device; runtime duration; source git branch/commit; strict backend status; seed status |

### 3.4 optional_test_side_artifacts

For `train_side_only_tuning=true`:

| Object | Required compact fields |
| --- | --- |
| `posthoc_selection` | `status=skipped_train_side_only_tuning`; `role=not_applicable_for_train_side_tuning`; `missingness_treatment=not_missing`; no `winner_step`; no `selected_candidate_steps` |
| `supplemental_final_probe` | `status=skipped_train_side_only_tuning`; `role=not_applicable_for_train_side_tuning`; `missingness_treatment=not_missing`; no ranking; no best-vs-last; no synthetic test-side facts |

For legacy/eval-enabled runs:

| Artifact | Compact extraction |
| --- | --- |
| `posthoc_selection` | enabled/present status; checkpoint-selection context; candidate counts; selected candidate metadata; `winner_step` only when factual and present; parseability |
| `supplemental_final_probe` | enabled/present status; supplemental held-out metrics; row count; compact ranking only when factual and present; parseability |
| Rule | conditional/supporting evidence only; never primary evidence |

### 3.5 missingness_parseability

Report: files found; files missing; parse failures; malformed JSON; missing required columns; empty CSVs; unverified items; skipped train-side-only artifacts; artifacts not inspected due to scope; factual summary sufficiency.

Status enum:
- `factual_summary_ready`
- `partial_factual_summary`
- `missing_core_training_monitoring`
- `reproducibility_unverified`
- `blocked_insufficient_input`
- `parse_failed`

## 4. JSON Output

Authorized write: one compact JSON report at `output_report_path`.

Required top-level object:

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
  "train_side_monitoring": {
    "row_counts": {},
    "parseability": {},
    "endpoint_metrics": {},
    "initial_to_endpoint_delta": {},
    "late_stage_delta": {},
    "key_behavior_diagnostics": {},
    "derived_exploration_efficiency": {
      "metric_role": "diagnostic_only"
    },
    "reward_breakdown": {},
    "reward_event_summary": {},
    "learner_replay_exploration_context": {},
    "missing_expected_columns": []
  },
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

Constants:
- `schema_version="1.0"`
- `report_type="training_run_factual_analysis"`
- `generated_by="codex"`
- `tuning_recommendation_provided=false`

Include:
- gate status and blocking missingness
- identity and sanitized source path
- mode detection
- compact configuration/runtime
- endpoint train-side metrics
- train-side deltas
- key diagnostics
- derived exploration-efficiency metrics
- reward breakdown
- learner/replay/exploration context
- optional skipped/eval status for test-side artifacts
- validation facts

Exclude:
- full CSV rows or headers
- full logs or command traces
- broad numeric dumps
- full artifact inventories
- checkpoint payloads or model weights
- plots, trajectories, raw outputs, or binary artifacts
- private absolute source paths in tracked JSON
- tuning decisions or next hyperparameters
- baseline acceptance, stop/continue, or branch decisions

## 5. Validation

Checklist:
- `json_parse`
- `required_top_level_keys`
- `constants_preserved`
- `commands_run_recorded`
- `files_inspected_recorded`
- `factual_summary_status_valid`
- `train_side_only_tuning_supported`
- `skipped_posthoc_final_probe_not_missing_when_train_side_only`
- `derived_exploration_efficiency_has_seven_diagnostics`
- `derived_metrics_diagnostic_only`
- `path_sanitization`
- `write_scope`
- `forbidden_artifact_status`
- `no_tuning_recommendation`
- `compact_report_density`
- `no_source_artifact_dumps`
- `no_private_paths`

Factual summary rule:
- Use `factual_summary_ready` when reproducibility and core train-side monitoring are ready, even when posthoc/final_probe are skipped by `train_side_only_tuning`.

## 6. Blockers

Block:
- `write_scope_violation`: only the authorized report path may be written.
- `source_integrity_violation`: training code and source run outputs are read-only.
- `artifact_copy_violation`: checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, and binary artifacts are metadata-only.
- `path_privacy_violation`: tracked outputs use sanitized paths, repository-relative paths, or stable labels.
- `decision_scope_violation`: factual-only output; tuning, next-parameter, baseline, stop/continue, branch, method, paper, routing, project-baseline, and workflow decisions are outside this skill.
- `evidence_role_violation`: posthoc/final_probe are conditional only; final_probe is supplemental when enabled.
- `train_side_only_missingness_violation`: skipped posthoc/final_probe under `train_side_only_tuning` are recorded as `not_missing`.
- `output_density_violation`: JSON includes source-artifact dumps instead of compact tuning-review summaries.
