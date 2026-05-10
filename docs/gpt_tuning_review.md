# GPT Tuning Review

## 1. Document Role

This document defines a GPT-side tuning review workflow for deriving a bounded next tuning recommendation from:

- `docs/training_system_manifest.md`
- `training_results/current/current_training_run_analysis.json`
- `training_results/history/history_index.json`
- archived `training_run_analysis.json` or `multi_run_analysis.json`
- archived `tuning_review.json`

This workflow produces a user-facing Chinese tuning review with an explicit next-step recommendation.

The archivable `tuning_review_payload` is the structured content contract used when the user requests archive/落盘 or when GPT prepares a separate Codex archive prompt. It is not mandatory in the default tuning-review response unless the user requests archive-ready output.

This document is not a Codex execution skill, a GPT-to-Codex prompt packaging document, a training-system manifest, a Codex factual-analysis extractor, a history archive writer, a raw training-log parser, a method-redesign workflow, or a paper-level conclusion policy.

GPT owns evidence interpretation after admissibility gates pass, prior rationale validation, historical tuning-chain synthesis, bounded next recommendation, user-facing tuning review, and structured tuning rationale content when archive-ready output is requested.

Codex owns factual analysis extraction through concrete Codex skills, archive writing through concrete Codex skills, and bounded file inspection and validation when explicitly assigned.

## 2. Required Inputs

Required inputs:

- engineering manifest: `docs/training_system_manifest.md`
- current factual analysis JSON
- history index JSON
- relevant archived factual analyses
- relevant archived GPT tuning reviews

GPT must not directly inspect or parse full logs, full CSVs, checkpoints, model weights, raw output directories, plots, or trajectories. Raw artifacts must be summarized first by a Codex factual-analysis task before GPT uses them for formal tuning review.

## 3. Evidence Admissibility Gate

GPT may enter formal tuning review only when the current factual analysis supports formal evidence use.

Required conditions:

- `reproducible_launch_status` is `reproducible_launch_confirmed`
- `factual_summary_status` is `factual_summary_ready`

If either condition is not met, GPT must not produce a formal next-run plan. `recommendation_type` must be `requires_more_evidence` or `repeat_or_repair_run`, depending on the failure mode.

Non-formal, debug, smoke, profile, incomplete, missing, contradictory, or blocked runs must not be treated as formal tuning evidence. GPT may still provide a bounded diagnostic explanation of missingness or repair needs.

## 4. Active Optimization Objective

Formal GPT tuning review is an active optimization workflow within the valid engineering tuning surface. Its goal is not only to summarize the current run, but also to reduce the most important remaining tuning uncertainty and improve system performance through bounded, evidence-grounded recommendations.

When evidence gates pass, GPT should identify the next actionable uncertainty unless the user explicitly asks to pause. Refuting one prior hypothesis should redirect the tuning chain to the next justified variable or comparison plan; it should not automatically terminate active tuning.

GPT must not claim that the current configuration is globally optimal unless the available evidence directly supports that claim. GPT must not claim that no further performance improvement is possible from a single-run or single-seed chain.

This objective does not force a `next_run_plan` when evidence supports `multi_run_comparison_needed` or when the user requests a pause.

## 5. Engineering Grounding

`docs/training_system_manifest.md` is the engineering grounding layer. GPT must use it to constrain:

- current algorithmic mainline
- training lifecycle
- artifact semantics
- configuration surface
- reproducibility and launcher context
- legacy/non-mainline boundaries
- distinction between tuning and method-level redesign

The manifest is not a tuning guide, but it defines the valid engineering interpretation space and the valid tuning action surface. GPT must not propose a recommendation that conflicts with the manifest unless the user explicitly asks for a method-level redesign discussion.

## 6. Prior Rationale Extraction

GPT identifies the prior tuning rationale by this preferred matching order:

1. Find an archived `tuning_review.json` whose `recommended_next_command_summary.run_name` matches the current run name.
2. If unavailable, use `history_index.json` to identify the latest relevant archive entry in the same tuning chain.
3. If no prior rationale is available, set `prior_rationale_validation.validation_status` to `not_applicable` and continue with current evidence and history limitations clearly stated.

GPT must extract:

- prior `recommendation_type`
- prior hypothesis
- primary uncertainty
- expected validation focus
- expected supporting evidence
- expected refuting evidence
- limitations
- recommended command metadata

## 7. Prior Hypothesis Validation

GPT must evaluate the current factual analysis against the prior hypothesis before proposing the next recommendation.

Use this `validation_status` enum:

- `supported`
- `partially_supported`
- `refuted`
- `not_verifiable`
- `not_applicable`

The validation must explicitly separate supporting evidence, contradicting evidence, unverified items, and limitations. The prior tuning review defines the experiment intent, but it must not override the current factual evidence.

## 8. Historical Trajectory Synthesis

GPT must synthesize the historical tuning chain from `history_index.json` and relevant archived records.

The synthesis must identify:

- relevant history entries used
- chain summary
- repeated uncertainties
- resolved uncertainties
- remaining uncertainties
- parameter dimensions already tested
- whether the next recommendation continues a single-variable chain or switches variable

History prevents short-sighted result-driven tuning and repeated experiments, but old history must not override current formal evidence.

## 9. Contextual Metric Judgement

GPT may judge metric movement without fixed universal thresholds. GPT may state that reward supports a hypothesis, RVR indicates degradation, coverage is stable, success rate weakens a claim, or episode length changes are decision-relevant only when the judgement is grounded in:

- the prior hypothesis being tested
- metric semantics and artifact lifecycle from the manifest
- current factual analysis
- relevant historical records
- direction consistency across related metrics
- checkpoint-selection behavior
- best-vs-last or winner-vs-last context when available
- train-final consistency when available
- reproducibility status
- final_probe limitations

Strict reproducible evidence means small metric movements should not be dismissed as random training noise by default. Reproducibility does not eliminate evaluation-sample limitations, held-out seed-set limitations, or recent-window interpretation limits.

GPT must not invent fixed global thresholds for reward support, RVR degradation, coverage improvement, or success-rate change unless the threshold comes from the training implementation, artifact schema, accepted project protocol, or explicit user instruction.

## 10. Evidence Roles

- Current factual analysis decides admissibility and provides current run facts.
- `docs/training_system_manifest.md` defines engineering fact boundaries and valid action space.
- Latest prior `tuning_review.json` defines the experiment hypothesis and validation focus.
- Historical analyses and reviews define the tuning chain.
- Train-side monitoring is primary evidence for current training dynamics.
- Post-hoc selection is checkpoint-selection context.
- `final_probe` is supplemental held-out validation.
- Runtime speed alone is not method-performance superiority evidence.

## 11. Recommendation Modes

`recommendation_type` must use this enum:

- `next_run_plan`: formal evidence supports an executable next training run.
- `hold_current_baseline`: formal evidence supports retaining a current reference baseline for the present comparison or tuning chain. It does not mean global optimality, tuning completion, or absence of further performance improvement. In an active optimization workflow, this mode should be used only when pausing is justified, when no bounded next tuning action is supported, or when the user requests holding the baseline.
- `requires_more_evidence`: available evidence is insufficient for a formal next-run plan.
- `repeat_or_repair_run`: the run or analysis should be repeated or repaired before formal review.
- `multi_run_comparison_needed`: evidence supports comparing a bounded set of runs before choosing the next single direction.
- `method_redesign_discussion_only`: the issue is outside the tuning surface and belongs to a method-level discussion.

`method_redesign_discussion_only` must not include an executable training command unless the user separately approves a method-level redesign task.

If the prior hypothesis is refuted and evidence gates pass, GPT must identify the next unresolved tuning uncertainty. GPT should normally choose `next_run_plan` when a bounded single-variable next run is justified. GPT should choose `multi_run_comparison_needed` when robustness, seed variation, or branch comparison should be established before choosing one next direction.

GPT may choose `hold_current_baseline` as a reference-baseline decision, but must also state whether active tuning should pause or what unresolved uncertainty should be addressed next. A held baseline is a reference point, not proof of optimality.

## 12. User-Facing Output Requirements

The standard user-facing review structure is Chinese:

- `证据准入结论`
- `上一轮假设验证`
- `工程约束核对`
- `历史链路综合`
- `本轮指标解释`
- `下一轮建议与验证重点`

Evidence-insufficient cases may collapse the structure but must still explain the gate failure and next evidence action. User-facing recommendations should be clear enough to produce a concrete PowerShell launch command when `recommendation_type` is `next_run_plan`.

The user-facing review must provide an explicit next-step recommendation.

If `recommendation_type` is `next_run_plan`, include a concrete PowerShell launch command and validation focus.

If `recommendation_type` is `multi_run_comparison_needed`, describe the bounded comparison plan and the uncertainty it resolves.

If `recommendation_type` is `hold_current_baseline`, state clearly that this means retaining a reference baseline, not proving global optimality or ending active tuning; also state the next unresolved tuning uncertainty or why pausing is justified.

If evidence is insufficient, explain the blocking condition and required evidence action.

The default user-facing tuning review does not need to include a complete `tuning_review_payload` JSON unless the user requests archive-ready output or asks GPT to prepare a Codex archive prompt.

## 13. Payload Content For Archive Step

This payload contract defines the structured tuning rationale that GPT supplies when the user requests archive/落盘 or when GPT prepares a separate Codex archive prompt. It is not the default output of every tuning-review response.

The archivable payload must be JSON-compatible and use this top-level shape:

```json
{
  "schema_version": "1.0",
  "report_type": "gpt_tuning_review",
  "generated_by": "gpt",
  "review_scope": "single_run_or_multi_run",
  "source_analysis_archive_id": "<archive_id_or_null>",
  "source_run_name_or_group": "<run_name_or_group>",
  "evidence_gate": {
    "status": "passed_or_blocked",
    "reproducible_launch_status": "<status_or_null>",
    "factual_summary_status": "<status_or_null>",
    "blocking_reason": "<reason_or_null>"
  },
  "engineering_context": {
    "manifest_used": "docs/training_system_manifest.md",
    "method_mainline": "<manifest_mainline_summary>",
    "tuning_surface_used": [],
    "method_redesign_required": false,
    "engineering_constraints": []
  },
  "prior_rationale_validation": {
    "prior_archive_id": "<archive_id_or_null>",
    "prior_recommendation_type": "<recommendation_type_or_null>",
    "prior_hypothesis": "<hypothesis_or_null>",
    "validation_status": "supported_or_partially_supported_or_refuted_or_not_verifiable_or_not_applicable",
    "supporting_evidence": [],
    "contradicting_evidence": [],
    "unverified_items": []
  },
  "historical_trajectory": {
    "history_entries_used": [],
    "chain_summary": "",
    "repeated_uncertainties": [],
    "resolved_uncertainties": [],
    "remaining_uncertainties": []
  },
  "current_evidence_interpretation": {
    "train_side_monitoring": {},
    "posthoc_selection_context": {},
    "supplemental_final_probe": {},
    "contextual_metric_judgements": [],
    "limitations": []
  },
  "recommendation_type": "<recommendation_type>",
  "recommended_next_command_summary": {
    "working_directory": "source_training_repo",
    "launcher": "scripts/launch_formal_train_stable.ps1",
    "run_name": "<next_run_name>",
    "stable_reproducible_mode_required": true,
    "command_template": "cd <source_training_repo>; .\\scripts\\launch_formal_train_stable.ps1 <arguments>",
    "command_arguments": {}
  },
  "next_hypothesis": {
    "hypothesis_id": "<id>",
    "hypothesis": "<bounded_next_run_hypothesis>",
    "expected_supporting_evidence_next_run": [],
    "expected_refuting_evidence_next_run": []
  },
  "rationale": [],
  "expected_validation_focus": [],
  "limitations": []
}
```

Required field constraints:

- `schema_version` must be `"1.0"`.
- `report_type` must be `"gpt_tuning_review"`.
- `generated_by` must be `"gpt"`.
- `evidence_gate.status` must be `passed` or `blocked`.
- `prior_rationale_validation.validation_status` must use the fixed enum from Section 7.
- `recommendation_type` must use the fixed enum from Section 11.
- `recommended_next_command_summary` may be `null` when no executable next run is recommended.
- `next_hypothesis` is required for `next_run_plan` and `multi_run_comparison_needed`.
- `next_hypothesis` should be `null` or minimal for `requires_more_evidence` when no run hypothesis exists.

Portable command metadata requirements:

- `working_directory`: `source_training_repo`
- `launcher`: `scripts/launch_formal_train_stable.ps1`
- `stable_reproducible_mode_required`: `true` for formal stable runs
- `command_template` must use `<source_training_repo>`
- `command_arguments` must store concrete launcher parameters
- private local absolute paths must not be stored in tracked history payloads

## 14. Forbidden Reasoning And Output

GPT must not:

- make formal recommendations when evidence gates fail
- treat `final_probe` alone as superiority evidence
- treat post-hoc winner alone as method superiority evidence
- treat runtime speed alone as method superiority evidence
- invent next hyperparameters without tying them to evidence and hypothesis
- treat `hold_current_baseline` as proof of global optimum
- treat a refuted prior hypothesis as automatic termination of active tuning
- output an archivable payload JSON by default unless the user requests archive-ready output or Codex archive prompt preparation
- propose method-level redesign as an unattended training command
- use full logs, full CSVs, checkpoints, model weights, raw outputs, plots, or trajectories directly
- store private local absolute paths in archivable payloads
- use fixed universal metric thresholds without source authority

## 15. Failure And Insufficient Evidence Cases

Missing current factual analysis, missing or failed reproducibility evidence, partial factual summary, missing history, no prior rationale match, malformed history index, missing archived payloads, and non-formal/debug/profile runs must be handled explicitly.

These cases should produce `requires_more_evidence` or `repeat_or_repair_run` unless a formal review remains justified by available evidence. If no prior rationale match exists, GPT may still complete the review when the evidence gate passes, but the output must state the history limitation and set prior validation to `not_applicable`.
