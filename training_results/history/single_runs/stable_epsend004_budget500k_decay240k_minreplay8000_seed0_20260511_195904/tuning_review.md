
# GPT Tuning Review Digest

## 1. Summary

* source_run_name: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904
* source_archive_id: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904
* recommendation_type: next_run_plan
* prior_validation_status: supported
* current_train_side_reference_baseline: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904
* recommended_next_run_name: stable_epsend0035_budget500k_decay240k_minreplay8000_seed0
* decision_summary: The epsend004 run supports the prior epsilon_end=0.04 hypothesis, updates epsend004 as the current train-side reference baseline, and recommends a bounded epsilon_end=0.035 single-variable next run.

## 2. Evidence Admission

* reproducible_launch_status: reproducible_launch_confirmed
* factual_summary_status: factual_summary_ready
* admission_result: passed
* note: reproducible_launch.contract_verdict == strict_contract_ready supports reproducible launch status and is not an additional admission gate.

## 3. Prior Validation

* prior_hypothesis: At 500k steps with epsilon_decay_steps=240k and min_replay_size=8000 fixed, epsilon_end=0.04 may preserve more late-stage training stability than 0.03 while reducing some residual exploration pressure relative to 0.05.
* validation_status: supported
* judgement: The epsend004 run improved train-side endpoint performance over both the epsend003 negative-control run and the previous 500k epsilon_end=0.05 reference baseline.
* key_evidence:

  * Endpoint reward, coverage, success_rate, episode_length, RVR, and timeout improved relative to the previous 500k epsilon_end=0.05 reference baseline.
  * Late-stage delta was positive for reward, coverage, success_rate, episode_length, RVR, and timeout.
  * Key diagnostics such as stall, zero_info, and timeout improved relative to epsend003 and were competitive with or better than the prior reference baseline.
  * final_probe/posthoc mismatch remains a limitation or validation focus, not a blocker for baseline update or next_run_plan.
* remaining_uncertainty: Whether epsilon_end=0.035 can further improve train-side system performance without repeating the endpoint degradation observed under epsilon_end=0.03.

## 4. Current Evidence Digest

* train_side_primary: Train-side endpoint metrics and late-stage trend support upgrading epsend004 to the current train-side reference baseline under the active single-seed performance search interface.
* posthoc_context: Train-side posthoc candidate scoring selected the endpoint as rank 1, while supplemental final_probe selected 480k, so checkpoint-selection and held-out mismatch should be tracked as stability context.
* final_probe_supplemental: final_probe is supplemental diagnostic evidence and does not override the train-side baseline update.
* main_limitation: This is a single-seed formal stable run comparison and does not establish global optimality, tuning completion, or cross-seed robustness.

## 5. Next Action

* target_uncertainty: Whether epsilon_end=0.035 is a better local residual-exploration setting than the current epsilon_end=0.04 reference baseline.
* next_hypothesis: With TotalEnvSteps=500000, EpsilonDecaySteps=240000, MinReplaySize=8000, FinalGreedyEpisodes=100, and Seed=0 fixed, EpsilonEnd=0.035 may further reduce residual-exploration diagnostics while avoiding the endpoint degradation observed at epsilon_end=0.03.
* rationale: The 650k budget extension and epsilon_end=0.03 direction have been refuted, while epsilon_end=0.04 is now the current train-side reference baseline. Testing epsilon_end=0.035 is a bounded launcher-ready local search action on the same tuning dimension.
* command:

```powershell
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_epsend0035_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.035 -MinReplaySize 8000 -FinalGreedyEpisodes 100
```

* expected_validation_focus:

  * Compare endpoint train-side reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, and turn diagnostics against epsend004.
  * Verify that epsilon_end=0.035 does not repeat the endpoint degradation observed under epsend003.
  * Check whether posthoc train-side candidate selection remains endpoint-centered or regresses to earlier-checkpoint dependence.
  * Treat final_probe as supplemental held-out diagnostic evidence only.
  * Preserve epsend004 as the current train-side reference baseline unless the next run improves the primary train-side evidence.

## 6. Boundaries

* reference_baseline_note: epsend004 is the current train-side reference baseline, not a global optimum claim.
* final_probe_boundary: final_probe is supplemental and cannot alone establish superiority.
* posthoc_boundary: posthoc winner is checkpoint-selection context and cannot alone establish method superiority.
* redesign_boundary: No method-level redesign is authorized.
