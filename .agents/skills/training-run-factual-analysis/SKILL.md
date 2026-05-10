---
name: training-run-factual-analysis
description: Inspect a completed DRL-path-finding training run and produce neutral factual analysis centered on train-side monitoring. Verify reproducible-launch evidence first. Treat final_probe as supplemental validation only. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## 1. Purpose

This skill defines a neutral Codex procedure for inspecting one completed `DRL-path-finding` training run and producing a factual run-analysis summary. The procedure focuses on source-side run artifacts, train-side monitoring, post-hoc checkpoint-selection facts, supplemental held-out validation outcomes, configuration facts, runtime facts, missingness, and parseability.

This skill is not a tuning guide, decision policy, router, project baseline, workflow document, accepted-baseline policy, or method-review authority. Codex reports facts only.

## 2. Authority Boundary

- Source training artifacts own run facts.
- Codex extracts and summarizes facts only.
- Codex must not provide tuning recommendations, accepted-baseline decisions, stop/continue decisions, branch decisions, next hyperparameters, method-level conclusions, or paper-level conclusions.
- This skill does not modify training code or run outputs.

## 3. Required Input

Expected task input:

- `source_run_dir`: path to one completed training run directory.
- `output_report_path`: optional JSON report path, only when the prompt authorizes writing a report.
- `destination_directory`: optional report destination for the structured JSON report, only when the prompt authorizes writing.
- `run_name`: optional run label supplied by the prompt or inferred from the run directory name.

If `source_run_dir` is missing, inaccessible, not a directory, or too ambiguous to identify one run, report `blocked_insufficient_input` and do not invent facts.

## 4. Reproducible-Launch Verification First

Before summarizing metrics, inspect source-side reproducibility and launch artifacts when present:

- `logs/reproducibility_contract.json`
- `logs/config_snapshot.json`
- `logs/benchmark_summary.json`
- `logs/artifact_index.json`
- launch command records, stdout summaries, or captured argv records when present

Extract these fields when present:

- `contract_verdict`
- `strict_reproducibility`
- `deterministic_algorithms_enabled`
- `deterministic_algorithms_warn_only`
- `PYTHONHASHSEED`
- `CUBLAS_WORKSPACE_CONFIG`
- backend requested flags for TF32, cuDNN benchmark, AMP, inference AMP, `torch.compile`, and channels-last
- backend runtime readbacks for TF32, cuDNN benchmark, deterministic algorithms, AMP-related settings when recorded, `torch.compile`, and channels-last
- fixed train episode seed settings:
  - `use_fixed_train_episode_seeds`
  - `fixed_train_episode_seed_base`
- fixed final-probe seed settings:
  - `use_fixed_eval_seeds`
  - `fixed_final_probe_seed_base`

Required interpretation constraints:

- Do not infer reproducible mode from the run name.
- The training code supports reproducible mode, but normal `TrainConfig` defaults are not strict reproducible mode.
- Actual reproducible mode must be verified from run artifacts or launch records.
- If reproducibility artifacts are missing, malformed, incomplete, or contradictory, report the missingness or contradiction and keep the analysis factual.
- Do not make formal accept/reject decisions.

Use neutral reproducible-launch statuses:

- `reproducible_launch_confirmed`
- `reproducible_launch_not_confirmed`
- `reproducible_launch_contradictory`
- `reproducibility_artifacts_missing`
- `reproducibility_unverified`

## 5. Evidence Priority

Use this evidence priority:

1. Train-side monitoring data as primary factual evidence.
2. Post-hoc selection facts as checkpoint-selection context.
3. `final_probe` / held-out validation data as supplemental validation evidence.
4. Runtime and configuration facts as contextual evidence.

Required limits:

- `final_probe` is not standalone superiority evidence.
- Small held-out episode counts should be reported as a limitation when applicable.
- Deterministic reproducibility does not remove finite-evaluation-sample limitations.
- Train-final consistency should be summarized factually when source artifacts provide it.
- Do not infer causality from metric movement.

## 6. Train-Side Monitoring Summary

Inspect and summarize these artifacts when present:

- `logs/train_steps.csv`
- `logs/train_episodes.csv`
- `logs/metric_snapshot.json`

Train-side monitoring is the primary factual summary target.

Required train-side categories:

- reward dynamics
- coverage dynamics
- success-rate dynamics
- episode-length dynamics
- repeat-visit-ratio / RVR dynamics
- timeout indicators
- stall diagnostics
- zero-info diagnostics
- recent revisit events
- turn diagnostics
- semantic monitoring:
  - `accessible_block_count`
  - `total_accessible_unknown_area`
  - `total_frontier_cluster_count`
  - `mean_block_area`
  - `local_frontier_coverage`
  - `local_frontier_block_area_mean`
  - value truncation / cap-hit diagnostics
- learner / replay / exploration:
  - `loss`
  - `q_mean`
  - `target_q_mean`
  - `td_abs_mean`
  - `grad_norm`
  - `replay_size`
  - `epsilon`
- reward breakdown and reward events when present

Use factual summaries such as:

- row counts
- parseability
- headers and missing expected columns
- first values when safely computable
- last values when safely computable
- best or maximum/minimum values when the field semantics are clear
- recent-window values when directly present or safely computable
- late-stage direction or slope when source artifacts provide it or when a simple computation is explicitly safe
- obvious empty, NaN, or non-finite issues
- explicit notes that a field is missing, unparseable, or not inspected

Do not provide causal interpretation beyond facts. Do not convert train-side metrics into tuning recommendations.

## 7. Post-Hoc Selection Summary

Inspect these artifacts when present:

- `logs/posthoc_candidate_scores.csv`
- `logs/posthoc_selection_summary.json`
- `logs/formal_selection_manifest.json`
- `logs/best_vs_last_gap_summary.json`
- checkpoint metadata only; never copy checkpoint binaries

Extract factual items:

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
- `checkpoints/best.pt` path as metadata only
- `checkpoints/last.pt` path as metadata only
- winner-vs-last or best-vs-last diagnostic facts
- whether `last.pt` received a held-out `final_probe` row
- candidate score CSV row count and parseability
- selected candidate checkpoint paths as metadata only

Do not say whether the winner is a good tuning outcome. Report only how it was selected and what source artifacts state.

## 8. Supplemental Final-Probe Summary

Inspect these artifacts when present:

- `logs/final_probe.csv`
- `logs/final_probe_summary.json`
- `final_probe` fields embedded in `logs/metric_snapshot.json`

Extract factual items:

- final-probe episode count
- seed base
- winner row
- candidate rows
- reward
- coverage
- success rate
- episode length
- repeat-visit-ratio / RVR
- timeout, stall, zero-info, and revisit diagnostics when present
- ranking order
- row count, parseability, and missing expected fields

Required limits:

- `final_probe` is supplemental held-out outcome evidence.
- Summarize `final_probe` after train-side and post-hoc facts.
- Do not treat `final_probe` as the sole basis for run quality.
- Do not claim that `final_probe` alone proves superiority.

## 9. Configuration And Runtime Facts

Inspect these artifacts when present:

- `logs/config_snapshot.json`
- `logs/benchmark_summary.json`
- `logs/artifact_index.json`

Extract factual items:

- `total_env_steps`
- `budget_mode`
- `total_train_episodes`
- `epsilon_decay_steps`
- `epsilon_end`
- `min_replay_size`
- `batch_size`
- `reward_info_scale`
- `reward_revisit_penalty`
- `reward_turn_penalty_scale`
- `scan_radius`
- `rows`
- `cols`
- `final_greedy_episodes`
- post-hoc candidate and selection fields
- runtime duration
- device
- run mode
- performance switches
- artifact inventory
- git branch and commit metadata when present
- observed run contract fields when present

Treat configuration and runtime facts as context. Do not transform them into next-parameter selection logic.

## 10. Missingness And Parseability

Report:

- files found
- files missing
- parse failures
- missing required columns
- empty CSVs
- malformed JSON
- unverified items
- artifacts not inspected due to scope
- whether outputs are sufficient for a factual summary

Use neutral evidence completeness statuses:

- `factual_summary_ready`
- `partial_factual_summary`
- `missing_core_training_monitoring`
- `reproducibility_unverified`
- `blocked_insufficient_input`
- `parse_failed`

Do not use statuses that imply decision acceptance.

## 11. Forbidden Artifact Handling

Codex must never copy or publish:

- checkpoints
- model weights
- full logs
- full CSV contents
- raw output directories
- plots
- trajectories
- private absolute paths unless sanitized and necessary

Codex may report existence, relative paths, file sizes, timestamps, and metadata of checkpoints when needed. Metadata reporting must not include binary payload contents.

## 12. Structured JSON Output

When the prompt authorizes writing a report, use this structured JSON object shape:

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

Output requirements:

- The output must be valid JSON.
- JSON parse success alone is insufficient for report validation.
- Every required top-level key in the structured JSON object shape must be present before commit/push.
- The report must not omit `factual_summary_status`.
- `factual_summary_status` must be set to the most appropriate neutral status from this skill:
  - `factual_summary_ready`
  - `partial_factual_summary`
  - `missing_core_training_monitoring`
  - `reproducibility_unverified`
  - `blocked_insufficient_input`
  - `parse_failed`
- `report_type` must be exactly `training_run_factual_analysis`.
- `generated_by` must be exactly `codex`.
- `tuning_recommendation_provided` must always be `false`.
- `tuning_recommendation_provided` must be verified as exactly `false`.
- Paths must be sanitized.
- Do not include full CSV contents.
- Do not include checkpoint payloads.
- Do not include model weights.
- Do not include private absolute paths.
- Do not include tuning decisions.

Required top-level keys:

- `schema_version`
- `report_type`
- `generated_by`
- `source_run_dir`
- `run_name`
- `files_inspected`
- `commands_run`
- `reproducible_launch_status`
- `reproducible_launch`
- `train_side_monitoring`
- `posthoc_selection`
- `supplemental_final_probe`
- `configuration_and_runtime`
- `missing_artifacts`
- `parse_failures`
- `unverified_items`
- `forbidden_artifact_findings`
- `factual_summary_status`
- `tuning_recommendation_provided`

## 13. Prohibited Output

Do not output:

- tuning recommendations
- next hyperparameters
- accepted-baseline decisions
- stop/continue/branch decisions
- method-level conclusions
- paper-level conclusions
- claims that `final_probe` alone proves superiority
- claims that deterministic reproducibility eliminates finite-evaluation-sample limitations
- modifications of training code
- modifications of run outputs
- routing rules
- project-baseline instructions
- cross-file workflow orchestration requirements

## 14. Validation For Skill Use

When this skill is used in a real task, record:

- commands run
- files inspected
- JSON parse check
- schema completeness check for all required top-level keys
- final `factual_summary_status` value
- confirmation that `tuning_recommendation_provided` is false
- whether the training repository was modified
- whether the generated JSON report file was written
- whether forbidden artifacts were copied
- whether no tuning recommendation was provided

The validation record must remain factual and must not include a tuning decision.
