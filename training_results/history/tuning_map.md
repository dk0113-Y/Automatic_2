# Tuning Map

## 1. Scope

This map is a compact navigation summary for GPT tuning review. It is based on `training_results/history/history_index.json` and archived GPT tuning reviews already present in history.

It is not decision authority. GPT still decides from the current factual report, matched prior review, this map, and `docs/training_system_manifest.md`.

Limits:
- No global optimum claim.
- No paper-level superiority claim.
- No cross-seed superiority claim.
- No raw artifact interpretation.
- No new tuning decisions.

## 2. Current Train-Side Reference Baseline

| Field | Value |
| --- | --- |
| current_train_side_reference_baseline | `stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904` |
| basis | Archived GPT review for `stable_epsend004...` recorded `prior_validation_status: supported` and updated epsend004 as the current train-side reference baseline. Later archived GPT reviews retained epsend004 after refuting epsend0035, epsdecay300k, revisit012, turn006, and lr75e5, after partially supporting bs256 without a baseline update, and after partially supporting the bounded learner-update comparison without a baseline update. |
| scope | `single_seed_train_side_reference_only` |
| caveat | Current reference baseline is not a global optimum, tuning completion claim, paper-level conclusion, or cross-seed conclusion. |

## 3. Active Tuning Branch

| Field | Value |
| --- | --- |
| branch objective | Single-seed train-side performance search under the formal stable run setting. |
| primary evidence | Train-side endpoint metrics and diagnostics. |
| context evidence | posthoc selection as checkpoint-selection/stability context; `final_probe` as supplemental held-out diagnostic context. |
| retained core reference settings | `TotalEnvSteps=500000`, `EpsilonDecaySteps=240000`, `EpsilonEnd=0.04`, `MinReplaySize=8000`, `Seed=0`, `FinalGreedyEpisodes=100` as represented by epsend004 archived review. |

## 4. Tuning Path Summary

| Archived run | Prior validation | GPT baseline outcome | GPT next direction |
| --- | --- | --- | --- |
| `stable_baseline_decay240k_minreplay8000_seed0_20260508_090552` | `not_applicable` | First paired archive record; baseline-to-budget-extension chain started. | Test `TotalEnvSteps=650000`. |
| `stable_extend650k_decay240k_minreplay8000_seed0_20260510_150603` | `refuted` | 500k epsilon_end=0.05 reference retained. | Test `EpsilonEnd=0.03`. |
| `stable_epsend003_decay240k_minreplay8000_seed0_20260510_220342` | `refuted` | 500k epsilon_end=0.05 reference retained. | Test `EpsilonEnd=0.04`. |
| `stable_epsend004_budget500k_decay240k_minreplay8000_seed0_20260511_195904` | `supported` | epsend004 became the current train-side reference baseline. | Test `EpsilonEnd=0.035`. |
| `stable_epsend0035_budget500k_decay240k_minreplay8000_seed0_20260512_140630` | `refuted` | epsend004 retained. | Test `EpsilonDecaySteps=300000` with `EpsilonEnd=0.04`. |
| `stable_epsdecay300k_epsend004_budget500k_minreplay8000_seed0_20260512_192305` | `refuted` | epsend004 retained. | Test `RewardRevisitPenalty=0.12`. |
| `stable_revisit012_epsend004_budget500k_decay240k_minreplay8000_seed0_20260513_135851` | `refuted` | epsend004 retained. | Recommended next run name: `stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| `stable_turn006_epsend004_budget500k_decay240k_minreplay8000_seed0_20260514_182435` | `refuted` | epsend004 retained. | Recommended next run name: `stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| `stable_lr75e5_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_103843` | `refuted` | epsend004 retained. | Recommended next run name: `stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| `stable_bs256_epsend004_budget500k_decay240k_minreplay8000_seed0_20260515_175304` | `partially_supported` | epsend004 retained. | Recommended next run names: `stable_bs192_epsend004_budget500k_decay240k_minreplay8000_seed0`; `stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0`; `stable_target2000_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| `stable_bs256_bounded_learner_update_comparison_20260515` | `partially_supported` | epsend004 retained. | Recommended next run name: `stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0`. |

## 5. Refuted Directions

| Direction | Archived GPT conclusion |
| --- | --- |
| Budget extension to 650k under the original epsilon_end=0.05 core | Refuted by `stable_extend650k...`; 500k reference retained. |
| Aggressive epsilon_end reduction to 0.03 | Refuted by `stable_epsend003...`; endpoint train-side performance worsened versus the 500k epsilon_end=0.05 reference. |
| EpsilonEnd down-search to 0.035 after epsend004 | Refuted by `stable_epsend0035...`; epsend004 retained. |
| EpsilonDecaySteps increase to 300000 under epsend004 core | Refuted by `stable_epsdecay300k...`; epsend004 retained. |
| RewardRevisitPenalty increase to 0.12 under epsend004 core | Refuted by `stable_revisit012...`; epsend004 retained. |
| RewardTurnPenaltyScale increase to 0.06 under epsend004 core | Refuted by `stable_turn006...`; epsend004 retained. |
| LearningRate reduction to 0.000075 under epsend004 core | Refuted by `stable_lr75e5...`; epsend004 retained. |

Notes:
- Exploration schedule slicing should not remain a primary direction unless new current evidence supports revisiting it.
- `RewardRevisitPenalty=0.12` should not remain a primary direction unless new current evidence supports revisiting it.
- `RewardTurnPenaltyScale=0.06` should not remain a primary direction unless new current evidence supports revisiting it.
- `LearningRate=0.000075` should not remain a primary direction unless new current evidence supports revisiting it.
- Improvement over an already-refuted run does not update the current train-side reference baseline unless it also improves over epsend004.

## 6. Supported / Retained Directions

| Direction | Archived GPT conclusion |
| --- | --- |
| `EpsilonEnd=0.04` at 500k with decay 240k and min replay 8000 | Supported by `stable_epsend004...`; became the current train-side reference baseline. |
| Keep epsend004 as reference after epsend0035 | Retained by archived GPT review for `stable_epsend0035...`. |
| Keep epsend004 as reference after epsdecay300k | Retained by archived GPT review for `stable_epsdecay300k...`. |
| Keep epsend004 as reference after revisit012 | Retained by archived GPT review for `stable_revisit012...`. |
| Keep epsend004 as reference after turn006 | Retained by archived GPT review for `stable_turn006...`. |
| Keep epsend004 as reference after lr75e5 | Retained by archived GPT review for `stable_lr75e5...`. |
| Keep epsend004 as reference after bs256 | Retained by archived GPT review for `stable_bs256...`. |
| Keep epsend004 as reference after bounded learner-update comparison | Retained by archived GPT review for `stable_bs256_bounded_learner_update_comparison_20260515`; Candidate B / `LearnerUpdatesPerIter=1` was the strongest mechanism candidate but did not update the baseline. |

## 7. Open Uncertainties

| Uncertainty | Status |
| --- | --- |
| Whether light turn shaping can reduce turn burden, repeated revisit induced inefficient paths, stall, zero_info, and timeout while preserving epsend004-level train-side endpoint performance. | Open in archived GPT review after revisit012. |
| Whether a bounded learner/update comparison can identify a local region where learner stability transfers to behavior efficiency: intermediate batch size, lower update-to-data ratio, or slower target sync cadence under epsend004 core settings. | Open in archived GPT review after bs256. |
| Whether `RewardInfoScale` can restore coverage, information gain, and success_rate on top of the efficient `LearnerUpdatesPerIter=1` structure without reintroducing high revisit, stall, turn burden, or timeout. | Open in archived GPT review after bounded learner-update comparison. |
| Whether posthoc/final_probe mismatch indicates a stability issue requiring later focused review. | Open as context only; not baseline decision authority by itself. |
| Whether learner/update dynamics such as learning rate or target update cadence should be considered if light reward shaping is weaker than epsend004. | Mentioned as conditional future consideration in archived GPT review after revisit012. |
| Cross-seed robustness. | Not evaluated by this single-seed tuning map and not the default blocker. |

## 8. Next Candidate Surfaces

| Surface | Status from archived GPT review |
| --- | --- |
| `bounded multi-run comparison over learner/update dynamics` with `Candidate A BatchSize: 128 -> 192; Candidate B LearnerUpdatesPerIter: 2 -> 1; Candidate C TargetUpdateInterval: 1000 -> 2000` | Recommended next candidate surface after bs256; represented by `recommended_next_run_name: stable_bs192_epsend004_budget500k_decay240k_minreplay8000_seed0; stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0; stable_target2000_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| `reward-function information-gain knob on top of validated lower update-to-data ratio` with `RewardInfoScale: 3.0 -> 3.2; keep LearnerUpdatesPerIter=1` | Recommended next surface after the bounded learner-update comparison; represented by `recommended_next_run_name: stable_updates1_info32_epsend004_budget500k_decay240k_minreplay8000_seed0`. |
| Systematic multi-run design or method_redesign_discussion_only | Conditional future surface only if all three bounded learner/update candidates remain weaker than epsend004. |

`stable_bs192_epsend004_budget500k_decay240k_minreplay8000_seed0`, `stable_updates1_epsend004_budget500k_decay240k_minreplay8000_seed0`, and `stable_target2000_epsend004_budget500k_decay240k_minreplay8000_seed0` are recorded only as recommended next run names. They are not summarized as archived results unless they become present as archived entries in `history_index.json`.

## 9. Maintenance Rule

Update this map only from `history_index.json` entries and archived GPT tuning reviews. Prefer archived GPT digest statements over Codex inference.

If a conclusion is not explicit in archived reviews, record `unresolved` or `not_recorded`. Do not duplicate full archived reviews, raw training outputs, full logs, full CSVs, plots, trajectories, checkpoints, model weights, or generated run artifacts.
