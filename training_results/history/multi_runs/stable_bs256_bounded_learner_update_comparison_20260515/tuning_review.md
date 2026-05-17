# GPT Tuning Review Digest

## 1. Summary

source_run_name: stable_bs256_bounded_learner_update_comparison_20260515

source_archive_id: stable_bs256_bounded_learner_update_comparison_20260515

recommendation_type: next_run_plan

prior_validation_status: partially_supported

validation_status: partially_supported

baseline_update_status: unchanged

current_train_side_reference_baseline_before: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_candidate: stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0_20260516_162932

current_train_side_reference_baseline_after: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904

baseline_scope: single_seed_train_side_reference_only

selected_next_surface: reward-function information-gain knob on top of validated lower update-to-data ratio

parameter_change: RewardInfoScale: 3.0 -> 3.2; keep LearnerUpdatesPerIter=1

recommended_next_run_name: stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0

decision_summary: The bounded multi-run comparison was partially supported. Candidate B with LearnerUpdatesPerIter=1 was the strongest mechanism candidate and improved reward, episode_length, repeat_visit_ratio, zero_info, recent_revisit, stall, turn burden, and learner scalars relative to the other candidates and the bs256 trigger. However, Candidate B still did not clearly exceed epsend004 on coverage, success_rate, and timeout, so the reference baseline remains epsend004. The next bounded single run should keep LearnerUpdatesPerIter=1 and lightly increase RewardInfoScale from 3.0 to 3.2 to recover coverage and information gain while preserving B's behavior-efficiency gains.

## 2. Evidence Admission

reproducible_launch_status: reproducible_launch_confirmed

factual_summary_status: factual_summary_ready

admission_result: admitted

note: The current input was current_multi_run_analysis.json with report_type=multi_run_factual_comparison and factual_summary_status=factual_summary_ready. All three candidates had reproducible_launch_status=reproducible_launch_confirmed and contract_verdict=strict_contract_ready. strict_contract_ready was treated only as support for reproducible_launch_status, not as a third gate. All three candidates were train-side-only runs with posthoc_selection and supplemental_final_probe skipped_train_side_only_tuning, missingness_treatment=not_missing, and no test-side metrics extracted.

## 3. Prior Validation

prior_hypothesis: Under epsend004 core settings, compare intermediate batch size, lower update-to-data ratio, and slower target sync cadence to determine whether learner/update dynamics contain a useful local direction that transfers learner stability into endpoint behavior efficiency and train-side system performance.

validation_status: partially_supported

judgement: The bounded comparison identified LearnerUpdatesPerIter=1 as the strongest learner/update sub-direction. Candidate B was best on reward, success_rate among candidates, episode_length, repeat_visit_ratio, timeout among candidates, zero_info, recent_revisit, stall, turn diagnostics, and learner scalars. Relative to epsend004, B improved reward, episode_length, repeat_visit_ratio, zero_info, recent_revisit, stall, turn_ge_90, turn_135, and turn_180, but did not clearly exceed epsend004 on coverage, success_rate, and timeout. Therefore the learner/update hypothesis is partially supported, but baseline update remains unchanged.

key_evidence:
- Candidate B reward 147.9161 was above epsend004 reward 147.4969 and above the other candidates.
- Candidate B episode_length 331.76 was shorter than epsend004 375.72.
- Candidate B repeat_visit_ratio 0.1641 was lower than epsend004 0.1873.
- Candidate B zero_info, recent_revisit, stall, turn_ge_90, turn_135, and turn_180 were all better than epsend004 and the other candidates.
- Candidate B learner scalars improved: loss 1.7958, td_abs_mean 2.0977, grad_norm 24.2916.
- Candidate B coverage 0.9214 remained below epsend004 coverage 0.9430.
- Candidate B success_rate 0.85 remained slightly below epsend004 success_rate 0.86.
- Candidate B timeout 0.15 remained slightly above epsend004 timeout 0.14.
- Candidate C had the strongest coverage among candidates at 0.9420 but weaker system efficiency than B.
- Candidate A improved over bs256 in places but remained weaker than B and epsend004.

remaining_uncertainty: Whether RewardInfoScale can restore coverage, information gain, and success_rate on top of the efficient LearnerUpdatesPerIter=1 structure without reintroducing high revisit, stall, turn burden, or timeout.

## 4. Current Evidence Digest

train_side_primary: Train-side monitoring was the primary evidence. The multi-run factual comparison was factual-only and did not decide a winner, baseline update, or next hyperparameters. GPT interpreted the comparison and found Candidate B strongest, but not sufficient to update the epsend004 baseline.

mechanism_summary: Lowering LearnerUpdatesPerIter from 2 to 1 reduced update pressure and improved behavior efficiency, including shorter episodes, lower repeat visits, lower zero-info behavior, lower stall, lower turn burden, and better learner scalars. The remaining weakness is information-gain and coverage side: B's coverage and info_reward_sum lag epsend004, while C retained stronger coverage but weaker efficiency. The next mechanism test should keep the lower update-to-data ratio and lightly increase information reward drive.

posthoc_context: posthoc_selection was skipped_train_side_only_tuning for all candidates with missingness_treatment=not_missing. It was a status-only test-side object and not used as checkpoint-selection evidence or superiority evidence.

final_probe_supplemental: supplemental_final_probe was skipped_train_side_only_tuning for all candidates with missingness_treatment=not_missing. It was a status-only test-side object and not used as held-out evidence.

main_limitation: This is a single-seed train-side reference decision. It does not establish global optimum, cross-seed robustness, paper-level superiority, or tuning completion.

## 5. Next Action

target_uncertainty: Whether light information-gain reward strengthening can restore coverage, info_reward_sum, and success_rate while preserving the efficiency gains of LearnerUpdatesPerIter=1.

next_hypothesis: Keep epsend004 core settings and Candidate B's LearnerUpdatesPerIter=1, then increase RewardInfoScale from 3.0 to 3.2. This may recover coverage and information-gain drive while preserving low repeat-visit ratio, low turn burden, low stall, low zero_info, and low timeout.

selected_next_surface: reward-function information-gain knob on top of validated lower update-to-data ratio

parameter_change: RewardInfoScale: 3.0 -> 3.2; keep LearnerUpdatesPerIter=1

rationale: Candidate B demonstrated that lower update-to-data ratio is the best learner/update mechanism found so far, but it still underperformed epsend004 on coverage, success_rate, timeout, and info_reward_sum. RewardInfoScale has not been refuted in the current chain and directly targets the residual information-gain deficit. This is more evidence-aligned than continuing batch-size, target-sync, exploration schedule, or already-refuted revisit/turn penalty directions.

command:
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100 -RewardRevisitPenalty 0.10 -RewardTurnPenaltyScale 0.05 -RewardTimeoutPenalty 8 -RewardStepPenalty 0.02 -RewardTerminalBonus 20 -RewardInfoScale 3.2 -RewardObstacleWeight 0.25 -BatchSize 128 -ReplayCapacity 100000 -NStep 3 -Gamma 0.99 -LearningRate 0.0001 -TargetUpdateInterval 1000 -GradClipNorm 10 -CollectStepsPerIter 16 -LearnerUpdatesPerIter 1 -TrainEveryEnvSteps 16 -TrainSideOnlyTuning:$true

expected_validation_focus: Compare the next run against epsend004 and Candidate B. Primary metrics are endpoint reward, coverage, success_rate, episode_length, repeat_visit_ratio, timeout, zero_info, recent_revisit, stall, turn_ge_90, turn_135, and turn_180. Verify that RewardInfoScale=3.2 improves coverage, info_reward_sum, and success_rate without regressing B's efficiency gains. Learner scalars and derived exploration-efficiency diagnostics remain mechanism diagnostics only and not standalone superiority evidence.

## 6. Boundaries

reference_baseline_note: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904 remains the current single-seed train-side reference baseline. This is not a global optimum, cross-seed conclusion, paper-level conclusion, or tuning completion claim.

evidence_boundary: factual_metric_leaders from current_multi_run_analysis.json were descriptive_only_not_tuning_decision. Posthoc_selection and supplemental_final_probe were skipped_train_side_only_tuning with missingness_treatment=not_missing and metrics_extracted=false for all candidates. Learner scalars and derived exploration-efficiency metrics are mechanism diagnostics only and cannot alone establish superiority.

redesign_boundary: This review authorizes only one launcher-ready next_run_plan using an exposed reward knob and exposed rollout/update parameter. It does not authorize method-level redesign, architecture changes, state representation changes, replay algorithm redesign, reward mechanism rewrite, action-space changes, environment semantics changes, source-code edits, or paper-level claims.
