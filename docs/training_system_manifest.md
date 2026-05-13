# Training System Manifest

## 1. Role

This document records implementation facts needed by GPT tuning review and Codex execution planning. It describes the current training mainline, stable launcher, artifact semantics, tuning surface, and redesign boundary for active single-seed train-side performance search.

It does not recommend parameter values and does not perform tuning review.

## 2. Source Boundaries

| Repository | Owns |
| --- | --- |
| `DRL-path-finding` source training repository | Training code, runtime behavior, generated outputs, checkpoints, full logs, full CSVs, plots, trajectories, and exact per-run artifact schemas. |
| `Automatic_2` | Compact implementation manifest and tracked evidence digests. |

Use repository-relative labels in reusable docs where possible. Private/local paths should not be stored in tracked history.

## 3. Current Mainline

| Surface | Current implementation fact |
| --- | --- |
| Main entry point | `train_q_agent.py` |
| Policy network | `agents/q_value_agent.py::ExplorationQNetwork` |
| State adapter | `agents/q_value_agent.py::StateTensorAdapter` |
| State representation | Shared semantic dual-state tensors: `advantage_canvas`, `value_block_features`, `value_entry_features`, `value_block_mask`, `value_entry_mask`. |
| Semantic layer | `env/shared_semantic_layer.py::SharedSemanticLayer` |
| Network shape | Dual-branch semantic dueling Q network: local decision canvas advantage branch plus Accessible Unknown Block value branch. |
| Decision head | `heads/semantic_dueling_head.py::SemanticDuelingHead` combines `V(s)` and `A(s,a)` into 8-action Q values. |
| Learning algorithm | Double DQN with n-step transitions and uniform replay. |
| Formal protocol | `formal_posthoc_trainselect_v1` |

Legacy near/mid/token and frontier-token paths are not the current method mainline.

## 4. Stable Launcher

| Field | Fact |
| --- | --- |
| Launcher | `scripts/launch_formal_train_stable.ps1` |
| Entry point | Runs `train_q_agent.py`. |
| Reproducibility | Sets `PYTHONHASHSEED=0` and `CUBLAS_WORKSPACE_CONFIG=:4096:8`; passes strict reproducibility flags. |
| Runtime path | Disables TF32, cuDNN benchmark, AMP, inference AMP, `torch.compile`, and channels-last. |
| Seed policy | Uses fixed train episode and fixed final-probe seed bases. |
| Command prefix | Launcher-only command blocks should start with `.\scripts\launch_formal_train_stable.ps1`. |

Launcher-ready parameters exposed by `scripts/launch_formal_train_stable.ps1`:

| Parameter | Stable-launch role |
| --- | --- |
| `RunName` | Run directory name prefix. |
| `Device` | Torch device request. |
| `Seed` | Global Python, NumPy, and Torch seed input. |
| `TotalEnvSteps` | Env-step training budget. |
| `EpsilonDecaySteps` | Linear epsilon decay duration. |
| `EpsilonEnd` | Linear epsilon terminal value. |
| `MinReplaySize` | Replay warm threshold before learner updates. |
| `FinalGreedyEpisodes` | Held-out final-probe episode count. |
| `RewardRevisitPenalty`, `RewardTurnPenaltyScale`, `RewardTimeoutPenalty`, `RewardStepPenalty`, `RewardTerminalBonus`, `RewardInfoScale`, `RewardObstacleWeight` | Reward coefficients exposed for formal train-performance tuning. |
| `BatchSize`, `ReplayCapacity`, `NStep`, `Gamma`, `LearningRate`, `TargetUpdateInterval`, `GradClipNorm` | Replay and learner knobs exposed for formal train-performance tuning. |
| `CollectStepsPerIter`, `LearnerUpdatesPerIter`, `TrainEveryEnvSteps` | Rollout/update cadence knobs exposed for formal train-performance tuning. |
| `FixedTrainEpisodeSeedBase` | Fixed train episode seed base. |
| `FixedFinalProbeSeedBase` | Fixed final-probe seed base. |

GPT may produce a launcher-only `next_run_plan` only for changes supported by the stable launcher or an already supported CLI/config path. If a desired parameter is in `TrainConfig` but not exposed by the stable launcher, GPT should request a bounded config/launcher implementation task before training, not invent a launcher flag.

## 5. Artifact Semantics For Tuning Review

| Artifact | Tuning-review role |
| --- | --- |
| `logs/metric_snapshot.json` | Train-side primary metrics, deltas, diagnostics, learner context, train-final consistency, and selection summaries. |
| `logs/config_snapshot.json` | Serialized config, comparability/runtime fields, observed run contract, evaluation contract, reward semantics, and artifact references. |
| `logs/reproducibility_contract.json` | Strict reproducibility and launch contract: backend flags, seed policy, environment variables, sanitized argv, and contract verdict. |
| `logs/posthoc_selection_summary.json` | Train-side post-hoc candidate range, scoring window, selected candidate context, and selection diagnostics. |
| `logs/final_probe_summary.json` and `logs/final_probe.csv` | Supplemental held-out final-probe evidence for selected candidates and the formal winner row. |
| `logs/best_vs_last_gap_summary.json` | Diagnostic gap context between selected winner and last checkpoint or train endpoint summary. |
| `logs/artifact_index.json` | Artifact presence metadata for required and optional outputs. |

Full logs, full CSVs, checkpoints, plots, trajectories, binary artifacts, and raw generated outputs remain in the training repository.

## 6. Active Tuning Surface

| Category | Examples / fields | Access level | GPT action |
| --- | --- | --- | --- |
| A. Launcher-ready knobs | Stable-launch training parameters. | launcher-ready | May generate launcher-only `next_run_plan` if evidence supports. |
| B. CLI / `TrainConfig` knobs not exposed by stable launcher | Remaining training dynamics, replay, learner, post-hoc selection, and map/state sizing fields. | CLI/config-supported; stable launcher extension may be needed. | Do not invent launcher flags. Propose bounded launcher/config implementation task before training when needed. |
| C. Reward-function knobs not exposed by stable launcher | Remaining reward coefficients and timeout-shaping fields. | `TrainConfig` / CLI-supported if `parse_args()` exposes them; stable launcher extension may be needed. | May identify a bounded tuning direction when evidence ties reward shaping to train-side metrics, diagnostics, or behavior patterns. If not launcher-ready, request bounded implementation work. |
| D. Config-only fields without direct CLI flag | Fields present in `TrainConfig` without a direct `parse_args()` flag. | config/code-level | Cannot be launched through the current stable launcher unless exposed. Propose bounded implementation/config task before training. |
| E. Runtime-only / profiling toggles | Throughput, backend, profiling, plot, and trajectory export toggles. | runtime/profiling | Do not treat runtime speedup as system-performance superiority. Do not use for active tuning unless the user explicitly asks for runtime/profiling optimization. |
| F. Method-level redesign boundary | Architecture, representation, algorithm, reward-mechanism, action-space, or environment-semantics changes. | method-level | Use `method_redesign_discussion_only` or bounded user-approved implementation discussion. Do not generate unattended training command. |

Category A field list:
- `TotalEnvSteps`
- `EpsilonDecaySteps`
- `EpsilonEnd`
- `MinReplaySize`
- `FinalGreedyEpisodes`
- `Seed`
- `FixedTrainEpisodeSeedBase`
- `FixedFinalProbeSeedBase`
- `RewardRevisitPenalty`
- `RewardTurnPenaltyScale`
- `RewardTimeoutPenalty`
- `RewardStepPenalty`
- `RewardTerminalBonus`
- `RewardInfoScale`
- `RewardObstacleWeight`
- `BatchSize`
- `ReplayCapacity`
- `NStep`
- `Gamma`
- `LearningRate`
- `TargetUpdateInterval`
- `GradClipNorm`
- `CollectStepsPerIter`
- `LearnerUpdatesPerIter`
- `TrainEveryEnvSteps`

Category B field list:
- `periodic_checkpoint_interval_env_steps`
- `posthoc_candidate_start_env_steps`
- `posthoc_candidate_end_env_steps`
- `posthoc_selection_window_env_steps`
- `posthoc_final_probe_topk`
- `rows`
- `cols`
- `obstacle_ratio`
- `scan_radius`
- `max_accessible_blocks`
- `max_entries_per_block`

Category C field list:
- `reward_turn_weight_45`
- `reward_turn_weight_90`
- `reward_turn_weight_135`
- `reward_turn_weight_180`

Category D field list:
- `trajectory_history_steps`
- `max_episode_steps`
- `coverage_stop_threshold`
- `weight_decay`

Category E examples:
- AMP
- inference AMP
- `torch.compile`
- channels-last
- TF32
- cuDNN benchmark
- timing/profiling flags
- plot/trajectory export flags

Category F examples:
- network architecture changes
- state representation changes
- semantic layer redesign
- replay algorithm redesign beyond exposed config knobs
- reward mechanism rewrite beyond coefficient tuning
- action space or environment semantics changes

Artifact comparability labels such as `ALLOWED_TUNING_FIELDS`, `MANUAL_REVIEW_FIELDS`, and `FROZEN_COMPARABILITY_FIELDS` describe artifact comparison semantics. They do not by themselves prohibit active train-side performance-search proposals. GPT must still respect launcher/config availability and method-level redesign boundaries.

## 7. Interpretation Rules

- Train-side monitoring is the primary decision evidence for active tuning.
- Post-hoc selection is checkpoint-selection and stability context.
- `final_probe` is supplemental held-out evidence.
- The reproducibility contract controls formal evidence admission.
- Runtime speed alone is not system-performance superiority.
- Full logs, full CSVs, checkpoints, plots, trajectories, and raw artifacts stay in the training repository and are not copied to `Automatic_2`.

## 8. Unverified Or Requires Future Audit

- No training run was executed for this manifest update.
- Per-run artifact existence and exact row contents remain run-specific.
- Optional plotting and trajectory output formats are outside this compact tuning-surface manifest.
