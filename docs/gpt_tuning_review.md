# GPT Tuning Review

## 1. Purpose

Use this interface when GPT must read compact training evidence, validate the prior hypothesis when available, and recommend the next bounded action for active single-seed train-side performance search under the formal stable run setting. GPT may update the current train-side reference baseline when admissible train-side evidence improves. Archiving and 落盘 are handled by a separate archive workflow and are out of scope for this tuning review interface.

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

## 3. Evidence Admission

| Check | Formal review value | Failed action |
| --- | --- | --- |
| `reproducible_launch_status` | `reproducible_launch_confirmed` | Do not provide `next_run_plan`; use `requires_more_evidence` or `repeat_or_repair_run`. |
| `factual_summary_status` | `factual_summary_ready` | Do not provide `next_run_plan`; use `requires_more_evidence` or `repeat_or_repair_run`. |

If `reproducible_launch.contract_verdict == strict_contract_ready` is present, use it as support for `reproducible_launch_status`, not as a separate gate.

If either required status is missing or not in the required value, explain the blocking condition and the evidence action needed before formal tuning judgement.

## 4. Evidence Roles And Judgement Rules

- Evaluate system performance under the current specified seed unless the user explicitly requests cross-seed robustness.
- Treat train-side monitoring as the primary decision evidence for the active tuning loop.
- Use train-side endpoint metrics, late-stage delta, and key diagnostics to decide train-side system performance.
- Treat posthoc selection as checkpoint-selection and stability context.
- Treat `final_probe` as supplemental held-out diagnostic evidence.
- Record final_probe/posthoc mismatch as a limitation or validation focus, not an automatic blocker for train-side baseline update or `next_run_plan`.
- Treat cross-seed confirmation as optional follow-up unless the user explicitly requests robustness; it is not the default blocker.
- Current admissible evidence may update the current train-side reference baseline when train-side evidence improves over the previous reference baseline.
- A train-side reference baseline update is not a global optimum claim, paper-level superiority claim, or tuning completion.
- Use no fixed universal metric thresholds unless sourced from implementation, artifact schema, accepted protocol, or user instruction.

`prior_rationale_validation.validation_status` enum:
- `supported`
- `partially_supported`
- `refuted`
- `not_verifiable`
- `not_applicable`

## 5. Recommendation Decision

`recommendation_type` enum:
- `next_run_plan`
- `hold_current_baseline`
- `requires_more_evidence`
- `repeat_or_repair_run`
- `multi_run_comparison_needed`
- `method_redesign_discussion_only`

Decision rules:
- Use `next_run_plan` when admissible train-side evidence supports one concrete bounded performance-improvement run.
- If current run improves train-side performance, identify the next parameter, config, or reward dimension likely to improve performance further instead of defaulting to robustness confirmation.
- Broader tuning directions may include exploration schedule, learning/training dynamics, replay/update dynamics, and reward-function-related parameters when configurable or proposed as bounded implementation work.
- Use `hold_current_baseline` only when no grounded next direction is available or the user asks to pause.
- Use `requires_more_evidence` when current evidence cannot support formal judgement.
- Use `repeat_or_repair_run` when the run or factual report must be repaired or repeated before review.
- Use `multi_run_comparison_needed` only when bounded alternatives must be compared before choosing a direction, not merely because final_probe, posthoc, or cross-seed confirmation is incomplete.
- Use `method_redesign_discussion_only` only for method-level changes outside tuning/config scope.
- If a change can be launched through existing parameters, provide `next_run_plan` with a launcher-only command.
- If a change requires code/config editing outside existing launcher parameters, state the bounded implementation task instead of inventing a launcher command.

## 6. Output Format

Required Chinese sections:
- `证据准入结论`
- `上一轮假设验证`
- `历史链路综合`
- `本轮指标解释`
- `下一轮建议与验证重点`

Optional Chinese section:
- `工程约束核对`

Output requirements:
- State `recommendation_type`.
- State whether the current run becomes the current train-side reference baseline when applicable.
- For `next_run_plan`, include target uncertainty, hypothesis, expected validation focus, and a launcher-only PowerShell command when supported by existing launcher/config parameters.
- Command block starts with `.\scripts\launch_formal_train_stable.ps1`; do not prepend `cd <source_training_repo>;`, do not use local absolute paths, and do not include working-directory placeholders.
- If the next change requires code/config editing rather than a launcher parameter, state the bounded implementation task instead of inventing a launcher command.
- Do not output `tuning_review_payload` JSON, archive digest, or Codex archive prompt.

## 7. Direct Boundaries

- `final_probe` alone cannot prove superiority.
- Posthoc winner alone cannot prove method superiority.
- Runtime speed alone cannot prove method-performance superiority.
- Recommendations must tie parameter, config, or reward changes to evidence and a testable hypothesis.
- Method-level redesign requires separate explicit user approval.
- Do not inspect or rely on full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, or generated run artifacts.
