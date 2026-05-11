# GPT Tuning Review

## 1. Contract

| Field | Contract |
| --- | --- |
| Role | Interpret admissible factual evidence, validate prior rationale, synthesize history, and recommend the next bounded tuning action. |
| Inputs | Manifest, current factual report, history index, relevant archived factual reports, and relevant archived GPT tuning reviews. |
| Output | User-facing Chinese tuning review with an explicit `recommendation_type`. |
| Archive output | Produce `tuning_review_payload` only when archive/落盘 or Codex archive prompt preparation is requested. |
| Decision authority | Bounded tuning recommendation after gates pass; method redesign requires separate user approval. |

## 2. Inputs

| Input | Use |
| --- | --- |
| `docs/training_system_manifest.md` | Engineering grounding, tuning surface, artifact semantics, launcher context, legacy boundaries. |
| `training_results/current/current_training_run_analysis.json` | Current compact evidence and gate values. |
| `training_results/history/history_index.json` | Archive lookup and tuning-chain continuity. |
| Relevant archived `training_run_analysis.json` or `multi_run_analysis.json` | Prior factual evidence. |
| Relevant archived `tuning_review.json` | Prior rationale, hypothesis, recommendation, command metadata, and validation focus. |

Rules:
- Use compact factual reports and tracked history.
- Require Codex factual analysis before raw/full artifact use.
- Do not directly use full logs, full CSVs, checkpoints, model weights, raw outputs, plots, or trajectories.

## 3. Evidence Gate

| Gate | Required value | Failed behavior |
| --- | --- | --- |
| `reproducible_launch_status` | `reproducible_launch_confirmed` | Block formal next-run plan; use `requires_more_evidence` or `repeat_or_repair_run`. |
| `factual_summary_status` | `factual_summary_ready` | Block formal next-run plan; use `requires_more_evidence` or `repeat_or_repair_run`. |
| Run type | formal, complete, non-contradictory | Non-formal/debug/profile/incomplete/contradictory runs are diagnostic evidence only. |

## 4. Review Procedure

| Step | Contract |
| --- | --- |
| Active objective | Reduce the most important remaining tuning uncertainty inside the valid engineering tuning surface. |
| Next uncertainty | When gates pass, identify the next actionable uncertainty unless the user asks to pause. |
| Refuted prior | Do not treat a refuted prior hypothesis as active tuning termination; redirect to the next justified variable or comparison. |
| Engineering grounding | Use `docs/training_system_manifest.md` to constrain mainline, lifecycle, artifact semantics, config surface, reproducibility, launcher context, legacy boundaries, and tuning versus method redesign. |
| Prior match order | 1. Archived `tuning_review.json` with `recommended_next_command_summary.run_name` matching the current run. 2. Latest relevant `history_index.json` entry in the same tuning chain. 3. If absent, use `not_applicable` and state the history limitation. |
| Prior fields | `recommendation_type`; hypothesis; primary uncertainty; validation focus; supporting/refuting evidence; limitations; command metadata. |
| Prior validation | Compare current factual evidence with prior hypothesis before recommending the next action. |
| History synthesis | Record relevant entries, chain summary, repeated/resolved/remaining uncertainties, tested dimensions, and single-variable continuation or variable switch. |
| Evidence roles | Current factual report decides admissibility and current facts; prior tuning review defines experiment intent; historical records define chain context. |
| Evidence hierarchy | Train-side monitoring is primary; posthoc checkpoint-selection is context; `final_probe` is supplemental held-out validation; runtime speed is not method-performance superiority evidence. |

`prior_rationale_validation.validation_status` enum:
- `supported`
- `partially_supported`
- `refuted`
- `not_verifiable`
- `not_applicable`

Validation content:
- supporting evidence
- contradicting evidence
- unverified items
- limitations

## 5. Metric Judgement

| Rule | Contract |
| --- | --- |
| Thresholds | Use no fixed universal thresholds unless sourced from implementation, artifact schema, accepted protocol, or user instruction. |
| Context | Ground judgement in current factual analysis, prior hypothesis, manifest semantics, history, direction consistency, checkpoint behavior, best-vs-last or winner-vs-last context, train-final consistency, reproducibility, and `final_probe` limitations. |
| Reproducibility | Strict reproducibility means small metric movements are not dismissed as random training noise by default. |
| Limits | Reproducibility does not remove evaluation-sample, held-out seed-set, or recent-window limitations. |

## 6. Recommendation Contract

| `recommendation_type` | Contract |
| --- | --- |
| `next_run_plan` | Executable next formal run; normally choose when a bounded single-variable run is justified. |
| `hold_current_baseline` | Reference-baseline decision only; not global optimum, tuning completion, or proof of no further improvement. State pause reason or open uncertainty. |
| `requires_more_evidence` | Evidence is insufficient for a formal next-run plan. |
| `repeat_or_repair_run` | Run or analysis must be repeated or repaired before formal review. |
| `multi_run_comparison_needed` | Bounded comparison is needed before choosing one direction. |
| `method_redesign_discussion_only` | Outside tuning surface; do not include executable training command unless the user separately approves redesign work. |

Rules:
- If gates pass and prior hypothesis is refuted, identify the next unresolved tuning uncertainty.
- Choose `multi_run_comparison_needed` for bounded robustness, seed-variation, or branch comparison before choosing one direction.
- Tie every executable recommendation to evidence and hypothesis.

## 7. User Output

Standard Chinese sections:
- `证据准入结论`
- `上一轮假设验证`
- `工程约束核对`
- `历史链路综合`
- `本轮指标解释`
- `下一轮建议与验证重点`

Output rules:
- State the recommendation explicitly.
- For `next_run_plan`, include a concrete PowerShell launch command and validation focus.
- For `multi_run_comparison_needed`, include bounded comparison plan and uncertainty resolved.
- For `hold_current_baseline`, state it is not global optimality and identify unresolved uncertainty or pause reason.
- For insufficient evidence, state blocking condition and required evidence action.
- Omit full `tuning_review_payload` unless archive-ready output or Codex archive prompt preparation is requested.

## 8. Archive Payload Contract

| Field | Contract |
| --- | --- |
| `schema_version` | Required constant: `"1.0"`. |
| `report_type` | Required constant: `"gpt_tuning_review"`. |
| `generated_by` | Required constant: `"gpt"`. |
| `review_scope` | `single_run_or_multi_run` scope label. |
| `source_analysis_archive_id` | Archive id or `null`. |
| `source_run_name_or_group` | Source run name or group label. |
| `evidence_gate` | `status`, `reproducible_launch_status`, `factual_summary_status`, `blocking_reason`; `status` is `passed` or `blocked`. |
| `engineering_context` | Manifest used, method mainline, tuning surface, redesign requirement, engineering constraints. |
| `prior_rationale_validation` | Prior archive id, prior recommendation type, prior hypothesis, fixed `validation_status` enum, supporting evidence, contradicting evidence, unverified items. |
| `historical_trajectory` | History entries used, chain summary, repeated/resolved/remaining uncertainties, tested dimensions when useful. |
| `current_evidence_interpretation` | Train-side monitoring, posthoc context, supplemental `final_probe`, contextual metric judgements, limitations. |
| `recommendation_type` | Uses the fixed enum in Section 6. |
| `recommended_next_command_summary` | Command metadata object or `null` when no executable next run is recommended. |
| `next_hypothesis` | Required for `next_run_plan` and `multi_run_comparison_needed`; `null` or minimal for `requires_more_evidence` without a run hypothesis. |
| `rationale` | Compact rationale list. |
| `expected_validation_focus` | Compact validation-focus list. |
| `limitations` | Compact limitations list. |

Portable command metadata:
- `working_directory = source_training_repo`
- `launcher = scripts/launch_formal_train_stable.ps1`
- `stable_reproducible_mode_required = true` for formal stable runs
- `command_template` uses `<source_training_repo>`
- `command_arguments` stores concrete launcher parameters
- Private local absolute paths are excluded from tracked history payloads.

## 9. Boundaries

| Boundary | Contract |
| --- | --- |
| `gate_boundary` | No formal recommendation when gates fail. |
| `evidence_role_boundary` | `final_probe` alone is not superiority evidence; posthoc winner alone is not method superiority evidence; runtime speed alone is not method-performance superiority evidence. |
| `recommendation_boundary` | No invented hyperparameters without evidence and hypothesis; `hold_current_baseline` is not proof of global optimum; refuted prior hypothesis is not automatic termination. |
| `artifact_boundary` | Do not directly use full logs, full CSVs, checkpoints, model weights, raw outputs, plots, or trajectories. |
| `archive_boundary` | Do not output archive payload JSON by default; exclude private local absolute paths from archive payload. |
| `metric_threshold_boundary` | No fixed universal metric thresholds without source authority. |
| `redesign_boundary` | Do not present method-level redesign as an unattended training command. |

## 10. Failure Cases

| Case | Handling |
| --- | --- |
| Missing current factual analysis | Use `requires_more_evidence` or `repeat_or_repair_run`. |
| Missing/failed reproducibility evidence | Use `requires_more_evidence` or `repeat_or_repair_run`. |
| Partial factual summary | Use `requires_more_evidence` unless formal review remains justified. |
| Missing history | State limitation; continue only if current evidence gate passes. |
| No prior rationale match | If gates pass, continue with `prior_rationale_validation.validation_status = not_applicable` and state history limitation. |
| Malformed history index | Use `requires_more_evidence` unless current-only formal review remains justified. |
| Missing archived payloads | State missing archive evidence and use available admissible evidence only. |
| Non-formal/debug/profile runs | Treat as diagnostic; do not use as formal tuning evidence. |
