# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0_20260514_182435

source_archive_id: stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0_20260514_182435

recommendation_type: next_run_plan

prior_validation_status: refuted

validation_status: refuted

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0_20260514_182435

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: learner/replay dynamics

parameter_change: LearningRate: 0.0001 -> 0.000075

recommended_next_run_name: stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: RewardTurnPenaltyScale=0.06 was refuted against the current train-side reference baseline stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904; keep epsend004 as the current single-seed train-side reference baseline and test a bounded learner-dynamics step by reducing LearningRate from 0.0001 to 0.000075.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The factual report satisfied the two direct evidence gates. reproducible_launch.contract_verdict == strict_contract_ready was treated only as support for reproducible_launch_status, not as a third gate. The run was train-side-only with posthoc_selection and supplemental_final_probe skipped_train_side_only_tuning and not_missing, so posthoc/final_probe were not evidence sources.

## 3. Prior Validation

prior_hypothesis: Restore RewardRevisitPenalty=0.10 and increase RewardTurnPenaltyScale from 0.05 to 0.06 while keeping epsend004 core settings fixed, testing whether light turn shaping can reduce turn burden, repeated-revisit-induced inefficient paths, stall, zero_info, and timeout while preserving epsend004-level coverage, success_rate, endpoint reward, and terminal_bonus.

validation_status: refuted

judgement: The turn006 run showed limited local improvement over the already-refuted revisit012 run in some turn-related diagnostics, but it did not improve over the current train-side reference baseline epsend004 and must not replace the baseline.

key_evidence:
- turn006 endpoint train-side metrics were weaker than epsend004: reward 134.4826 vs 147.4969, coverage 0.9130 vs 0.9430, success_rate 0.68 vs 0.86, episode_length 397.70 vs 375.72, RVR 0.2269 vs 0.1873, timeout 0.32 vs 0.14.
- turn006 key diagnostics remained weaker than epsend004: zero_info 226.66 vs 199.22, recent_revisit 55.17 vs 45.47, stall 169.31 vs 144.03, turn_ge_90 146.76 vs 135.42, turn_135 37.04 vs 31.12, turn_180 34.47 vs 32.75.
- turn006 had positive within-run learning and late-stage movement, but this did not translate into reference-baseline improvement.
- posthoc_selection and supplemental_final_probe were skipped by train_side_only_tuning and were status-only test-side objects, not evidence sources.

remaining_uncertainty: Whether restoring epsend004 reward/exploration core settings and adjusting learner dynamics, specifically reducing LearningRate from 0.0001 to 0.000075, can improve endpoint stability and train-side system performance without further reward-penalty increases.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring is the primary evidence. turn006 learned relative to its own initial phase but remained weaker than epsend004 on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, and most key diagnostics, so the current reference baseline remains unchanged.

mechanism_summary: Light turn shaping reduced some local turn-related burden relative to the refuted revisit012 run, but it did not reduce total behavior inefficiency enough to beat epsend004. Reward breakdown remained weaker than epsend004, with lower information reward and terminal bonus and heavier revisit, turn, and timeout penalties. Learner context also pointed to higher loss, td_abs_mean, and grad_norm than epsend004, supporting a shift toward learner/replay dynamics rather than further reward penalty increments.

posthoc_context: posthoc_selection.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as checkpoint-selection evidence or superiority evidence.

final_probe_supplemental: supplemental_final_probe.status was skipped_train_side_only_tuning with missingness_treatment=not_missing. It was a status-only test-side object and not used as supplemental evidence for this train-side-only run.

main_limitation: This is a single-seed train-side reference decision only. It does not establish global optimum, cross-seed robustness, paper-level superiority, or tuning completion.

## 5. Next Action

target_uncertainty: Under epsend004 core settings, can a more conservative learner update step reduce TD/gradient update noise and improve endpoint stability, behavior efficiency, and train-side performance without sacrificing late-stage learning progress.

next_hypothesis: Restore epsend004 reward and exploration core settings and reduce LearningRate from 0.0001 to 0.000075. A slightly lower learning rate may stabilize Q updates, reduce endpoint instability, and improve train-side endpoint reward, coverage, success_rate, RVR, timeout, stall, zero_info, and terminal quality.

selected_next_surface: learner/replay dynamics

parameter_change: LearningRate: 0.0001 -> 0.000075

rationale: EpsilonEnd=0.035, EpsilonDecaySteps=300000, RewardRevisitPenalty=0.12, and RewardTurnPenaltyScale=0.06 have all failed to update the epsend004 reference baseline. The strongest remaining bounded direction is launcher-ready learner dynamics. Reducing LearningRate is a single-primary-parameter test that preserves epsend004 reward/exploration core while addressing the observed learner context and endpoint instability.

command: .\scripts\launch_formal_train_stable.ps1 -RunName "stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.000075 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 2 -TrainEveryEnvSteps 16

expected_validation_focus:
- Compare against stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 as the current train-side reference baseline.
- Focus on endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, turn_ge_90, turn_135, and turn_180.
- Check learner context: loss, td_abs_mean, and grad_norm should become more stable, but learner scalars must not be used as standalone superiority evidence.
- Check reward breakdown: info_reward_sum, recent_revisit_penalty_sum, turn_penalty_sum, timeout_penalty_sum, and terminal_bonus_sum should approach or exceed epsend004 quality.
- Check derived exploration-efficiency diagnostics as diagnostic-only evidence: coverage_gain_per_step, weighted_info_gain_per_step, zero_info_rate, recent_revisit_rate, stall_rate, turn_burden_rate, and timeout_rate.
- If LearningRate=0.000075 is still weaker than epsend004, stop reward-penalty and single learning-rate one-off attempts and consider bounded multi-run comparison or rollout/update cadence surface.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline; this is not a global optimum claim, cross-seed conclusion, paper-level superiority claim, or tuning completion claim.

evidence_boundary: Train-side monitoring is the primary evidence. posthoc_selection and supplemental_final_probe were skipped_train_side_only_tuning status-only objects and cannot establish superiority or evidence insufficiency.

redesign_boundary: This recommendation is launcher-ready learner-parameter tuning within the current method. It does not authorize method-level redesign, architecture changes, state representation changes, replay algorithm redesign, reward mechanism rewrite, action-space changes, environment-semantics changes, or paper-level claims.
