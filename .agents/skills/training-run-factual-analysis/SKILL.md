---
name: training-run-factual-analysis
description: Inspect a completed DRL-path-finding training run and produce neutral factual analysis centered on train-side monitoring. Verify reproducible-launch evidence first. Treat final_probe as supplemental validation only. Never provide tuning recommendations or decisions.
---

# Training Run Factual Analysis

## 1. Skill Contract

Use this skill to inspect one completed `DRL-path-finding` run and write a neutral factual JSON report only when the prompt authorizes that write.

Authority:
- Source training artifacts own run facts.
- Codex extracts, summarizes, validates, and records facts.
- Reproducible-launch evidence is checked before metrics.
- Train-side monitoring is primary evidence.
- Post-hoc selection is checkpoint-selection context.
- `final_probe` is supplemental held-out validation.
- Configuration and runtime facts are context.

## 2. Inputs

Expected fields:
- `source_run_dir`: one completed source run directory.
- `output_report_path`: optional report path when writing is authorized.
- `destination_directory`: optional report directory when writing is authorized.
- `run_name`: optional run label from the prompt or `source_run_dir`.

Input blocker:
- Missing, inaccessible, non-directory, or ambiguous `source_run_dir` yields `blocked_insufficient_input`; invent no facts.

## 3. Output Allowlist

Output is limited to:
- neutral factual extraction
- missingness and parseability
- artifact existence and metadata
- configuration and runtime context
- structured JSON fields defined by the schema
- validation contract fields

Evidence roles:
- train-side monitoring: primary
- post-hoc selection: checkpoint-selection context
- `final_probe`: supplemental held-out validation
- configuration/runtime: context

## 4. Extraction Contract

Use evidence order:
1. Train-side monitoring.
2. Post-hoc checkpoint-selection facts.
3. Supplemental `final_probe` / held-out validation facts.
4. Runtime and configuration context.

Cross-cutting limits:
- `final_probe` is not standalone superiority evidence.
- Finite held-out episode limitations remain.
- Deterministic reproducibility does not remove finite-evaluation-sample limitations.
- Train-final consistency is factual only when source artifacts provide it.
- Metric movement is factual, not causal.

### 4.1 Reproducible Launch

Inspect:
- `logs/reproducibility_contract.json`
- `logs/config_snapshot.json`
- `logs/benchmark_summary.json`
- `logs/artifact_index.json`
- launch command records, stdout summaries, captured argv records

Extract:
- `contract_verdict`
- `strict_reproducibility`
- `deterministic_algorithms_enabled`
- `deterministic_algorithms_warn_only`
- `PYTHONHASHSEED`
- `CUBLAS_WORKSPACE_CONFIG`
- backend requested flags: TF32, cuDNN benchmark, AMP, inference AMP, `torch.compile`, channels-last
- backend runtime readbacks
- fixed train episode seed settings: `use_fixed_train_episode_seeds`, `fixed_train_episode_seed_base`
- fixed final-probe seed settings: `use_fixed_eval_seeds`, `fixed_final_probe_seed_base`

Report:
- missing, malformed, incomplete, or contradictory reproducibility artifacts
- neutral reproducible-launch status

Statuses:
- `reproducible_launch_confirmed`
- `reproducible_launch_not_confirmed`
- `reproducible_launch_contradictory`
- `reproducibility_artifacts_missing`
- `reproducibility_unverified`

Rule: verify actual reproducible mode from artifacts or launch records; do not infer reproducible mode from run name.

### 4.2 Train-Side Monitoring

Inspect:
- `logs/train_steps.csv`
- `logs/train_episodes.csv`
- `logs/metric_snapshot.json`

Extract:
- reward, coverage, success_rate, episode_length, repeat_visit_ratio / RVR
- timeout, stall, zero_info, recent_revisit, turns
- semantic monitoring: `accessible_block_count`, `total_accessible_unknown_area`, `total_frontier_cluster_count`, `mean_block_area`, `local_frontier_coverage`, `local_frontier_block_area_mean`
- value truncation / cap-hit diagnostics
- learner / replay / exploration: `loss`, `q_mean`, `target_q_mean`, `td_abs_mean`, `grad_norm`, `replay_size`, `epsilon`
- reward breakdown and reward events

Report:
- row counts and parseability
- headers and missing expected columns
- first/last values when safe
- min/max/best/recent-window values when field semantics are clear
- late-stage direction or slope when source artifact provides it or simple computation is safe
- empty, NaN, non-finite, missing, unparseable, or not-inspected fields

### 4.3 Post-Hoc Selection

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
- `best.pt` / `last.pt` metadata
- selected candidate checkpoint metadata
- winner-vs-last or best-vs-last facts
- whether `last.pt` received a `final_probe` row
- candidate score row count and parseability

Report: checkpoint-selection facts only.

### 4.4 Supplemental Final Probe

Inspect:
- `logs/final_probe.csv`
- `logs/final_probe_summary.json`
- `metric_snapshot` `final_probe` fields

Extract:
- final-probe episode count
- seed base
- winner row and candidate rows
- reward, coverage, success_rate, episode_length, repeat_visit_ratio / RVR
- timeout, stall, zero-info, recent-revisit, turn diagnostics
- ranking order
- row count, parseability, missing expected fields

Report: supplemental held-out outcome after train-side and post-hoc facts.

### 4.5 Configuration And Runtime

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
- runtime duration, device, run mode, performance switches
- artifact inventory
- git branch and commit metadata when present
- observed run contract fields when present

### 4.6 Missingness And Parseability

Report:
- files found and files missing
- parse failures and malformed JSON
- missing required columns
- empty CSVs
- unverified items
- artifacts not inspected due to scope
- factual summary sufficiency

Statuses:
- `factual_summary_ready`
- `partial_factual_summary`
- `missing_core_training_monitoring`
- `reproducibility_unverified`
- `blocked_insufficient_input`
- `parse_failed`

## 5. JSON Contract

When writing is authorized, write JSON only with this top-level object:

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

Required constants:
- `schema_version`: `"1.0"`
- `report_type`: `"training_run_factual_analysis"`
- `generated_by`: `"codex"`
- `tuning_recommendation_provided`: `false`

Validate:
- JSON parse success and required top-level key completeness.
- `factual_summary_status` is present and uses a neutral status from Section 4.6.
- Paths are sanitized.
- JSON excludes full CSV contents, checkpoint payloads, model weights, private absolute paths, and tuning decisions.

## 6. Validation Contract

Record and validate:
- `commands_run`
- `files_inspected`
- JSON parse result
- top-level key completeness
- `report_type`
- `generated_by`
- `factual_summary_status`
- `tuning_recommendation_provided=false`
- path sanitization
- write scope
- forbidden-artifact status
- no-tuning-decision status

Keep validation factual.

## 7. Hard Blockers

- Do not modify training code.
- Do not modify source run outputs.
- Do not copy checkpoints, model weights, full logs, full CSV contents, raw outputs, plots, trajectories, or binary artifacts.
- Do not store private absolute paths unless sanitized and necessary.
- Do not provide tuning recommendations.
- Do not provide next hyperparameters.
- Do not provide accepted-baseline decisions.
- Do not provide stop/continue decisions or branch decisions.
- Do not provide method-level conclusions or paper-level conclusions.
- Do not claim `final_probe` alone proves superiority.
- Do not claim deterministic reproducibility removes finite-sample limitations.
- Do not output routing rules, project-baseline instructions, or cross-file workflow orchestration.
