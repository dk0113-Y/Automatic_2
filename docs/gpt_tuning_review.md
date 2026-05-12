# GPT Tuning Review

## 1. Purpose

Use this interface when GPT must read compact training evidence and produce an objective, evidence-grounded, bounded tuning recommendation for the active formal stable run setting.

Core task:
- Check whether current evidence is admissible for formal tuning judgement.
- Validate the prior hypothesis or prior recommendation when available.
- Optimize single-seed train-side system performance under the specified active run setting.
- Interpret current run evidence in historical context.
- Update the current train-side reference baseline when admissible evidence supports improvement.
- Identify the remaining performance-improvement uncertainty.
- Recommend the next bounded performance-improvement action.
- Include an executable PowerShell launch command only when `recommendation_type = next_run_plan` and the change is supported by existing launcher/config parameters.

Archiving and 落盘 are handled by a separate archive workflow and are out of scope for this tuning review interface.

## 2. Evidence Priority

| Priority | Evidence | Use |
| --- | --- | --- |
| Required | `training_results/current/current_training_run_analysis.json` or `training_results/current/current_multi_run_analysis.json` | Current compact factual evidence and gate status. |
| Use when available | Relevant `training_results/history/history_index.json` entries | Locate matched prior tuning review path, tuning-chain continuity, and prior experiments. |
| Use when available | Matched archived `tuning_review.md` digest, or legacy `tuning_review.json` when the relevant history entry still points to the old format. | Prior hypothesis, prior recommendation, expected validation focus, and command intent. |
| Use when available | Relevant archived factual analyses | Historical train-side reference context. |
| Use only when needed | `docs/training_system_manifest.md` | Tuning surface, launcher context, configurable parameters, method mainline, and method-redesign boundary. |

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

## 4. Evidence Role And Decision Priority

- Evaluate system performance under the current specified seed unless the user explicitly requests cross-seed robustness.
- Treat train-side monitoring as the primary decision evidence for active parameter search.
- Use train-side endpoint metrics, late-stage delta, and key diagnostics to decide whether a run improves train-side system performance.
- Treat posthoc checkpoint selection as checkpoint-selection and stability context.
- Treat `final_probe` as supplemental held-out diagnostic evidence.
- Record final_probe/posthoc mismatch as a limitation or validation focus, not an automatic reason to block `next_run_plan`.
- Treat cross-seed confirmation as optional follow-up unless the user explicitly requests robustness; do not make it the default blocker for continuing single-seed performance tuning.
- Exclude runtime speed alone, posthoc winner alone, and `final_probe` alone as system-performance superiority evidence.

## 5. Analysis Procedure

1. Identify the current run intent from run name, compact config, current report facts, and matched prior review when available.
2. Match the current run to the prior hypothesis or prior recommendation when available.
3. Set `prior_rationale_validation.validation_status` using the fixed enum below.
4. Separate supporting evidence, contradicting evidence, unverified items, and limitations.
5. Compare current train-side endpoint, late-stage trend, and diagnostics with the current train-side reference baseline.
6. Record posthoc and final_probe facts as context according to their evidence roles.
7. Identify repeated, resolved, and remaining performance-improvement uncertainties.
8. Decide whether to continue one variable, switch variable, compare bounded alternatives, hold a reference baseline, request more evidence, or identify a bounded implementation/config task.
9. Recommend the next bounded performance-improvement action.

`prior_rationale_validation.validation_status` enum:
- `supported`
- `partially_supported`
- `refuted`
- `not_verifiable`
- `not_applicable`

Metric judgement:
- Use no fixed universal thresholds unless sourced from implementation, artifact schema, accepted protocol, or user instruction.
- Ground metric interpretation in current facts, prior hypothesis, history, endpoint direction, late-stage direction, diagnostic movement, checkpoint behavior, reproducibility, and final_probe limitations.
- Strict reproducibility means small movements are not dismissed as random training noise by default.
- Reproducibility does not remove evaluation-sample, held-out seed-set, or recent-window limitations.

## 6. Baseline Update Rule

- If current admissible evidence shows clear train-side improvement over the current reference baseline, GPT may mark the current run as the current train-side reference baseline for the active tuning loop.
- Baseline update evidence should cite train-side endpoint metrics, late-stage trend, and key diagnostics.
- A current train-side reference baseline is not a global optimum claim, paper-level superiority claim, tuning completion claim, or proof that no further improvement is possible.
- Final_probe inconsistency, posthoc mismatch, or missing cross-seed robustness may remain validation focus without blocking a train-side baseline update.

## 7. Recommendation Decision

`recommendation_type` enum:
- `next_run_plan`
- `hold_current_baseline`
- `requires_more_evidence`
- `repeat_or_repair_run`
- `multi_run_comparison_needed`
- `method_redesign_discussion_only`

Decision rules:
- Use `next_run_plan` when admissible train-side evidence supports one concrete bounded performance-improvement run.
- If current run improves train-side performance, do not default to `multi_run_comparison_needed` only to confirm final_probe, posthoc, or cross-seed robustness; identify the next likely performance-improvement dimension.
- Candidate next dimensions may include exploration schedule, learning/training dynamics, replay/update dynamics, and reward-function-related parameters, subject to manifest or implementation constraints.
- Use `hold_current_baseline` only when no sufficiently grounded next tuning direction is available or the user asks to pause.
- Use `requires_more_evidence` when current evidence cannot support formal judgement.
- Use `repeat_or_repair_run` when the run or factual report must be repaired or repeated before review.
- Use `multi_run_comparison_needed` only when multiple bounded alternatives must be compared before choosing a direction.
- Use `method_redesign_discussion_only` only for method-level changes outside the tuning/config surface; do not include unattended executable training commands.
- If a next change is exposed through existing launcher/config parameters, provide a launcher-only `next_run_plan`.
- If a next change requires code/config editing outside existing launcher parameters, state the bounded implementation/change task needed before training instead of inventing a launcher command.

## 8. Output Format

Required Chinese sections:
- `证据准入结论`
- `上一轮假设验证`
- `历史链路综合`
- `本轮指标解释`
- `下一轮建议与验证重点`

Optional Chinese section:
- `工程约束核对`

Use `工程约束核对` when the recommendation depends on launcher behavior, tuning surface, implementation/config boundary, method-redesign boundary, non-standard run mode, or executable command generation.

Output requirements:
- State `recommendation_type` explicitly.
- State whether the current run becomes the current train-side reference baseline when applicable.
- Explain the train-side evidence basis for the recommendation.
- For `next_run_plan`, include a concrete executable PowerShell launcher command when the change is supported by existing launcher/config parameters. The command block must start with `.\scripts\launch_formal_train_stable.ps1`; do not prepend `cd <source_training_repo>;`, do not use local absolute paths, and do not include working-directory placeholders. If a working-directory reminder is needed, state it in prose outside the command block.
- If the next recommended change requires code/config editing rather than a launcher parameter, state the bounded implementation task instead of inventing a launcher command.
- For `multi_run_comparison_needed`, state the bounded comparison and the uncertainty it resolves.
- For `hold_current_baseline`, state the reference-baseline or pause reason and remaining uncertainty.
- For insufficient evidence, state the failed gate or missing evidence and required evidence action.
- Do not output archive payload JSON, archive digest, or Codex archive prompt.

## 9. Direct Boundaries

- Formal `next_run_plan` requires admissible evidence.
- `final_probe` alone cannot prove superiority, but final_probe inconsistency alone does not block train-side baseline update.
- Posthoc winner alone cannot prove method superiority, but posthoc mismatch alone does not block `next_run_plan`.
- Runtime speed alone cannot prove system-performance superiority.
- Recommendations must tie parameter, config, reward, or implementation-bound changes to evidence and a testable hypothesis.
- Method-level redesign requires separate explicit user approval.
- Do not inspect or rely on full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, or generated run artifacts.
