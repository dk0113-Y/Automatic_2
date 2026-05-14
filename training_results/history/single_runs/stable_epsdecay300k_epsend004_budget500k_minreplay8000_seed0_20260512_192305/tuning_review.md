# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_epsdecay300k_epsend004_budget500k_minreplay8000_seed0_20260512_192305

source_archive_id: stable_epsdecay300k_epsend004_budget500k_minreplay8000_seed0_20260512_192305

recommendation_type: next_run_plan

prior_validation_status: refuted

recommended_next_run_name: stable_revisit012_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: EpsilonDecaySteps=300000 was refuted against the current train-side reference baseline stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904. Keep epsend004 as the current train-side reference baseline and move from exploration-schedule tuning to reward revisit shaping. The next bounded test should keep epsend004 core settings and increase reward_revisit_penalty from 0.10 to 0.12.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The factual report satisfied the two direct admission gates. contract_verdict=strict_contract_ready was treated only as support for reproducible_launch_status, not as an additional gate.

## 3. Prior Validation

prior_hypothesis: Keep TotalEnvSteps=500000, EpsilonEnd=0.04, MinReplaySize=8000, FinalGreedyEpisodes=100, and Seed=0 fixed, then increase EpsilonDecaySteps from 240000 to 300000 to test whether longer mid-stage exploration improves endpoint train-side performance and reduces stability mismatch.

validation_status: refuted

judgement: The current run was weaker than epsend004 on all core train-side endpoint metrics and key diagnostics, so EpsilonDecaySteps=300000 should not replace the current reference baseline.

key_evidence: Current epsdecay300k endpoint metrics were reward 127.7547, coverage 0.9027, success_rate 0.56, episode_length 447.70, RVR 0.2874, timeout 0.44. The epsend004 reference baseline was reward 147.4969, coverage 0.9430, success_rate 0.86, episode_length 375.72, RVR 0.1873, timeout 0.14. Diagnostics also worsened: recent_revisit 65.92 vs 45.47, stall 219.67 vs 144.03, zero_info 277.09 vs 199.22, turn_ge_90 182.29 vs 135.42, turn_180 39.96 vs 32.75.

remaining_uncertainty: Whether reward-function tuning, especially repeated revisit shaping, can reduce RVR, recent revisit, zero_info, stall, timeout, and turn burden while preserving coverage, success_rate, and terminal_bonus under the epsend004 core configuration.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. The run learned relative to its own initial phase but was clearly worse than the epsend004 current reference baseline at endpoint. The evidence does not support baseline update. The strongest mechanism signal is repeated revisit, zero-info, stall, turn burden, and timeout degradation, with worse recent_revisit_penalty, turn_penalty, timeout_penalty, and lower terminal_bonus.

posthoc_context: Posthoc context showed checkpoint stability issues and suggested earlier checkpoint quality, but this was checkpoint-selection context only and did not establish method or parameter superiority.

final_probe_supplemental: final_probe was supplemental held-out diagnostic evidence only. It was directionally consistent with some recent train metrics but could not override the weaker train-side endpoint comparison against epsend004.

main_limitation: The run was legacy/eval-enabled and included posthoc/final_probe outputs, but the active decision remained train-side performance search. posthoc/final_probe mismatch or support did not control the primary conclusion.

## 5. Next Action

target_uncertainty: Whether increasing reward_revisit_penalty under the epsend004 core configuration can reduce repeated revisit behavior, RVR, zero_info, stall, timeout, and turn burden without sacrificing coverage, success_rate, terminal_bonus, or endpoint reward.

next_hypothesis: Increase reward_revisit_penalty from 0.10 to 0.12 while keeping epsend004 core settings fixed. This may discourage repeated revisit loops more directly than further exploration-schedule tuning.

rationale: EpsilonEnd=0.035 and EpsilonDecaySteps=300000 were both refuted. The remaining bottleneck is better aligned with reward-function knobs than with launcher convenience or continued schedule slicing. reward_revisit_penalty has the most direct evidence-to-mechanism relevance to RVR, recent_revisit, zero_info, stall, and repeated revisit penalty.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_revisit012_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.12

expected_validation_focus: Compare against stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 as the current train-side reference baseline. Focus on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, turn diagnostics, reward breakdown, terminal_bonus, recent_revisit_penalty, turn_penalty, and timeout_penalty. Confirm RVR and recent_revisit improve without coverage/success degradation.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline. This is not a global optimum claim, not a tuning completion claim, and not a paper-level conclusion.

final_probe_boundary: final_probe is supplemental held-out diagnostic evidence only and cannot alone establish superiority.

posthoc_boundary: posthoc winner or checkpoint-selection context cannot alone establish method or parameter superiority.

redesign_boundary: This is reward coefficient tuning within the current method. It does not authorize method-level redesign, architecture changes, state representation changes, replay redesign, action-space changes, or paper-level claims.
