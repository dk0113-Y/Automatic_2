# GPT Tuning Review Digest

## 1. Summary
- source_run_name: stable_epsend003_decay240k_minreplay8000_seed0_20260510_220342
- source_archive_id: stable_epsend003_decay240k_minreplay8000_seed0_20260510_220342
- recommendation_type: next_run_plan
- prior_validation_status: refuted
- recommended_next_run_name: stable_epsend004_budget500k_decay240k_minreplay8000_seed0
- decision_summary: The epsilon_end=0.03 run refuted the prior residual-exploration reduction hypothesis, so the next bounded tuning action is a single-variable epsilon_end=0.04 validation at 500k steps.

## 2. Evidence Admission
- reproducible_launch_status: reproducible_launch_confirmed
- factual_summary_status: factual_summary_ready
- admission_result: passed
- note: reproducible_launch.contract_verdict == strict_contract_ready supports reproducible launch status and is not an additional admission gate.

## 3. Prior Validation
- prior_hypothesis: Lowering epsilon_end from 0.05 to 0.03 at 500k steps would reduce late-stage residual-exploration inefficiency without sacrificing coverage or success.
- validation_status: refuted
- judgement: The train-side endpoint evidence contradicts the prior hypothesis because epsilon_end=0.03 worsened reward, coverage, success_rate, episode_length, RVR, timeout, stall, and zero_info relative to the 500k epsilon_end=0.05 reference baseline.
- key_evidence:
  - Endpoint success_rate fell from 0.81 to 0.60, while timeout rose from 0.19 to 0.40.
  - Endpoint reward and coverage were lower than the 500k epsilon_end=0.05 reference baseline.
  - RVR, stall, zero_info, and recent_revisit increased instead of improving.
  - The 480k posthoc/final_probe signal is useful context but cannot override weaker train-side endpoint evidence.
- remaining_uncertainty: Whether an intermediate epsilon_end value between 0.05 and 0.03 can retain endpoint stability while reducing residual exploration pressure.

## 4. Current Evidence Digest
- train_side_primary: The epsilon_end=0.03 run completed learning but failed to improve endpoint train-side performance over the 500k epsilon_end=0.05 reference baseline. The main endpoint and diagnostic directions consistently indicate degradation rather than improvement.
- posthoc_context: The best checkpoint shifted to 480k, suggesting checkpoint-selection dependence and weaker endpoint stability under epsilon_end=0.03.
- final_probe_supplemental: The 480k final_probe result shows partial held-out efficiency signal but remains supplemental and cannot establish superiority alone.
- main_limitation: This is a single-seed formal comparison, so the result supports bounded tuning direction but not global optimality.

## 5. Next Action
- target_uncertainty: Whether epsilon_end=0.04 is a better intermediate residual-exploration setting than the tested 0.05 and 0.03 endpoints.
- next_hypothesis: At 500k steps with epsilon_decay_steps=240k and min_replay_size=8000 fixed, epsilon_end=0.04 may preserve more late-stage training stability than 0.03 while reducing some residual exploration pressure relative to 0.05.
- rationale: The 650k budget extension and aggressive epsilon_end=0.03 reduction have both been refuted, leaving intermediate residual exploration calibration as the next bounded single-variable uncertainty. Testing epsilon_end=0.04 minimally perturbs the active configuration while directly addressing the remaining uncertainty.
- command:
```powershell
.\scripts\launch_formal_train_stable.ps1 -RunName "stable_epsend004_budget500k_decay240k_minreplay8000_seed0" -Device "cuda" -Seed 0 -TotalEnvSteps 500000 -EpsilonDecaySteps 240000 -EpsilonEnd 0.04 -MinReplaySize 8000 -FinalGreedyEpisodes 100
```
- expected_validation_focus:
  - Compare train-side endpoint reward, coverage, success_rate, episode_length, RVR, timeout, stall, zero_info, and recent_revisit against the 500k epsilon_end=0.05 reference baseline and the epsilon_end=0.03 negative-control run.
  - Check whether posthoc winner returns to endpoint or remains clearly earlier than endpoint.
  - Treat final_probe as supplemental held-out validation, not as standalone superiority evidence.
  - Verify that epsilon_end=0.04 does not repeat the endpoint degradation observed under epsilon_end=0.03.

## 6. Boundaries
- reference_baseline_note: The 500k epsilon_end=0.05 baseline remains the current reference baseline, not a global optimum claim.
- final_probe_boundary: final_probe is supplemental and cannot alone establish superiority.
- posthoc_boundary: posthoc winner is checkpoint-selection context and cannot alone establish method superiority.
- redesign_boundary: This is tuning-surface validation only and does not authorize method-level redesign.
