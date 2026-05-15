# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_103843

source_archive_id: stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_103843

recommendation_type: next_run_plan

prior_validation_status: refuted

validation_status: refuted

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_103843

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: learner/replay dynamics

parameter_change: BatchSize: 128 -> 256; LearningRate: 0.000075 -> 0.0001 restored to epsend004 core

recommended_next_run_name: stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: LearningRate=0.000075 produced partial repair versus the already-refuted turn006 run but failed to outperform the current train-side reference baseline epsend004 and did not support the learner-noise reduction hypothesis; keep epsend004 as the current single-seed train-side reference baseline and test BatchSize=256 while restoring LearningRate=0.0001 to epsend004 core.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The factual report satisfied the two direct evidence gates. reproducible_launch.contract_verdict == strict_contract_ready was treated only as support for reproducible_launch_status, not as a third gate. The run was train-side-only with posthoc_selection and supplemental_final_probe skipped_train_side_only_tuning and not_missing, so posthoc/final_probe were not evidence sources.

## 3. Prior Validation

prior_hypothesis: Restore epsend004 reward/exploration core settings and reduce LearningRate from 0.0001 to 0.000075 to test whether a more conservative learner update can reduce TD/gradient update noise and improve endpoint stability, behavior efficiency, and train-side performance.

validation_status: refuted

judgement: The lr75e5 run repaired some metrics relative to the already-refuted turn006 run, but it did not improve over the current train-side reference baseline epsend004 and did not support the mechanism claim that lowering LearningRate to 0.000075 reduces learner noise.

key_evidence:
- lr75e5 improved locally over refuted turn006 in reward, coverage, success_rate, timeout, stall, and zero_info, but improvement over a refuted run does not update the reference baseline.
- lr75e5 remained weaker than epsend004 on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, and key behavior diagnostics.
- Learner context did not support the learner-noise reduction hypothesis because loss, td_abs_mean, and grad_norm were higher than epsend004 and also higher than the prior turn006 context.
- posthoc_selection and supplemental_final_probe were skipped by train_side_only_tuning and were status-only test-side objects, not evidence sources.

remaining_uncertainty: Whether restoring epsend004 learning rate and increasing BatchSize from 128 to 256 can reduce minibatch gradient variance and improve endpoint stability, behavior efficiency, and train-side system performance without continuing reward-penalty or exploration-schedule directions already refuted in history.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. lr75e5 learned relative to its own initial phase and partially repaired the prior failed turn006 direction, but it remained weaker than epsend004 on endpoint system performance and key diagnostics, so the current reference baseline remains unchanged.

mechanism_summary: Lowering LearningRate from 0.0001 to 0.000075 did not deliver the intended learner-noise reduction; learner scalars worsened rather than stabilizing, and reward breakdown still showed lower information reward and terminal bonus with heavier revisit, turn, and timeout penalty burden than epsend004. The evidence supports leaving reward/exploration core unchanged and testing minibatch variance reduction through BatchSize rather than continuing single learning-rate reduction.

posthoc_context: posthoc_selection.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as checkpoint-selection evidence or superiority evidence.

final_probe_supplemental: supplemental_final_probe.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as supplemental evidence for this train-side-only run.

main_limitation: This is a single-seed train-side reference decision only. It does not establish global optimum, cross-seed robustness, paper-level superiority, or tuning completion.

## 5. Next Action

target_uncertainty: Under epsend004 core settings, can increasing BatchSize from 128 to 256 reduce minibatch gradient variance and improve endpoint stability, behavior efficiency, and train-side system performance while avoiding the failure modes observed under reward-penalty increments, exploration-schedule changes, and lower learning rate.

next_hypothesis: Restore epsend004 reward, exploration, and learning-rate core settings, and increase BatchSize from 128 to 256. A larger minibatch may smooth Q updates and improve endpoint reward, coverage, success_rate, RVR, timeout, stall, zero_info, and terminal quality without further reducing learning rate.

selected_next_surface: learner/replay dynamics

parameter_change: BatchSize: 128 -> 256; LearningRate: 0.000075 -> 0.0001 restored to epsend004 core

rationale: EpsilonEnd=0.035, EpsilonDecaySteps=300000, RewardRevisitPenalty=0.12, RewardTurnPenaltyScale=0.06, and LearningRate=0.000075 have all failed to update the epsend004 reference baseline. The next bounded learner/replay test should restore epsend004 core settings and use BatchSize=256 as the single primary parameter to test whether lower minibatch variance improves train-side system performance.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25 -BatchSize 256 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 2 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

expected_validation_focus:
- Compare against stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 as the current train-side reference baseline.
- Focus on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, turn_ge_90, turn_135, and turn_180.
- Check learner context: loss, td_abs_mean, and grad_norm should become more stable, but learner scalars must not be used as standalone superiority evidence.
- Check reward breakdown: info_reward_sum, recent_revisit_penalty_sum, turn_penalty_sum, timeout_penalty_sum, and terminal_bonus_sum should approach or exceed epsend004 quality.
- Check derived exploration-efficiency diagnostics as diagnostic-only evidence: coverage_gain_per_step, weighted_info_gain_per_step, zero_info_rate, recent_revisit_rate, stall_rate, turn_burden_rate, and timeout_rate.
- If BatchSize=256 is still weaker than epsend004, do not continue one-off single-parameter serial probing by default; consider bounded multi-run comparison or rollout/update cadence surface.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline; this is not a global optimum claim, cross-seed conclusion, paper-level superiority claim, or tuning completion claim.

evidence_boundary: Train-side monitoring is the primary evidence. posthoc_selection and supplemental_final_probe were skipped_train_side_only_tuning status-only objects and cannot establish superiority or evidence insufficiency.

redesign_boundary: This recommendation is launcher-ready learner/replay-parameter tuning within the current method. It does not authorize method-level redesign, architecture changes, state representation changes, replay algorithm redesign, reward mechanism rewrite, action-space changes, environment-semantics changes, or paper-level claims.
