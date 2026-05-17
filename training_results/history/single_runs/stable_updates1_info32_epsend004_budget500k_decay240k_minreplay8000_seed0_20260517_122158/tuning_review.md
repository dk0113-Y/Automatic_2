# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_122158

source_archive_id: stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_122158

recommendation_type: next_run_plan

prior_validation_status: partially_supported

validation_status: partially_supported

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0_20260517_122158

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: reward-function terminal-completion knob on top of updates1_info32 strong candidate

parameter_change: RewardTerminalBonus: 20 -> 22; keep RewardInfoScale=3.2 and LearnerUpdatesPerIter=1

recommended_next_run_name: stable_updates1_info32_terminal22_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: The current run validates updates1_info32 as a strong candidate but does not replace epsend004 as the current train-side reference baseline. It improves reward, information reward, episode length, repeat-visit behavior, turn burden, recent revisit, and learner diagnostics, but remains weaker than epsend004 on coverage, success_rate, timeout, zero_info, and stall. Because RewardInfoScale changed from 3.0 to 3.2, reward improvement is partially affected by reward-scale changes and should not override coverage/success/timeout deficits. The next bounded launcher-ready test should increase RewardTerminalBonus from 20 to 22 to target terminal completion and timeout behavior while preserving the updates1_info32 core settings.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: passed

note: strict_contract_ready supports reproducible launch status; posthoc/final_probe are skipped train-side-only status objects and are not performance evidence.

## 3. Prior Validation

prior_hypothesis: Combining LearnerUpdatesPerIter=1 with RewardInfoScale increased from 3.0 to 3.2 should improve information-gain drive, coverage, reward, and exploration efficiency while preserving the lower path burden observed in Candidate B.

validation_status: partially_supported

judgement: partially_supported. Reward, info_reward_sum, and coverage improved relative to Candidate B, but success_rate, timeout, episode_length, RVR, zero_info, stall, and turn burden did not fully preserve Candidate B's best efficiency behavior.

key_evidence:
- Current run endpoint reward 157.5278 and info_reward_sum 155.4997 exceed epsend004 and Candidate B.
- Coverage 0.93499 improves over Candidate B but remains below epsend004 0.94302.
- Success_rate 0.81 remains below epsend004 0.86 and Candidate B 0.85.
- Timeout 0.19 remains worse than epsend004 0.14 and Candidate B 0.15.
- Episode_length 362.52, RVR 0.1769, recent_revisit 26.56, and turn_ge_90 106.58 are better than epsend004, but the core completion metrics are not sufficient for baseline replacement.

remaining_uncertainty: Whether terminal-completion incentive can recover success_rate, terminal_bonus_sum, and timeout behavior without losing the reward, information-gain, and path-efficiency advantages of updates1_info32.

## 4. Current Evidence Digest

train_side_primary: current run is admissible train-side-only factual evidence with reproducible launch confirmed and factual summary ready.

mechanism_summary: LearnerUpdatesPerIter=1 reduces update pressure and supports path efficiency; RewardInfoScale=3.2 strengthens information-gain reward, but terminal success and timeout remain weaker than epsend004.

posthoc_context: skipped_train_side_only_tuning; status-only, not evidence.

final_probe_supplemental: skipped_train_side_only_tuning; status-only, not evidence.

main_limitation: mixed metrics: reward/path efficiency improved, but coverage/success/timeout do not justify replacing epsend004 reference baseline.

## 5. Next Action

target_uncertainty: whether increasing terminal completion reward can recover success_rate and timeout while retaining updates1_info32 reward and path-efficiency advantages.

next_hypothesis: RewardTerminalBonus 22 will increase successful terminal completion incentive and may improve success_rate, terminal_bonus_sum, and timeout without materially degrading reward, coverage, RVR, recent_revisit, or turn burden.

selected_next_surface: reward-function terminal-completion knob on top of updates1_info32 strong candidate

parameter_change: RewardTerminalBonus: 20 -> 22; keep RewardInfoScale=3.2 and LearnerUpdatesPerIter=1

rationale: the remaining deficits are success_rate, timeout, terminal_bonus_sum, zero_info, and stall; RewardTerminalBonus is the most direct launcher-ready knob for terminal completion and avoids repeating refuted directions such as EpsilonEnd=0.035, EpsilonDecaySteps=300000, RewardRevisitPenalty=0.12, RewardTurnPenaltyScale=0.06, LearningRate=0.000075, or BatchSize=256.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_updates1_info32_terminal22_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 22 -RewardInfoScale 3.2 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 1 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

expected_validation_focus: compare against retained epsend004 reference baseline and current updates1_info32 strong candidate; check endpoint reward, coverage, success_rate, timeout, episode_length, RVR, zero_info, recent_revisit, stall, turn_ge_90, turn_135, turn_180, terminal_bonus_sum, info_reward_sum, and learner diagnostics. Success_rate and timeout should improve without obvious loss of reward, information gain, and path-efficiency advantages.

## 6. Boundaries

reference_baseline_note: epsend004 remains the current single-seed train-side reference baseline; updates1_info32 is a strong candidate and next-run base, not the accepted reference baseline.

evidence_boundary: train-side monitoring is primary evidence; posthoc/final_probe are skipped status-only objects under train-side-only tuning.

redesign_boundary: no method-level redesign, no old near/mid/token mainline, no architecture/state/action-space/environment semantics changes.
