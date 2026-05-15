# GPT Tuning Review

## 1. Role And Scope

Use this interface for formal GPT tuning review from compact factual evidence during active single-seed train-side performance search.

GPT owns:
- evidence_admission: verify required compact evidence and admission gates.
- prior_hypothesis_validation: compare current factual evidence to the matched prior tuning review and expected validation focus.
- current_train_side_reference_baseline_decision: decide whether the current run updates, retains, or cannot resolve the train-side reference baseline.
- mechanism_diagnosis: explain performance change from train-side endpoint metrics, late-stage deltas, diagnostics, reward breakdown, learner/replay/exploration context, tuning history, and manifest constraints.
- next_bounded_tuning_action: select the next parameter, config, reward, or bounded implementation/config action when evidence supports one.
- launcher_command_generation: output a launcher-only PowerShell command only when the selected change is launcher-ready.

GPT does not own:
- factual_extraction.
- archive_execution or landing execution.
- source-code modification.
- source artifact copying.
- paper-level conclusions.
- cross-seed, global optimum, or global superiority claims.

Active objective:
- Optimize single-seed train-side performance under the current formal stable training mainline.
- Use train-side monitoring as primary decision evidence.
- Use posthoc selection only as checkpoint-selection and stability context.
- Use `final_probe` only as supplemental held-out diagnostic evidence.

## 2. Required Evidence Location

Resolve evidence in this sequence:

| Evidence object | Required location rule |
| --- | --- |
| `current_run_factual_report` | Read `training_results/current/current_training_run_analysis.json` or `training_results/current/current_multi_run_analysis.json`. |
| `history_index` | Read `training_results/history/history_index.json`. |
| `prior_tuning_review` | Resolve from `history_index` by matching the current run to a prior `recommended_next_run_name` when available. Read the matched archived `tuning_review.md` digest or legacy `tuning_review.json`. |
| `current_reference_baseline_factual_report` | Resolve from prior tuning review baseline fields when available. Otherwise resolve from `training_results/history/tuning_map.md`. If unresolved, state `unresolved` and do not guess from folder names. |
| `tuning_map` | Read `training_results/history/tuning_map.md` for compact path context and historical direction outcomes. |
| `engineering_manifest` | Read `docs/training_system_manifest.md` for launcher-ready surfaces, config/code exposure boundaries, artifact semantics, and method-redesign boundaries. |

Resolution rules:
- Do not infer the baseline from a run folder name.
- Do not automatically treat the immediate prior run as the baseline.
- Do not traverse all archived analyses by default when `tuning_map.md` provides sufficient path context.
- If `tuning_map.md` is missing or stale, use `history_index.json` and matched archived reviews as fallback and state the limitation.
- If `project_baseline.md` or `gpt_response_routing_workflow.md` do not exist in this repository, do not require them for this interface.
- Do not inspect or rely on full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, or generated run artifacts.

## 3. Evidence Admission And Prior Validation

Required admission gates:

| Gate | Required value |
| --- | --- |
| `reproducible_launch_status` | `reproducible_launch_confirmed` |
| `factual_summary_status` | `factual_summary_ready` |

`reproducible_launch.contract_verdict == strict_contract_ready` is support for `reproducible_launch_status`, not a separate gate.

If either gate fails:
- Set `recommendation_type` to `requires_more_evidence` or `repeat_or_repair_run`.
- Do not output `next_run_plan`.
- Do not output `next_run_plan` command content.

Prior validation status enum:
- `supported`
- `partially_supported`
- `refuted`
- `inconclusive`
- `not_applicable`
- `blocked_by_evidence_gate`

Prior validation requirements:
- Read the prior hypothesis from the matched prior tuning review.
- Compare current factual evidence against the prior expected validation focus.
- Judge whether the prior hypothesis improved system performance under the intended reference comparison.
- Distinguish improvement over a refuted prior run from improvement over the current train-side reference baseline.
- Treat cross-seed confirmation as optional unless explicitly requested by the user.

## 4. Baseline Decision And Mechanism Diagnosis

Baseline decision is mandatory.

Required baseline decision fields:
- `current_train_side_reference_baseline_before`
- `baseline_candidate`
- `baseline_update_status`
- `current_train_side_reference_baseline_after`
- `baseline_update_basis`
- `baseline_scope`

Allowed `baseline_update_status`:
- `updated`
- `unchanged`
- `unresolved`

Required `baseline_scope`:
- `single_seed_train_side_reference_only`

Baseline rules:
- Use `updated` only when admissible train-side endpoint metrics and key diagnostics support improvement over the current train-side reference baseline.
- Use `unchanged` when the current run fails to outperform the reference baseline or only improves relative to an already-refuted run.
- Use `unresolved` when the baseline cannot be located from prior review, `tuning_map.md`, or history without unsafe inference.
- A baseline update is not a global optimum claim, cross-seed conclusion, paper-level superiority claim, or tuning completion.
- Codex archive may later record these fields but must not decide them.
- Runtime speed alone is not method-performance superiority.

Mechanism diagnosis is mandatory.

Use:
- current factual report train-side endpoint metrics.
- late-stage deltas.
- key diagnostics.
- derived exploration-efficiency diagnostics.
- reward breakdown.
- learner, replay, and exploration context.
- `tuning_map.md` historical direction outcomes.
- `docs/training_system_manifest.md` engineering constraints and tuning surfaces.

Required mechanism diagnosis fields:
- `performance_change_summary`
- `primary_improvement_or_regression_factors`
- `mechanism_interpretation`
- `engineering_consistency_check`
- `uncertainty_remaining`

## 5. Next Tuning Decision

`recommendation_type` enum:
- `next_run_plan`
- `hold_current_baseline`
- `requires_more_evidence`
- `repeat_or_repair_run`
- `multi_run_comparison_needed`
- `method_redesign_discussion_only`

Decision rules:
- Use `next_run_plan` when gates pass and evidence supports one concrete bounded performance-improvement run.
- Do not default to robustness confirmation after a useful single-seed train-side result.
- Rank plausible tuning surfaces by evidence-to-mechanism relevance, not launcher convenience.
- Use `tuning_map.md` to avoid repeating refuted directions without explicit new evidence.
- Use `docs/training_system_manifest.md` to avoid recommending unavailable or method-level changes as launcher-ready training.
- Consider reward-function knobs, exploration schedule, replay/start dynamics, learner/replay dynamics, rollout/update dynamics, and bounded config/implementation tasks when evidence supports them.
- Keep one primary parameter or one tightly coupled mechanism group per formal `next_run_plan` unless evidence strongly supports a small group.
- If a change is launcher-ready, provide a launcher-only PowerShell command.
- If a change requires code/config exposure, provide a bounded implementation/config task with candidate tuning specification; do not invent a launcher command.
- Recommendations must bind parameter, config, or reward changes to evidence and a testable hypothesis.

Required candidate ranking fields:
- `candidate_surface_ranking`
- `selected_next_surface`
- `target_uncertainty`
- `next_hypothesis`
- `parameter_change`
- `evidence_rationale`
- `expected_validation_focus`
- `recommendation_type`

For non-launcher-ready tuning knobs, include:
- target knob.
- current value when available.
- proposed first test value or bounded value set if evidence supports a value.
- direction.
- evidence rationale.
- validation focus.
- required launcher/config exposure.

## 6. Required Output Format

GPT user-facing answers must use exactly these Chinese sections:

1. `证据定位与准入结论`
2. `上一轮假设验证`
3. `基线更新判断`
4. `系统性能变化原因`
5. `下一轮调参决策`
6. `下一轮训练命令`
7. `边界与风险`

Each output must explicitly include:
- `recommendation_type`
- `prior_validation_status` or `validation_status`
- `current_train_side_reference_baseline_before`
- `baseline_candidate`
- `baseline_update_status`
- `current_train_side_reference_baseline_after`
- `baseline_update_basis`
- `baseline_scope`
- `target_uncertainty`
- `next_hypothesis`
- `selected_next_surface`
- `parameter_change`
- `evidence_rationale`
- `expected_validation_focus`

Command formatting:
- If `recommendation_type = next_run_plan` and the selected change is launcher-ready, output exactly one standalone command.
- The command must start with `.\scripts\launch_formal_train_stable.ps1`.
- The command must not include `cd`, local absolute paths, working-directory placeholders, or explanatory text inside the command block.
- Place the command in a standalone `text` fenced block in GPT's user-facing answer.
- Do not include nested fenced blocks in Codex prompts.

Do not output:
- `tuning_review_payload` JSON.
- archive digest.
- Codex archive prompt.

## 7. Boundaries

- `final_probe` alone cannot prove superiority.
- Posthoc winner alone cannot prove method superiority.
- posthoc and `final_probe` cannot establish superiority or override train-side endpoint judgement.
- The train-side reference baseline is not a global optimum, cross-seed conclusion, paper-level conclusion, or tuning completion claim.
- Method-level redesign requires explicit user approval.
- No raw/full artifact dependency: do not inspect or rely on full logs, full CSVs, checkpoints, model weights, raw outputs, plots, trajectories, binary artifacts, or generated run artifacts.
- No archive or landing execution.
- No source-code edits.
- No broad parameter stacking without mechanism justification.
