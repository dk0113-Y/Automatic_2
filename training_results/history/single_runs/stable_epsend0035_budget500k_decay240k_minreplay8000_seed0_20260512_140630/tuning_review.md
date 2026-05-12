# GPT Tuning Review Digest

## 1. Summary

* source_run_name: stable_epsend0035_budget500k_decay240k_minreplay8000_seed0_20260512_140630
* source_archive_id: stable_epsend0035_budget500k_decay240k_minreplay8000_seed0_20260512_140630
* recommendation_type: next_run_plan
* prior_validation_status: refuted
* current_train_side_reference_baseline: stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904
* recommended_next_run_name: stable_epsdecay300k_epsend004_budget500k_minreplay8000_seed0
* decision_summary: The epsend0035 run refuted the epsilon_end=0.035 down-search hypothesis, keeps epsend004 as the current train-side reference baseline, and turns the next bounded action to EpsilonDecaySteps schedule-shape tuning.

## 2. Evidence Admission

* reproducible_launch_status: reproducible_launch_confirmed
* factual_summary_status: factual_summary_ready
* admission_result: passed
* note: reproducible_launch.contract_verdict == strict_contract_ready supports reproducible launch status and is not an additional admission gate.

## 3. Prior Validation

* prior_hypothesis: With TotalEnvSteps=500000, EpsilonDecaySteps=240000, MinReplaySize=8000, FinalGreedyEpisodes=100, and Seed=0 fixed, EpsilonEnd=0.035 may further reduce residual-exploration diagnostics while avoiding the endpoint degradation observed at epsilon_end=0.03.
* validation_status: refuted
* judgement: The epsend0035 run did not improve train-side system performance over the epsend004 current train-side reference baseline, so epsend0035 must not become the new train-side reference baseline.
* key_evidence:
  * Endpoint reward, coverage, success_rate, episode_length, RVR, and timeout were weaker than epsend004.
  * Stall, zero_info, recent_revisit, turn diagnostics, timeout penalty, revisit penalty, and turn penalty were worse than epsend004.
  * epsend0035 did not fully repeat the epsend003 collapse and its final_probe winner was the endpoint, but this does not override weaker primary train-side evidence.
  * EpsilonEnd down-search from 0.04 to 0.035 was refuted as a performance-improvement step.
* remaining_uncertainty: Whether preserving epsilon_end=0.04 while changing EpsilonDecaySteps can improve train-side system performance and stability without the endpoint regression seen under epsend0035.

## 4. Current Evidence Digest

* train_side_primary: Train-side endpoint and diagnostics show epsend0035 is weaker than the epsend004 current reference baseline, even though epsend0035 still completed effective learning and retained positive late-stage movement.
* posthoc_context: Posthoc and final_probe provide checkpoint-selection and stability context only; the endpoint final_probe winner in epsend0035 is useful but not standalone superiority evidence.
* final_probe_supplemental: final_probe is supplemental diagnostic evidence and cannot override the primary train-side comparison against epsend004.
* main_limitation: This is a single-seed formal stable run comparison and does not establish global optimality, tuning completion, or cross-seed robustness.

## 5. Next Action

* target_uncertainty: Whether changing EpsilonDecaySteps while keeping epsilon_end=0.04 can improve exploration schedule shape beyond the epsend004 reference setting.
* next_hypothesis: Keeping TotalEnvSteps=500000, EpsilonEnd=0.04, MinReplaySize=8000, FinalGreedyEpisodes=100, and Seed=0 fixed while increasing EpsilonDecaySteps from 240000 to 300000 may extend mid-training exploration, improve endpoint train-side performance, and reduce final_probe/posthoc stability mismatch without changing the terminal exploration floor.
* rationale: The EpsilonEnd dimension now has a local bracket in which 0.04 outperforms 0.05, 0.035, and 0.03, so further local cutting is lower priority. EpsilonDecaySteps is launcher-ready and directly tests schedule shape while preserving the current best terminal epsilon.
* command:
  .\scripts\launch_formal_train_stable.ps1 -RunName "stable_epsdecay300k_epsend004_budget500k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 300000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100
* expected_validation_focus:
  * Compare endpoint train-side reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, recent_revisit, and turn diagnostics against epsend004.
  * Verify that the run avoids the epsend0035 endpoint regression pattern.
  * Check late-stage delta for reward, coverage, success_rate, episode_length, RVR, and timeout.
  * Check posthoc stability, including whether the train-side candidate winner and formal final_probe winner remain endpoint-centered or near-endpoint.
  * Treat final_probe and posthoc as context or supplemental diagnostics, not standalone superiority evidence.

## 6. Boundaries

* reference_baseline_note: epsend004 remains the current train-side reference baseline, not a global optimum claim.
* final_probe_boundary: final_probe is supplemental and cannot alone establish superiority.
* posthoc_boundary: posthoc winner is checkpoint-selection context and cannot alone establish method superiority.
* redesign_boundary: No method-level redesign is authorized.
