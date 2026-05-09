# Current Training Run Factual Analysis

## Scope

- Report type: `training_run_factual_analysis`.
- Skill used: `training-run-factual-analysis`.
- Source run: `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/`.
- Output destination: `Automatic_2:training_results/current/`.
- This report is a neutral factual artifact summary. It is not a GPT tuning review, baseline decision, continuation/stop decision, branch decision, or hyperparameter recommendation task.
- Path sanitization: private absolute paths are omitted. Source artifacts are referenced with sanitized labels.

## Run identity

- Run name: `stable_baseline_decay240k_minreplay8000_seed0_20260508_090552`.
- Training run directory exists: true.
- Run mode: `cli`.
- Budget mode: `env_steps`.
- Configured total env steps: 500000.
- Final observed env steps: 500000.
- Total train episodes completed: 1016.
- Source git metadata from artifacts: branch `main`, commit `736a925ef3402d89056c7e9edca6c895674465e0`.

## Files inspected

- `Automatic_2:.agents/skills/training-run-factual-analysis/SKILL.md`
- `Automatic_2:docs/training_system_manifest.md`
- `Automatic_2:README.md`
- `DRL_automatic:README.md`
- `DRL_automatic:docs/project_baseline.md`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/reproducibility_contract.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/config_snapshot.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/benchmark_summary.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/artifact_index.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/metric_snapshot.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/train_steps.csv`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/train_episodes.csv`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/posthoc_candidate_scores.csv`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/posthoc_selection_summary.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/formal_selection_manifest.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/best_vs_last_gap_summary.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/final_probe.csv`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/final_probe_summary.json`
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/logs/training_summary.txt` summary header only
- `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/checkpoints/` metadata only

## Reproducible-launch verification

- Reproducible-launch status: `reproducible_launch_confirmed`.
- Contract verdict: `strict_contract_ready`.
- `strict_reproducibility`: true.
- `deterministic_algorithms_enabled`: true.
- `deterministic_algorithms_warn_only`: false.
- `PYTHONHASHSEED`: value `0`, present true.
- `CUBLAS_WORKSPACE_CONFIG`: value `:4096:8`, present true.
- Backend requested flags: `enable_tf32=false`, `enable_cudnn_benchmark=false`, `enable_amp=false`, `enable_inference_amp=false`, `enable_torch_compile=false`, `enable_channels_last=false`.
- Backend runtime readback: requested device `cuda`, `torch.backends.cudnn.deterministic=true`, `torch.backends.cudnn.benchmark=false`, TF32 matmul false, cuDNN TF32 false, deterministic algorithms enabled true.
- Fixed train episode seeds: `use_fixed_train_episode_seeds=true`, `fixed_train_episode_seed_base=20259323`.
- Fixed final-probe seeds: `use_fixed_eval_seeds=true`, `fixed_final_probe_seed_base=20261323`.
- Sanitized argv record present: true.

## Train-side monitoring summary

Train-side monitoring is the primary factual evidence in this report.

- `train_steps.csv`: parseable, 992 rows, 58 columns, no missing expected columns.
- `train_episodes.csv`: parseable, 1022 rows, 56 columns, no missing expected columns.
- `metric_snapshot.json`: parseable and includes recent train metrics, training dynamics, semantic monitoring, reward breakdown, reward events, and train-final consistency fields.

Primary train-step recent-window metrics from `train_steps.csv`:

| Category | First | Last | Last minus first | Recent mean | Recent window rows | Missing/non-finite |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| reward dynamics (`recent_mean_reward`) | 9.975186 | 145.010254 | 135.035068 | 138.833527 | 100 | 0 |
| coverage dynamics (`recent_mean_coverage`) | 0.4528 | 0.934791 | 0.481991 | 0.926996 | 100 | 0 |
| success_rate dynamics (`recent_success_rate`) | 0 | 0.81 | 0.81 | 0.7147 | 100 | 0 |
| episode_length dynamics (`recent_mean_episode_length`) | 600 | 385.73999 | -214.26001 | 417.2346 | 100 | 0 |
| repeat_visit_ratio dynamics (`recent_mean_repeat_visit_ratio`) | 0.626812 | 0.191746 | -0.435066 | 0.21592 | 100 | 0 |
| timeout indicators (`recent_timeout_flag`) | 1 | 0.19 | -0.81 | 0.2853 | 100 | 0 |
| stall diagnostics (`recent_stall_trigger_count`) | 311.857147 | 145.509995 | -166.347153 | 175.1662 | 100 | 0 |
| zero_info diagnostics (`recent_zero_info_step_count`) | 493.428558 | 208.119995 | -285.308563 | 240.3253 | 100 | 0 |
| recent revisit events (`recent_recent_revisit_trigger_count`) | 205.142853 | 40.21 | -164.932854 | 49.8957 | 100 | 0 |
| turn diagnostics (`recent_turn_ge_90_count`) | 387.285706 | 141.860001 | -245.425705 | 159.0723 | 100 | 0 |
| turn diagnostics (`recent_turn_135_count`) | 157.428574 | 38.619999 | -118.808575 | 44.6989 | 100 | 0 |
| turn diagnostics (`recent_turn_180_count`) | 78.85714 | 25.040001 | -53.817139 | 31.2984 | 100 | 0 |

Semantic monitoring from `train_steps.csv`:

| Field | First | Last | Last minus first | Recent mean | Missing/non-finite |
| --- | ---: | ---: | ---: | ---: | ---: |
| accessible_block_count | 2.225477 | 5.075697 | 2.85022 | 4.96289 | 0 |
| total_accessible_unknown_area | 424.649506 | 465.726562 | 41.077057 | 451.770362 | 0 |
| total_frontier_cluster_count | 8.565238 | 10.79704 | 2.231802 | 10.501713 | 0 |
| mean_block_area | 243.858414 | 120.983284 | -122.87513 | 120.100269 | 0 |
| local_frontier_coverage | 0.029014 | 0.027377 | -0.001637 | 0.025736 | 0 |
| local_frontier_block_area_mean | 0.607893 | 0.443375 | -0.164518 | 0.425003 | 0 |
| value_truncated_block_count | 0 | 0 | 0 | 0 | 0 |
| value_block_cap_hit_flag | 0 | 0 | 0 | 0 | 0 |
| value_truncated_entry_count | 0.326429 | 0.116217 | -0.210212 | 0.087956 | 0 |
| value_entry_cap_hit_flag | 0.182857 | 0.063192 | -0.119665 | 0.050569 | 0 |

Learner, replay, and exploration from `train_steps.csv`:

| Field | First numeric | Last | Last minus first | Recent mean | Missing/non-finite |
| --- | ---: | ---: | ---: | ---: | ---: |
| loss | 0.213267 | 2.317205 | 2.103938 | 2.474818 | 8 |
| q_mean | -0.08657 | 18.568867 | 18.655436 | 17.583103 | 8 |
| target_q_mean | 0.135255 | 19.202785 | 19.067531 | 17.751929 | 8 |
| td_abs_mean | 0.460748 | 2.667741 | 2.206993 | 2.829664 | 8 |
| grad_norm | 0.984113 | 29.291674 | 28.307561 | 30.3308 | 8 |
| replay_size | 4510 | 100000 | 95490 | 100000 | 0 |
| epsilon | 0.982203 | 0.05 | -0.932203 | 0.05 | 0 |

Reward breakdown and reward events from `train_steps.csv`:

| Field | First | Last | Last minus first | Recent mean | Missing/non-finite |
| --- | ---: | ---: | ---: | ---: | ---: |
| info_reward_sum | 62.196613 | 145.908707 | 83.712093 | 144.595655 | 0 |
| step_penalty_sum | -12 | -7.7148 | 4.2852 | -8.344692 | 0 |
| recent_revisit_penalty_sum | -20.514284 | -4.021 | 16.493284 | -4.98957 | 0 |
| turn_penalty_sum | -11.707143 | -3.842667 | 7.864476 | -4.439467 | 0 |
| timeout_penalty_sum | -8 | -1.52 | 6.48 | -2.2824 | 0 |
| terminal_bonus_sum | 0 | 16.2 | 16.2 | 14.294 | 0 |
| delta_empty_sum |  | 1444.349976 |  |  | 0 |
| delta_obstacle_sum |  | 334.410004 |  |  | 0 |
| empty_info_gain_sum |  | 45.975086 |  |  | 0 |
| obstacle_info_gain_sum |  | 10.6446 |  |  | 0 |
| weighted_info_gain_sum |  | 48.636234 |  |  | 0 |

Training dynamics reported by `metric_snapshot.json`:

- Final-window stats: reward 145.010254, coverage 0.934791, success_rate 0.81, episode_length 385.73999, repeat_visit_ratio 0.191746.
- Growth rates: reward 0.026636, coverage 0.000011, success_rate 0.000281, repeat_visit_ratio -0.000154.
- Threshold reach steps: success_050 at 217504, coverage_090 at 211008, reward_custom null.
- Late-stage variance: reward 14.972443, coverage 0.000024, success_rate 0.002786, repeat_visit_ratio 0.000475.
- Late phase slopes per 1000 env steps were present in `metric_snapshot.json`; the fields include reward, coverage, success_rate, episode_length, repeat_visit_ratio, accessible_block_count, frontier count, stall triggers, zero-info steps, turn_180_count, recent revisit penalty, and terminal bonus.
- Train-final consistency artifact verdict: `supports_final_probe`, with 5 aligned tracked metrics and 0 insufficient-evidence counts. This is reported as a source artifact fact, not as a tuning decision.

## Post-hoc selection summary

Post-hoc selection is summarized as checkpoint-selection context after train-side monitoring.

- Protocol name: `formal_posthoc_trainselect_v1`.
- Candidate range: 200000 to 500000 env steps.
- Checkpoint interval: 20000.
- Selection window env steps: 40000.
- Selection weights: reward 0.35, coverage 0.25, success_rate 0.20, episode_length -0.10, repeat_visit_ratio -0.10.
- Candidate score CSV: parseable, 16 rows, 19 columns, no missing expected columns.
- Candidate count: 16.
- Valid candidate count: 16.
- Selected candidate count: 3.
- Selected candidate steps: 500000, 400000, 420000.
- Winner step recorded by artifacts: 500000.
- Winner checkpoint metadata path: `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/checkpoints/ckpt_step_500000.pt`.
- `best.pt` metadata path: `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/checkpoints/best.pt`.
- `last.pt` metadata path: `training_run:stable_baseline_decay240k_minreplay8000_seed0_20260508_090552/checkpoints/last.pt`.
- `last.pt` received a held-out final_probe row: true, because the last candidate step was selected into the post-hoc top-k.
- best-vs-last comparison mode: `heldout_winner_vs_heldout_last_checkpoint`.
- best-vs-last diagnostic facts: reward, coverage, success_rate, episode_length, and repeat_visit_ratio all have `best_minus_last=0.0` in the artifact because the selected winner and last checkpoint both refer to env step 500000.

| Rank | Step | Selection score | Train-window reward | Coverage | Success rate | Episode length | RVR |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 500000 | 1.077135 | 139.709529 | 0.927699 | 0.722963 | 413.970247 | 0.212127 |
| 2 | 400000 | 1.008951 | 138.138682 | 0.927431 | 0.726543 | 419.245556 | 0.215291 |
| 3 | 420000 | 0.906463 | 137.381599 | 0.927014 | 0.695679 | 416.230248 | 0.228069 |

Checkpoint metadata only:

- 18 checkpoint files were present: `best.pt`, `last.pt`, and `ckpt_step_200000.pt` through `ckpt_step_500000.pt` at 20000-step intervals.
- `best.pt` size: 4755358 bytes.
- periodic candidate checkpoint file size observed for each `ckpt_step_*.pt`: 4743598 bytes.
- `last.pt` size: 14173678 bytes.
- Checkpoint payloads were not opened or copied.

## Supplemental final_probe summary

- `final_probe` is supplemental held-out validation evidence.
- `final_probe` is not standalone superiority evidence.
- Deterministic reproducibility does not remove finite-evaluation-sample limitations.
- `final_probe` is not treated as the sole basis for run quality.
- `final_probe.csv`: parseable, 3 rows, 64 columns, no missing expected columns.
- Episode count: 100.
- Seed base: 20261323.
- Ranking order: success_rate, coverage, reward.
- Winner row step: 500000.

| Final-probe rank | Source | Step | Episodes | Reward | Coverage | Success rate | Episode length | RVR | Timeout flag | Winner |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| 1 | `posthoc_final_winner` | 500000 | 100 | 133.409958 | 0.913702 | 0.75 | 398.149994 | 0.240474 | 0.25 | true |
| 2 | `posthoc_candidate_final_probe` | 420000 | 100 | 121.717377 | 0.890114 | 0.61 | 440.149994 | 0.288817 | 0.39 | false |
| 3 | `posthoc_candidate_final_probe` | 400000 | 100 | 124.34948 | 0.899973 | 0.56 | 440.230011 | 0.28499 | 0.44 | false |

Supplemental final-probe diagnostics:

- Winner row stall trigger count: 177.759995.
- Winner row zero-info step count: 231.179993.
- Winner row recent revisit trigger count: 75.21.
- Winner row turn counts: turn_ge_90 161.929993, turn_135 26.78, turn_180 62.92.
- Winner row semantic monitoring: accessible_block_count 5.242034, total_accessible_unknown_area 466.389648, total_frontier_cluster_count 11.065923, mean_block_area 115.725319, local_frontier_coverage 0.027506, local_frontier_block_area_mean 0.429366.
- Winner row value cap diagnostics: value_truncated_block_count 0.0, value_block_cap_hit_flag 0.0, value_truncated_entry_count 0.088311, value_entry_cap_hit_flag 0.055942.

## Configuration and runtime facts

- `total_env_steps`: 500000.
- `budget_mode`: `env_steps`.
- `total_train_episodes`: 600.
- `epsilon_decay_steps`: 240000.
- `epsilon_end`: 0.05.
- `min_replay_size`: 8000.
- `batch_size`: 128.
- `reward_info_scale`: 3.0.
- `reward_revisit_penalty`: 0.1.
- `reward_turn_penalty_scale`: 0.05.
- `scan_radius`: 10.
- `rows`: 40.
- `cols`: 60.
- `final_greedy_episodes`: 100.
- Post-hoc config fields: candidate start 200000, candidate end 0 in config meaning use total env steps, checkpoint interval 20000, selection window 40000, final-probe top-k 3.
- Runtime duration: 04:07:55, 14874.588540800001 seconds.
- Device: requested `cuda`, actual `cuda:0`.
- Runtime/performance switches: AMP false, inference AMP false, torch.compile false, cuDNN benchmark false, TF32 false, strict reproducibility true, deterministic warn-only false, channels-last false, plot export false, trajectory export false.
- Artifact inventory: 7 CSV entries, 18 checkpoint entries, 9 structured summaries, 1 stdout summary, 0 plot entries, 0 trajectory entries.

## Missingness and parseability

- No required artifact was missing from the inspected run.
- No parse failures were recorded for required JSON or CSV artifacts.
- Optional legacy/non-current CSVs absent per `artifact_index.json`: `logs/model_select_eval.csv`, `logs/best_recheck_eval.csv`, `logs/eval_metrics.csv`.
- Optional plots and trajectories had no artifact entries and were not copied.

## Forbidden artifact check

- Checkpoint files were observed only as metadata. No checkpoint payload was opened, copied, or published.
- No model weights, full logs, full CSV contents, raw output directories, plots, or trajectories were copied into `Automatic_2`.
- Report paths use sanitized labels rather than private absolute paths.
- Forbidden artifact findings: none.

## Unverified items

- Checkpoint binaries were not opened, copied, or inspected beyond metadata.
- No training command was run.
- No evaluation command was run.
- Plots, trajectories, and raw output directories were not copied or published.
- The source training repository had a pre-existing untracked directory in git status before report writing; no tracked modification was observed.

## Factual summary status

- Factual summary status: `factual_summary_ready`.

## No tuning recommendation provided

- `tuning_recommendation_provided`: false.
- No accepted-baseline decision, next hyperparameter, stop/continue decision, branch decision, method-level conclusion, or paper-level conclusion is provided.
