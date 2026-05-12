# GPT Tuning Review

## 1. Purpose

Use this interface when GPT must read compact training evidence and produce an objective, evidence-grounded, bounded tuning recommendation.

Core task:
- Check whether current evidence is admissible for formal tuning judgement.
- Validate the prior hypothesis or prior recommendation when available.
- Interpret current run evidence in historical context.
- Identify the remaining tuning uncertainty.
- Recommend the next bounded tuning action.
- Include an executable PowerShell launch command only for `recommendation_type = next_run_plan`.

Archiving and 落盘 are handled by a separate archive workflow and are out of scope for this tuning review interface.

## 2. Evidence Priority

| Priority | Evidence | Use |
| --- | --- | --- |
| Required | `training_results/current/current_training_run_analysis.json` or `training_results/current/current_multi_run_analysis.json` | Current compact factual evidence and gate status. |
| Use when available | Relevant `training_results/history/history_index.json` entries | Locate matched prior tuning review path, tuning-chain continuity, and prior experiments. |
| Use when available | Matched archived `tuning_review.md` digest, or legacy `tuning_review.json` when the relevant history entry still points to the old format. | Prior hypothesis, prior recommendation, expected validation focus, and command intent. |
| Use when available | Relevant archived factual analyses | Historical evidence context. |
| Use only when needed | `docs/training_system_manifest.md` | Tuning surface, launcher context, method mainline, and method-redesign boundary. |

Evidence rules:
- Use compact factual reports and tracked history.
- Use `history_index.json` to locate the relevant prior tuning review path.
- Read the matched prior review in whatever format `history_index.json` records: `tuning_review.md` digest for new single-run archives, or `tuning_review.json` for legacy archives.
- Do not require old archives to be migrated, and do not treat legacy `tuning_review.json` presence as a blocker.
- Require Codex factual analysis before raw/full artifact evidence is used.
- Do not directly inspect full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, or generated run artifacts.

## 3. Evidence Admission

| Check | Formal review value | Failed action |
| --- | --- | --- |
| `reproducible_launch_status` | `reproducible_launch_confirmed` | Do not provide `next_run_plan`; use `requires_more_evidence` or `repeat_or_repair_run`. |
| `factual_summary_status` | `factual_summary_ready` | Do not provide `next_run_plan`; use `requires_more_evidence` or `repeat_or_repair_run`. |

If `reproducible_launch.contract_verdict == strict_contract_ready` is present, use it as support for `reproducible_launch_status`, not as a separate gate.

If either required status is missing or not in the required value, explain the blocking condition and the evidence action needed before formal tuning judgement.

## 4. Analysis Procedure

1. Identify the current run intent from run name, compact config, current report facts, and matched prior review when available.
2. Match the current run to the prior hypothesis or prior recommendation when available.
3. Set `prior_rationale_validation.validation_status` using the fixed enum below.
4. Separate supporting evidence, contradicting evidence, unverified items, and limitations.
5. Interpret train-side monitoring as primary evidence.
6. Treat posthoc checkpoint selection as checkpoint-selection context.
7. Treat `final_probe` as supplemental held-out validation only.
8. Exclude runtime speed alone, posthoc winner alone, and `final_probe` alone as method-performance superiority evidence.
9. Compare current evidence with relevant historical trajectory.
10. Identify repeated, resolved, and remaining tuning uncertainties.
11. Decide whether to continue one variable, switch variable, compare bounded alternatives, hold a reference baseline, or request more evidence.
12. Recommend the next bounded tuning action.

`prior_rationale_validation.validation_status` enum:
- `supported`
- `partially_supported`
- `refuted`
- `not_verifiable`
- `not_applicable`

Metric judgement:
- Use no fixed universal thresholds unless sourced from implementation, artifact schema, accepted protocol, or user instruction.
- Ground metric interpretation in current facts, prior hypothesis, history, direction consistency, checkpoint behavior, best-vs-last or winner-vs-last context, train-final consistency, reproducibility, and `final_probe` limitations.
- Strict reproducibility means small movements are not dismissed as random training noise by default.
- Reproducibility does not remove evaluation-sample, held-out seed-set, or recent-window limitations.

## 5. Recommendation Decision

`recommendation_type` enum:
- `next_run_plan`
- `hold_current_baseline`
- `requires_more_evidence`
- `repeat_or_repair_run`
- `multi_run_comparison_needed`
- `method_redesign_discussion_only`

Decision rules:
- Use `next_run_plan` when admissible evidence supports one executable bounded formal run.
- If `next_run_plan`, include target uncertainty, hypothesis, expected validation focus, and why this run is the next bounded tuning action.
- Use `hold_current_baseline` only as a reference-baseline or pause decision; do not present it as global optimum, tuning completion, or proof that no improvement is possible.
- Use `requires_more_evidence` when current evidence cannot support formal judgement.
- Use `repeat_or_repair_run` when the run or factual report must be repaired or repeated before review.
- Use `multi_run_comparison_needed` when bounded alternatives should be compared before choosing one direction.
- Use `method_redesign_discussion_only` only for issues outside the tuning surface; do not include unattended executable training commands.

## 6. Output Format

Required Chinese sections:
- `证据准入结论`
- `上一轮假设验证`
- `历史链路综合`
- `本轮指标解释`
- `下一轮建议与验证重点`

Optional Chinese section:
- `工程约束核对`

Use `工程约束核对` only when the recommendation depends on launcher behavior, tuning surface, method-redesign boundary, non-standard run mode, or executable command generation.

Output requirements:
- State `recommendation_type` explicitly.
- Explain the evidence basis for the recommendation.
- For `next_run_plan`, include a concrete executable PowerShell launcher command. The command block must be launcher-only and assume execution from the source training repository root. Start with `.\scripts\launch_formal_train_stable.ps1 ...`; do not prepend `cd <source_training_repo>;`, do not use local absolute paths, and do not include working-directory placeholders inside the command block. If a working-directory reminder is needed, state it in prose outside the command block.
- For `multi_run_comparison_needed`, state the bounded comparison and the uncertainty it resolves.
- For `hold_current_baseline`, state the reference-baseline or pause reason and remaining uncertainty.
- For insufficient evidence, state the failed gate or missing evidence and required evidence action.

## 7. Direct Boundaries

- Formal `next_run_plan` requires admissible evidence.
- `final_probe` is supplemental and cannot alone prove superiority.
- Posthoc winner alone cannot prove method superiority.
- Runtime speed alone cannot prove method-performance superiority.
- Recommendations must tie hyperparameter changes to evidence and a testable hypothesis.
- Method-level redesign requires separate explicit user approval.
- Do not inspect or rely on full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, or generated run artifacts.
