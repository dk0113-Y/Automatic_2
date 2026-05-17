# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_updates1_info32_terminal22_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_165918

source_archive_id: stable_updates1_info32_terminal22_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_165918

recommendation_type: next_run_plan

prior_validation_status: partially_supported

validation_status: refuted

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_updates1_info32_terminal22_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_165918

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: reward-function timeout-completion knob on top of updates1_info32 strong candidate

parameter_change: RewardTimeoutPenalty: 8 -> 10; restore RewardTerminalBonus=20; keep RewardInfoScale=3.2 and LearnerUpdatesPerIter=1

recommended_next_run_name: stable_updates1_info32_timeout10_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: RewardTerminalBonus=22 was refuted because it increased terminal_bonus_sum locally but worsened success_rate, timeout, RVR, recent_revisit, stall, reward, coverage, and learner diagnostics relative to updates1_info32; keep epsend004 as the current train-side reference baseline and test RewardTimeoutPenalty 8 -> 10 while restoring RewardTerminalBonus=20 on the updates1_info32 strong-candidate core.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: passed

note: strict_contract_ready supports reproducible launch status but is not a third admission gate. The run is train-side-only, and posthoc/final_probe are skipped_train_side_only_tuning status-only objects with missingness_treatment=not_missing, not performance evidence.

## 3. Prior Validation

prior_hypothesis: Increasing RewardTerminalBonus from 20 to 22 on top of the updates1_info32 strong candidate should improve terminal completion, success_rate, terminal_bonus_sum, and timeout while preserving the reward, information-gain, and path-efficiency advantages of RewardInfoScale=3.2 and LearnerUpdatesPerIter=1.

validation_status: refuted

judgement: refuted. Terminal_bonus_sum improved from the updates1_info32 value, but the core terminal-completion and path-efficiency objectives moved in the wrong direction: success_rate decreased, timeout increased, and repeat-visit, stall, reward, coverage, and learner diagnostics all regressed relative to updates1_info32.

key_evidence:
- terminal22 terminal_bonus_sum 16.72 exceeded updates1_info32 terminal_bonus_sum 16.20, but success_rate fell from 0.81 to 0.76 and timeout rose from 0.19 to 0.24.
- terminal22 regressed versus updates1_info32 on reward 154.3775 vs 157.5278, coverage 0.9286 vs 0.9350, RVR 0.2061 vs 0.1769, recent_revisit 42.25 vs 26.56, and stall 164.59 vs 156.18.
- terminal22 remained weaker than epsend004 on coverage 0.9286 vs 0.9430, success_rate 0.76 vs 0.86, timeout 0.24 vs 0.14, RVR 0.2061 vs 0.1873, zero_info 206.40 vs 199.22, and stall 164.59 vs 144.03.
- terminal22 learner context regressed versus updates1_info32: loss 2.4887 vs 1.6389, td_abs_mean 2.8399 vs 1.9331, and grad_norm 30.1338 vs 23.9707.

remaining_uncertainty: Whether a more direct timeout-completion penalty can reduce timeout and improve terminal completion while preserving the updates1_info32 strong-candidate reward, information-gain, and path-efficiency advantages.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. The run is valid and learned over training, but terminal22 did not improve the intended terminal-completion outcomes and did not qualify as a reference baseline update.

mechanism_summary: RewardTerminalBonus=22 increased terminal_bonus_sum locally, but it did not directly suppress timeout or inefficient exploration. It coincided with higher timeout, higher RVR, higher recent_revisit, higher stall, lower reward, lower coverage, and weaker learner diagnostics relative to updates1_info32.

posthoc_context: posthoc_selection was skipped_train_side_only_tuning with missingness_treatment=not_missing. It is status-only and not evidence.

final_probe_supplemental: supplemental_final_probe was skipped_train_side_only_tuning with missingness_treatment=not_missing. It is status-only and not evidence.

main_limitation: This is a single-seed train-side reference decision only. It does not establish global optimum, cross-seed robustness, paper-level superiority, method superiority, or tuning completion.

## 5. Next Action

target_uncertainty: Whether increasing RewardTimeoutPenalty from 8 to 10 while restoring RewardTerminalBonus=20 can directly reduce timeout and improve terminal completion without losing updates1_info32 reward, information-gain, and path-efficiency advantages.

next_hypothesis: Restoring RewardTerminalBonus=20 and increasing RewardTimeoutPenalty from 8 to 10 may penalize timeout failure more directly than terminal bonus scaling, improving timeout and success_rate while preserving RewardInfoScale=3.2 and LearnerUpdatesPerIter=1 benefits.

selected_next_surface: reward-function timeout-completion knob on top of updates1_info32 strong candidate

parameter_change: RewardTimeoutPenalty: 8 -> 10; restore RewardTerminalBonus=20; keep RewardInfoScale=3.2 and LearnerUpdatesPerIter=1

rationale: Terminal22 showed that increasing RewardTerminalBonus to 22 is not an effective fix for success_rate and timeout. The remaining deficit is timeout/completion quality, and RewardTimeoutPenalty is the most direct launcher-ready knob for that mechanism. This avoids repeating historically refuted directions such as EpsilonEnd=0.035, EpsilonDecaySteps=300000, RewardRevisitPenalty=0.12, RewardTurnPenaltyScale=0.06, LearningRate=0.000075, and BatchSize=256.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_updates1_info32_timeout10_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 10 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3.2 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 1 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

expected_validation_focus: compare against retained epsend004 reference baseline and updates1_info32 strong candidate; check endpoint success_rate, timeout, terminal_bonus_sum, reward, coverage, episode_length, RVR, zero_info, recent_revisit, stall, turn_ge_90, turn_135, turn_180, timeout_penalty_sum, info_reward_sum, td_abs_mean, and grad_norm. The run should reduce timeout and improve success_rate relative to updates1_info32 without repeating terminal22 regression in reward, RVR, recent_revisit, stall, and learner stability.

## 6. Boundaries

reference_baseline_note: epsend004 remains the current single-seed train-side reference baseline. updates1_info32 remains a strong candidate and mechanism base, while terminal22 does not update the reference baseline.

evidence_boundary: Train-side monitoring is primary evidence. posthoc/final_probe are skipped status-only objects under train-side-only tuning and cannot establish superiority or evidence insufficiency.

redesign_boundary: This recommendation is launcher-ready reward-coefficient tuning within the current stable method. It does not authorize method-level redesign, old near/mid/token mainline, architecture changes, state representation changes, action-space changes, environment-semantics changes, or paper-level claims.
