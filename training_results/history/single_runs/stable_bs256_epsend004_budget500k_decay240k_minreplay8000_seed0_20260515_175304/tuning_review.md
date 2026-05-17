# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_175304

source_archive_id: stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_175304

recommendation_type: multi_run_comparison_needed

prior_validation_status: partially_supported

validation_status: partially_supported

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_175304

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: bounded multi-run comparison over learner/update dynamics

parameter_change: Candidate A BatchSize: 128 -> 192; Candidate B LearnerUpdatesPerIter: 2 -> 1; Candidate C TargetUpdateInterval: 1000 -> 2000

recommended_next_run_name: stable_bs192_epsend004_budget500k_decay240k_minreplay8000_seed0; stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0; stable_target2000_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: BatchSize=256 partially supported the learner-scalar stability sub-hypothesis because loss, td_abs_mean, and grad_norm improved relative to epsend004 and lr75e5, but it did not improve train-side system performance. Endpoint reward, coverage, success_rate, episode_length, RVR, timeout, key behavior diagnostics, and reward breakdown remained weaker than epsend004. Keep epsend004 as the current single-seed train-side reference baseline and use a bounded launcher-ready multi-run comparison over learner/update dynamics.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The factual report satisfied the two direct admission gates. reproducible_launch.contract_verdict == strict_contract_ready was treated only as support for reproducible_launch_status, not as a third gate. The run was train-side-only with posthoc_selection and supplemental_final_probe skipped_train_side_only_tuning and missingness_treatment not_missing; posthoc/final_probe were status-only test-side objects and were not evidence sources.

## 3. Prior Validation

prior_hypothesis: Restore epsend004 reward, exploration, and learning-rate core settings, then increase BatchSize from 128 to 256 to test whether a larger minibatch can reduce minibatch gradient variance and improve TD/gradient stability, endpoint behavior efficiency, and train-side system performance.

validation_status: partially_supported

judgement: The mechanism-level learner scalar sub-hypothesis was supported, but the train-side system-performance goal was refuted. bs256 had lower loss, td_abs_mean, and grad_norm than epsend004 and lr75e5, but it remained weaker than epsend004 on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, behavior diagnostics, and reward breakdown. Therefore it cannot update the baseline.

key_evidence:
- bs256 learner context improved: loss 2.3352, td_abs_mean 2.7075, grad_norm 32.6729.
- epsend004 learner context was weaker on those scalars: loss 2.8861, td_abs_mean 3.2611, grad_norm 35.1494.
- lr75e5 learner context was weaker: loss 3.6270, td_abs_mean 3.9767, grad_norm 47.5332.
- bs256 endpoint remained weaker than epsend004: reward 134.9958 vs 147.4969, coverage 0.9290 vs 0.9430, success_rate 0.66 vs 0.86, episode_length 447.21 vs 375.72, RVR 0.2516 vs 0.1873, timeout 0.34 vs 0.14.
- bs256 behavior diagnostics were weaker than epsend004: zero_info 260.90 vs 199.22, recent_revisit 62.55 vs 45.47, stall 184.96 vs 144.03, turn_ge_90 194.14 vs 135.42, turn_135 54.89 vs 31.12, turn_180 37.36 vs 32.75.
- bs256 reward breakdown remained weaker than epsend004: info_reward_sum 145.1109 vs 147.3457, step_penalty_sum -8.9442 vs -7.5144, recent_revisit_penalty_sum -6.2550 vs -4.547, turn_penalty_sum -5.3958 vs -3.8673, timeout_penalty_sum -2.72 vs -1.12, terminal_bonus_sum 13.20 vs 17.20.

remaining_uncertainty: Whether a bounded learner/update comparison can identify a local region where learner stability transfers to behavior efficiency: intermediate batch size, lower update-to-data ratio, or slower target sync cadence under epsend004 core settings.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. bs256 was an effective training run and improved learner scalar stability, but did not outperform epsend004 on endpoint system metrics or key diagnostics. The current reference baseline remains epsend004.

mechanism_summary: BatchSize=256 likely reduced minibatch-level update noise, but this smoother learner state did not improve policy behavior. It may have reduced effective policy correction speed or responsiveness to exploration errors under the 500k budget. Behavior efficiency and terminal quality regressed: zero_info, recent_revisit, stall, turn burden, timeout, step penalty, revisit penalty, turn penalty, timeout penalty, and terminal bonus were all worse than epsend004.

posthoc_context: posthoc_selection.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as checkpoint-selection evidence or superiority evidence.

final_probe_supplemental: supplemental_final_probe.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as supplemental held-out evidence for this train-side-only run.

main_limitation: This is a single-seed train-side reference decision only. It does not establish global optimum, cross-seed robustness, paper-level superiority, or tuning completion.

## 5. Next Action

target_uncertainty: Under epsend004 core settings, is the bs256 failure caused by batch size being too large and slowing policy adaptation, by an update-to-data ratio mismatch, or by target sync cadence; which learner/update sub-direction best transfers learner stability into endpoint behavior efficiency and train-side system performance.

next_hypothesis: A bounded multi-run comparison over learner/update dynamics should compare three launcher-ready candidates: intermediate BatchSize=192, lower LearnerUpdatesPerIter=1, and slower TargetUpdateInterval=2000. This comparison can identify whether learner/update dynamics contain a useful local direction without continuing unbounded serial single-parameter trials.

selected_next_surface: bounded multi-run comparison over learner/update dynamics

parameter_change: Candidate A BatchSize: 128 -> 192; Candidate B LearnerUpdatesPerIter: 2 -> 1; Candidate C TargetUpdateInterval: 1000 -> 2000

rationale: bs256 improved learner scalar stability but worsened endpoint system performance. Historical exploration schedule, reward penalty, and single learning-rate reduction directions have been refuted. The current evidence does not justify one single next_run_plan, but it does justify a bounded comparison across closely related launcher-ready learner/update variants.

command:
Candidate A
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_bs192_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25 -BatchSize 192 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 2 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

Candidate B
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 1 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

Candidate C
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_target2000_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 2000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 2 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

expected_validation_focus: Compare all three candidates against epsend004 as the current train-side reference baseline. Primary metrics are endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, turn_ge_90, turn_135, and turn_180. Learner scalars loss, td_abs_mean, and grad_norm are mechanism diagnostics only, not standalone superiority evidence. Reward breakdown and derived exploration-efficiency diagnostics should be used to explain whether learner/update changes improve behavior efficiency and terminal quality. If all three candidates remain weaker than epsend004, pause learner/update single-direction search and consider a more systematic multi-run design or method_redesign_discussion_only rather than continuing unbounded serial trials.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline. This is not a global optimum, cross-seed conclusion, paper-level conclusion, or tuning completion claim.

evidence_boundary: posthoc_selection and supplemental_final_probe were skipped_train_side_only_tuning with missingness_treatment=not_missing and metrics_extracted=false. They are not evidence sources. Learner scalars are mechanism diagnostics only and cannot alone establish superiority.

redesign_boundary: This review authorizes only launcher-ready bounded learner/update comparison. It does not authorize method-level redesign, architecture changes, state representation changes, replay algorithm redesign, reward mechanism rewrite, action-space changes, environment semantics changes, source-code edits, or paper-level claims.
