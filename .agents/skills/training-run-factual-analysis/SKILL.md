---
name: training-run-factual-analysis
description: Inspect a completed DRL-path-finding training run and produce neutral factual analysis centered on train-side monitoring. Verify reproducible-launch evidence first. Treat final_probe as supplemental validation only. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## 1. Skill Contract

Use this skill to inspect one completed `DRL-path-finding` training run and produce a neutral factual JSON report when the prompt authorizes a report write.

Authority:
- Source training artifacts own run facts.
- Codex extracts and summarizes facts only.
- Train-side monitoring is the primary evidence surface.
- Post-hoc selection is checkpoint-selection context.
- `final_probe` is supplemental held-out validation evidence.
- Configuration and runtime facts are context.

Boundaries:
- Do not modify training code or source run outputs.
- Do not copy checkpoints, model weights, full logs, full CSV contents, raw output directories, plots, trajectories, or binary artifacts.
- Do not provide tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, or paper-level conclusions.

## 2. Inputs

Expected task fields:
- `source_run_dir`: one completed training run directory.
- `output_report_path`: optional JSON output path when writing is authorized.
- `destination_directory`: optional report destination when writing is authorized.
- `run_name`: optional run label supplied by the prompt or inferred from `source_run_dir`.

Blocker:
- If `source_run_dir` is missing, inaccessible, not a directory, or too ambiguous to identify one run, report `blocked_insufficient_input` and do not invent facts.

## 3. Evidence Order

Use evidence in this order:
1. Train-side monitoring as primary factual evidence.
2. Post-hoc selection as checkpoint-selection context.
3. `final_probe` / held-out validation as supplemental validation evidence.
4. Runtime and configuration as context.

Limits:
- `final_probe` is not standalone superiority evidence.
- Finite held-out episode limitations remain, including small held-out episode counts when applicable.
- Deterministic reproducibility does not remove finite-evaluation-sample limitations.
- Train-final consistency is factual only when source artifacts provide it.
- Do not infer causality from metric movement.

## 4. Reproducible Launch

Verify reproducible-launch evidence before summarizing metrics. Do not infer reproducible mode from the run name.

Inspect when present:
- `logs/reproducibility_contract.json`
- `logs/config_snapshot.json`
- `logs/benchmark_summary.json`
- `logs/artifact_index.json`
- launch command records, stdout summaries, or captured argv records

Extract when present:
- `contract_verdict`
- `strict_reproducibility`
- `deterministic_algorithms_enabled`
- `deterministic_algorithms_warn_only`
- `PYTHONHASHSEED`
- `CUBLAS_WORKSPACE_CONFIG`
- backend requested flags for TF32, cuDNN benchmark, AMP, inference AMP, `torch.compile`, and channels-last
- backend runtime readbacks for TF32, cuDNN benchmark, deterministic algorithms, AMP-related settings, `torch.compile`, and channels-last
- fixed train episode seed settings: `use_fixed_train_episode_seeds`, `fixed_train_episode_seed_base`
- fixed final-probe seed settings: `use_fixed_eval_seeds`, `fixed_final_probe_seed_base`

Report missing, malformed, incomplete, or contradictory reproducibility artifacts factually.

Use neutral reproducible-launch statuses:
- `reproducible_launch_confirmed`
- `reproducible_launch_not_confirmed`
- `reproducible_launch_contradictory`
- `reproducibility_artifacts_missing`
- `reproducibility_unverified`

## 5. Train-Side Monitoring

Inspect:
- `logs/train_steps.csv`
- `logs/train_episodes.csv`
- `logs/metric_snapshot.json`

Report for CSV and JSON artifacts:
- row counts
- parseability
- headers and missing expected columns
- first and last values when safely computable
- minimum, maximum, best, or recent-window values when field semantics are clear
- late-stage direction or slope when the source artifact provides it or a simple computation is safe
- empty, NaN, non-finite, missing, unparseable, or not-inspected fields

Preserve train-side categories and field groups:
- reward: reward dynamics, reward breakdown, reward events
- coverage: coverage dynamics
- success_rate: success-rate dynamics
- episode_length: episode-length dynamics
- repeat_visit_ratio / RVR: RVR dynamics
- timeout: timeout indicators and timeout penalties
- stall: stall diagnostics
- zero_info: zero-info diagnostics
- recent_revisit: recent revisit events and penalties
- turns: turn diagnostics, turn penalties, turn counts
- semantic monitoring: `accessible_block_count`, `total_accessible_unknown_area`, `total_frontier_cluster_count`, `mean_block_area`, `local_frontier_coverage`, `local_frontier_block_area_mean`
- value truncation / cap hits: value total, packed, truncated, block cap, entry cap, and frontier-cluster diagnostics
- learner / replay / exploration: `loss`, `q_mean`, `target_q_mean`, `td_abs_mean`, `grad_norm`, `replay_size`, `epsilon`

Do not convert train-side metrics into tuning recommendations or causal claims.

## 6. Post-Hoc Selection

Inspect:
- `logs/posthoc_candidate_scores.csv`
- `logs/posthoc_selection_summary.json`
- `logs/formal_selection_manifest.json`
- `logs/best_vs_last_gap_summary.json`
- checkpoint metadata only

Extract:
- `protocol_name`
- `candidate_start_step`
- `candidate_end_step`
- `checkpoint_interval`
- `selection_window_env_steps`
- `selection_weights`
- `candidate_count`
- `valid_candidate_count`
- `selected_candidate_steps`
- `winner_step`
- `checkpoints/best.pt` metadata
- `checkpoints/last.pt` metadata
- selected candidate checkpoint paths as metadata
- winner-vs-last or best-vs-last diagnostic facts
- whether `last.pt` received a held-out `final_probe` row
- candidate score CSV row count and parseability

Report how source artifacts selected candidates and checkpoints. Do not state whether the winner is a tuning success.

## 7. Supplemental Final Probe

Inspect:
- `logs/final_probe.csv`
- `logs/final_probe_summary.json`
- `final_probe` fields embedded in `logs/metric_snapshot.json`

Extract:
- final-probe episode count
- seed base
- winner row
- candidate rows
- reward, coverage, success_rate, episode_length, repeat_visit_ratio / RVR
- timeout, stall, zero-info, recent-revisit, and turn diagnostics when present
- ranking order
- row count, parseability, and missing expected fields

Limits:
- Summarize `final_probe` after train-side and post-hoc facts.
- Treat `final_probe` as supplemental held-out outcome evidence.
- Do not treat `final_probe` as the sole basis for run quality.
- Do not claim that `final_probe` alone proves superiority.

## 8. Configuration And Runtime

Inspect:
- `logs/config_snapshot.json`
- `logs/benchmark_summary.json`
- `logs/artifact_index.json`

Extract:
- budget: `total_env_steps`, `budget_mode`, `total_train_episodes`
- exploration/replay/learner: `epsilon_decay_steps`, `epsilon_end`, `min_replay_size`, `batch_size`
- reward config: `reward_info_scale`, `reward_revisit_penalty`, `reward_turn_penalty_scale`
- map/eval config: `scan_radius`, `rows`, `cols`, `final_greedy_episodes`
- post-hoc candidate and selection fields
- runtime: duration, device, run mode, performance switches
- artifact inventory
- git branch and commit metadata when present
- observed run contract fields when present

Treat configuration and runtime as context. Do not transform these fields into next-parameter selection logic.

## 9. Missingness And Parseability

Report:
- files found
- files missing
- parse failures
- missing required columns
- empty CSVs
- malformed JSON
- unverified items
- artifacts not inspected due to scope
- factual summary sufficiency

Use neutral factual summary statuses:
- `factual_summary_ready`
- `partial_factual_summary`
- `missing_core_training_monitoring`
- `reproducibility_unverified`
- `blocked_insufficient_input`
- `parse_failed`

Do not use statuses that imply decision acceptance.

## 10. Artifact Handling

Hard blockers:
- Do not copy or publish checkpoints, model weights, full logs, full CSV contents, raw output directories, plots, trajectories, or binary artifacts.
- Do not store private absolute paths unless sanitized and necessary.

Allowed metadata:
- Report existence, repository-relative paths or stable labels, file sizes, timestamps, and checkpoint metadata without binary payload contents.

## 11. JSON Output Contract

When writing is authorized, write valid JSON only and use this top-level object:

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

Required top-level keys are exactly those shown in the schema: `schema_version`, `report_type`, `generated_by`, `source_run_dir`, `run_name`, `files_inspected`, `commands_run`, `reproducible_launch_status`, `reproducible_launch`, `train_side_monitoring`, `posthoc_selection`, `supplemental_final_probe`, `configuration_and_runtime`, `missing_artifacts`, `parse_failures`, `unverified_items`, `forbidden_artifact_findings`, `factual_summary_status`, `tuning_recommendation_provided`.

Validation requirements:
- JSON parse success alone is insufficient.
- Every required top-level key must be present.
- `schema_version` must be exactly `"1.0"`.
- `report_type` must be exactly `"training_run_factual_analysis"`.
- `generated_by` must be exactly `"codex"`.
- `factual_summary_status` must be present and use an allowed neutral status from Section 9.
- `tuning_recommendation_provided` must be exactly `false`.
- Paths must be sanitized.
- The report must not include full CSV contents, checkpoint payloads, model weights, private absolute paths, or tuning decisions.

## 12. Prohibited Output

Do not output:
- tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, or branch decisions
- method-level conclusions or paper-level conclusions
- claims that `final_probe` alone proves superiority
- claims that deterministic reproducibility eliminates finite-evaluation-sample limitations
- training code modifications or run output modifications
- routing rules, project-baseline instructions, or cross-file workflow orchestration requirements

## 13. Validation Record

Record:
- commands run
- files inspected
- JSON parse check
- schema completeness check for all required top-level keys
- final `factual_summary_status`
- confirmation that `tuning_recommendation_provided=false`
- training repository modification status
- generated JSON written status
- forbidden artifacts copied status
- no tuning recommendation provided

Keep the validation record factual and free of tuning decisions.
