# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_revisit012_epsend004_budget500k_decay240k_minreplay8000_seed0_20260513_135851

source_archive_id: stable_revisit012_epsend004_budget500k_decay240k_minreplay8000_seed0_20260513_135851

recommendation_type: next_run_plan

prior_validation_status: refuted

recommended_next_run_name: stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: RewardRevisitPenalty=0.12 was refuted against the current train-side reference baseline stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904. The current run improved relative to the already-refuted epsdecay300k run but did not improve over epsend004. Keep epsend004 as the current train-side reference baseline and move from repeated-revisit penalty increase to light turn-shaping. The next bounded test should restore RewardRevisitPenalty=0.10 and increase RewardTurnPenaltyScale from 0.05 to 0.06 under the epsend004 core configuration.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The factual report satisfied the two direct admission gates. contract_verdict=strict_contract_ready was treated only as support for reproducible_launch_status, not as an additional gate. The run was legacy/eval-enabled with train_side_only_tuning=false and test_side_evaluation_status=enabled; train-side monitoring remained the primary decision evidence, while posthoc and final_probe remained non-primary contextual evidence.

## 3. Prior Validation

prior_hypothesis: Keep epsend004 core settings fixed and increase RewardRevisitPenalty from 0.10 to 0.12 to test whether stronger repeated-revisit shaping reduces RVR, recent_revisit, zero_info, stall, timeout, and turn burden while preserving coverage, success_rate, terminal_bonus, and endpoint reward.

validation_status: refuted

judgement: The current run did not achieve train-side baseline improvement over stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904. It should not replace the current train-side reference baseline.

key_evidence: Current revisit012 endpoint recent train window was reward 140.1382, coverage 0.9350, success_rate 0.75, episode_length 409.22, RVR 0.2161, timeout 0.25. The epsend004 reference baseline was reward 147.4969, coverage 0.9430, success_rate 0.86, episode_length 375.72, RVR 0.1873, timeout 0.14. Current revisit012 was weaker on all core endpoint metrics. Mechanism diagnostics also did not support the hypothesis: revisit012 had recent_revisit 50.22, stall 165.95, zero_info 228.50, turn_ge_90 167.21, turn_135 47.64, timeout 0.25, whereas epsend004 had recent_revisit 45.47, stall 144.03, zero_info 199.22, turn_ge_90 135.42, turn_135 31.12, timeout 0.14.

secondary_fact: revisit012 did repair several metrics relative to the already-refuted epsdecay300k run, including reward, success_rate, episode_length, RVR, and timeout. This does not change the formal validation result because the intended reference comparison was against epsend004.

remaining_uncertainty: Whether light turn shaping under the epsend004 core configuration can reduce turn burden, inefficient zig-zag behavior, repeated revisit, stall, zero_info, and timeout while preserving epsend004-level coverage, success_rate, reward, and terminal_bonus.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. revisit012 learned relative to its own initial phase and was not a failed run, but it did not improve over the current train-side reference baseline epsend004. The evidence does not support baseline update.

posthoc_context: Posthoc was present because this was a legacy/eval-enabled output. It was used only as checkpoint-selection and stability context. It did not establish superiority and did not override the train-side endpoint comparison.

final_probe_supplemental: final_probe was present because this was a legacy/eval-enabled output. It was used only as supplemental held-out diagnostic evidence. The final_probe context did not establish superiority and did not override the train-side endpoint comparison.

reward_breakdown_context: revisit012 had info_reward_sum 145.9421, step_penalty_sum -8.1844, recent_revisit_penalty_sum -6.0264, turn_penalty_sum -4.5932, timeout_penalty_sum -2.0, terminal_bonus 15.0. epsend004 had info_reward_sum 147.3457, step_penalty_sum -7.5144, recent_revisit_penalty_sum -4.547, turn_penalty_sum -3.8673, timeout_penalty_sum -1.12, terminal_bonus 17.2. The higher revisit penalty did not produce better behavioral efficiency and was associated with heavier revisit, turn, and timeout penalties plus lower terminal bonus.

learner_context: Learner scalars did not provide enough evidence to overturn the endpoint train-side conclusion. Optimizer scalars cannot replace train-side system metrics.

main_limitation: The run was legacy/eval-enabled, but the active decision remained train-side performance search. posthoc/final_probe context did not control the primary judgement.

## 5. Next Action

target_uncertainty: In the epsend004 core configuration, can light turn shaping reduce turn burden, repeated revisit induced inefficient paths, stall/zero_info, and timeout while preserving epsend004-level coverage, success_rate, endpoint reward, and terminal_bonus.

next_hypothesis: Restore RewardRevisitPenalty=0.10 and increase RewardTurnPenaltyScale from 0.05 to 0.06 while keeping epsend004 core settings fixed. This single-primary-parameter reward coefficient test may suppress zig-zag and turn-heavy behavior more directly than continuing to increase revisit penalty.

rationale: RewardRevisitPenalty=0.12 was refuted because it did not lower RVR, recent_revisit, stall, zero_info, timeout, or turn burden relative to epsend004. The stronger mechanism signal is turn burden coupled with timeout and terminal quality. RewardTurnPenaltyScale is launcher-ready and directly tied to turn diagnostics.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.06 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25

expected_validation_focus: Compare against stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 as the current train-side reference baseline. Focus on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, turn_ge_90, turn_135, turn_180, reward breakdown, turn_penalty, timeout_penalty, terminal_bonus, and whether turn_penalty reduction or behavior improvement is achieved without sacrificing info_reward, coverage, success_rate, or terminal_bonus. If RewardTurnPenaltyScale=0.06 is still weaker than epsend004, stop single reward-penalty increase direction and consider learner/update dynamics such as learning rate or target update cadence.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline. This is not a global optimum claim, not a cross-seed conclusion, not a paper-level superiority claim, and not a tuning completion claim.

final_probe_boundary: final_probe is supplemental held-out diagnostic evidence only and cannot alone establish superiority.

posthoc_boundary: posthoc winner or checkpoint-selection context cannot alone establish method or parameter superiority.

redesign_boundary: This is reward coefficient tuning within the current method. It does not authorize method-level redesign, architecture changes, state representation changes, replay redesign, action-space changes, or paper-level claims.