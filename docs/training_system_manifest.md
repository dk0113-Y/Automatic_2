# Training System Manifest

## Document Role

This manifest is a passive engineering description of the current `DRL-path-finding` training implementation. It summarizes the implemented project, algorithmic mainline, major modules, state construction pipeline, network architecture, learner lifecycle, checkpoint lifecycle, generated outputs, configuration surface, and legacy boundaries.

The document is stored in `Automatic_2`, but it does not describe `Automatic_2` workflow behavior. It is not a GPT tuning guide, a Codex evidence checklist, a control-plane routing document, a decision policy, a metric-review policy, a run review, or a value-selection document.

Confirmed statements are based on source inspection of the local training repository. Statements that were not confirmed by source inspection or by executing a run are listed under "Unverified Or Requires Future Audit". This manifest does not replace source inspection when implementation facts are needed.

## Source-of-Truth Boundary

`DRL-path-finding` / the local training repository owns the training implementation and runtime facts:

- training source code
- command-line interface behavior
- output directory layout
- logs and structured JSON outputs
- checkpoints and model weights
- plots, trajectories, and other generated training artifacts
- exact artifact schemas for a completed run

`Automatic_2` stores this summary. It does not own training code, runtime behavior, model weights, checkpoints, full logs, full CSV contents, plots, trajectories, or generated run artifacts.

The public training repository reference is `https://github.com/dk0113-Y/DRL-path-finding`. The local training checkout remains the implementation source for current facts.

## Current Algorithmic Mainline

Confirmed current mainline:

- Main training entry point: `train_q_agent.py`.
- Current policy network: `agents/q_value_agent.py::ExplorationQNetwork`.
- Current state adapter: `agents/q_value_agent.py::StateTensorAdapter`.
- Current state representation: shared semantic dual-state tensors:
  - `advantage_canvas`
  - `value_block_features`
  - `value_entry_features`
  - `value_block_mask`
  - `value_entry_mask`
- Current environment semantic layer: `env/shared_semantic_layer.py::SharedSemanticLayer`.
- Current decision architecture: dual-branch semantic dueling Q network:
  - the advantage branch reads a local decision canvas
  - the value branch reads an Accessible Unknown Block tree
  - `heads/semantic_dueling_head.py::SemanticDuelingHead` combines `V(s)` and `A(s,a)` into 8-action Q values
- Current learning algorithm: Double DQN with n-step transitions and uniform replay.
- Current default protocol name: `formal_posthoc_trainselect_v1`.

The active network imports `AdvantageCanvasEncoder`, `ValueTreeEncoder`, and `SemanticDuelingHead`. It does not import the legacy near/mid/token encoder and split-head path for the current training mainline.

Confirmed legacy/non-mainline files include:

- `encoders/local_encoder.py`, documented as the legacy near-map encoder path.
- `encoders/global_encoder.py`, documented as the legacy near/mid/token encoder path.
- `heads/q_head.py`, documented as the legacy split-input dueling head.
- `env/frontier_token_builder.py`, documented as the legacy frontier-token builder.

## Training Entry Points

Confirmed training entry points and helper presets:

- `train_q_agent.py`
  - defines `TrainConfig`
  - defines `parse_args()`
  - defines `build_system(cfg)`
  - defines `run_training(cfg, run_mode="cli")`
  - uses `if __name__ == "__main__"` for CLI launch
- Standard CLI launch documented by the training README:
  - `python .\train_q_agent.py --device cuda`
- Smoke/debug launch:
  - `python .\train_q_agent.py --smoke --device cpu`
- VSCode/local preset helpers:
  - `build_vscode_config()`
  - `build_profile_config()`
  - `build_fast_cuda_config()`
  - `build_fast_cuda_profile_config()`
  - `build_profile_compile_config()`
- Runtime profiling entry:
  - CLI `--profile` enables timing flags.
- Experimental runtime acceleration entry:
  - CLI `--fast-cuda` enables AMP, inference AMP, `torch.compile`, channels-last, cuDNN benchmark, and TF32.
  - The code marks this as a runtime path, not an algorithmic mainline change.

Stable reproducibility and local automation scripts:

- `scripts/launch_formal_train_stable.ps1`
  - launches `train_q_agent.py` with strict reproducibility flags.
  - sets `PYTHONHASHSEED=0` and `CUBLAS_WORKSPACE_CONFIG=:4096:8`.
  - passes `--strict-reproducibility`, `--no-deterministic-warn-only`, `--no-enable-tf32`, `--no-enable-cudnn-benchmark`, and disables AMP/compile/channels-last runtime paths.
  - script parameter defaults differ from main CLI defaults for `EpsilonDecaySteps`, `EpsilonEnd`, and `MinReplaySize`.
- `scripts/launch_stable_same_seed_ab.ps1`
  - runs two stable launches sequentially and invokes `scripts/check_reproducibility_contract.py` after each completed run.
- `scripts/check_reproducibility_contract.py`
  - validates fields inside `logs/reproducibility_contract.json`.
- `scripts/experiment_scheduler.py` and `scripts/tuning/**`
  - implement local sequential scheduling and comparison logic inside the training repository.
  - These scripts are not part of the default `train_q_agent.py` training entry point.
- `scripts/replay_posthoc_diagnostics.py`
  - replays post-hoc candidate selection diagnostics from existing logs.
- `tools/supplementary_multi_checkpoint_probe.py`
  - runs supplementary held-out checkpoint probes.
  - Includes compatibility support for historical 5/6/7 channel advantage layouts.
- `tools/backfill_formal_run_artifacts.py` and `tools/generate_historical_baseline_summary.py`
  - are historical/backfill summary tools for existing outputs.
  - They were inspected for role but not executed.

## Repository Structure And Module Responsibilities

Confirmed source layout:

- `train_q_agent.py`
  - Main assembly point for configuration, runtime setup, model/replay/collector/learner/evaluator construction, training loop, checkpoint lifecycle, post-hoc final selection, final probe, and structured artifact writing.
- `agents/`
  - Current Q network, state tensor adapter, legal-action masking helpers, and greedy action selection.
- `encoders/`
  - Current mainline: `advantage_encoder.py`, `value_encoder.py`.
  - Legacy historical paths: `local_encoder.py`, `global_encoder.py`.
- `heads/`
  - Current mainline: `semantic_dueling_head.py`.
  - Legacy historical path: `q_head.py`.
- `env/`
  - Random map generation, grid topology, radar/local observation, cumulative belief map, shared semantic layer, advantage state builder, and value state builder.
- `training/`
  - Collector, evaluator, replay buffer, DDQN learner, checkpointing, CSV logging, post-hoc selection, formal artifact writer, plotting, trajectory plotting, and reward calculations.
- `scripts/`
  - Stable launchers, reproducibility contract checker, post-hoc diagnostic replay, local scheduler, and local tuning support modules.
- `tools/`
  - Supplementary probes, historical backfill/generation scripts, architecture/visualization helpers, and shared semantic layer asset export helpers.
- `formal_artifacts/`
  - Contains tracked historical summary artifacts such as `historical_baseline_summary.json`; this directory is not current training source code.
- `outputs/`
  - Generated run outputs. This directory is ignored by the training repository `.gitignore` and was not audited as source.

Top-level `demos/`, `docs/`, `run_picture/`, `tmp/`, generated caches, and runtime directories were observed but were not audited as current formal training mainline for this manifest.

## Environment And State Construction Pipeline

Confirmed pipeline:

1. `env/block_random_g.py::RandomMapGenerator`
   - Builds bounded random obstacle maps and valid 1x1 start states.
   - Adds border obstacles.
   - Creates rectangular obstacle structures.
   - Keeps the largest free component.
   - Can bind map generation to explicit seeds.
   - `compute_map_fingerprint()` records a compact SHA-1 based grid/start fingerprint.
2. `env/core_radar.py::RadarSensor`
   - Defines the local observation footprint.
   - `scan_radius` controls local window shape `(2R+1, 2R+1)`.
   - Uses Euclidean disk candidate visibility with a small cardinal-tip shoulder.
   - Precomputes line-of-sight templates.
   - `block_corner_peeking` controls diagonal corner occlusion handling.
3. `env/agent_version.py::LocalObservationModel`
   - Renders local observations from the true map.
   - The policy does not receive the full true map.
   - The hot path is `observe_fast()`, which reuses a persistent `local_snap` buffer.
4. `env/core_cummap.py::CumulativeBeliefMap`
   - Maintains agent-side cumulative knowledge in `self.map`, `visit_count`, frontier caches, coverage metrics, and `analysis_box`.
   - Agent-side state construction uses cumulative belief, visit history, and frontier state.
   - Effective coverage is a simulator-side metric and termination statistic.
   - The analysis box is a strict known-region bounding box with `analysis_margin = 0`.
   - The frontier cache has debug comparison support through `debug_frontier_consistency_stats()` and `debug_compare_frontier_with_full_recompute()`.
5. `env/shared_semantic_layer.py::SharedSemanticLayer`
   - Analyzes the cumulative belief map.
   - Extracts 8-connected frontier clusters from the current frontier view.
   - Groups adjacent unknown components through frontier-first ownership.
   - Produces `UnknownBlock` objects as Accessible Unknown Blocks.
   - Attaches `FrontierCluster` children to each block.
   - Attaches `SupportGeometry` with known-side local obstacle-density summaries.
   - Exposes semantic metrics through `SharedSemanticSnapshot.metrics()`, including accessible block count, total accessible unknown area, total frontier cluster count, and mean block area.
6. `env/advantage_state_builder.py::AdvantageStateBuilder`
   - Builds the local decision canvas consumed by the advantage branch.
   - Canvas size is tied to the radar local observation window.
   - Confirmed channel layout:
     - `free`
     - `obstacle`
     - `frontier_block_area_map`
     - `visit_count_log_norm`
     - `recent_trajectory_decay`
   - Paints local frontier cluster cells with the parent block area ratio.
   - Includes cumulative revisit pressure from visit counts.
   - Includes short-horizon trajectory decay from recent occupied positions.
7. `env/value_state_builder.py::ValueStateBuilder`
   - Builds the block-tree tensor consumed by the value branch.
   - Block features:
     - `block_area_ratio`
     - `frontier_cluster_count`
   - Entry features:
     - `delta_r_ratio`
     - `delta_c_ratio`
     - `entry_width_ratio`
     - `support_obstacle_density`
   - Masks:
     - `value_block_mask`
     - `value_entry_mask`
   - Caps:
     - `max_accessible_blocks`
     - `max_entries_per_block`
   - Emits diagnostic fields for truncation and cap-hit monitoring, including `value_truncated_block_count`, `value_entry_cap_hit_flag`, and related count fields.

## Network / Encoding / Decision Head

Confirmed network mainline:

- `agents/q_value_agent.py::ExplorationQNetwork`
  - Validates action dimension and feature dimensions against current builders before module construction.
  - Uses `AdvantageCanvasEncoder`, `ValueTreeEncoder`, and `SemanticDuelingHead`.
- Advantage branch:
  - `encoders/advantage_encoder.py::AdvantageCanvasEncoder`
  - Convolutional residual backbone over the local decision canvas.
  - Creates one action-specific state per move using directional spatial masks, landing-cell features, center features, and action embeddings.
  - Default action state dimension: `160`.
- Value branch:
  - `encoders/value_encoder.py::ValueTreeEncoder`
  - Encodes parent block tokens and child frontier-entry tokens.
  - Runs masked self-attention over sibling entries inside each block.
  - Uses parent-conditioned child aggregation.
  - Applies parent-grounded gated block updates.
  - Runs masked block self-attention across accessible blocks.
  - Uses learned state-query weighted pooling into the value state.
  - Default value state dimension: `192`.
- Decision head:
  - `heads/semantic_dueling_head.py::SemanticDuelingHead`
  - Computes scalar `V(s)` from the value state.
  - Computes per-action `A(s,a)` from advantage states.
  - Outputs `Q(s,a) = V(s) + A(s,a) - mean_a A(s,a)`.
- Action space:
  - 8-neighbor grid movement from `env/grid_topology.py::ACTIONS_8`.
  - Movement legality uses corner-cut prevention.
  - Legal action masks are applied during greedy selection and Double DQN target selection.

## Learning Pipeline

Confirmed learning pipeline:

- `training/collector.py::TransitionCollector`
  - Owns environment interaction.
  - Resets episodes by generating a map/start, local observation, cumulative belief map, frontier, shared semantic snapshot, and initial state tensors.
  - Selects actions with epsilon-greedy policy under legal-action masks.
  - Random branches use Python `random`.
  - Greedy branches run model inference under legal-action masks.
  - Applies reward bookkeeping, coverage termination, timeout termination, stall diagnostics, zero-info diagnostics, repeat-visit metrics, and semantic monitoring.
  - Pushes transitions into the n-step builder and replay buffer.
- `training/replay_buffer.py::NStepTransitionBuilder`
  - Default `n_step=3`, `gamma=0.99`.
  - Builds n-step rewards and bootstrap discounts.
  - Flushes terminal prefixes at episode end.
- `training/replay_buffer.py::ReplayBuffer`
  - Uniform replay by default.
  - Stores current state tensors, action masks, action, reward, next state tensors, next action mask, done flag, and bootstrap discount.
  - `prioritized` is present as a reserved extension point and is not active by default.
- `training/learner.py::DDQNLearner`
  - Uses online and target networks.
  - Selects target action by online Q values over `next_action_mask`.
  - Gathers target value from the target network.
  - Uses Smooth L1 loss on TD error.
  - Uses Adam optimizer.
  - Uses hard target sync every `target_update_interval`.
  - Optional AMP affects runtime execution path, not metric field definitions.
- `training/evaluator.py::GreedyEvaluator`
  - Runs greedy held-out evaluations.
  - Uses the same state adapter and reward/event accounting surfaces as the collector.
- `training/logger.py::CSVMetricLogger`
  - Writes append-only CSVs:
    - `logs/train_episodes.csv`
    - `logs/train_steps.csv`
    - `logs/final_probe.csv`
    - `logs/model_select_eval.csv`
    - `logs/best_recheck_eval.csv`

Reward and diagnostic semantics are implemented in `training/rewarding.py`. The reward mainline uses weighted information gain, step penalty, recent revisit penalty, turn penalty, terminal bonus, and timeout penalty. Stall and zero-info events are recorded as diagnostics and are not formal reward terms in the inspected code.

## Formal Training Lifecycle

Confirmed current default protocol:

- `formal_protocol = formal_posthoc_trainselect_v1`.
- Budget modes:
  - `env_steps`: training stops at `total_env_steps`; warmup uses `warmup_steps`.
  - `episodes`: training stops at `total_train_episodes`; warmup uses `warmup_episodes`.
- Training loop:
  - Performs warmup collection.
  - Alternates rollout collection and learner updates.
  - Logs train snapshots by step or episode cadence.
  - Saves train-side periodic post-hoc checkpoints after `posthoc_candidate_start_env_steps` at `periodic_checkpoint_interval_env_steps`.
- Under `formal_posthoc_trainselect_v1`:
  - No checkpoint validation runs during training.
  - No recheck runs during training.
  - No final probe runs during training.
  - Collector/learner execution is not paused for selection evaluation during training.
  - Periodic candidates are train-only checkpoint files matching `checkpoints/ckpt_step_<env_steps>.pt`.
- After training:
  - `checkpoints/last.pt` is saved as the training endpoint checkpoint.
  - `training.posthoc_selection.select_posthoc_candidates()` reads train-side logs and candidate checkpoint filenames.
  - Candidate scoring uses windowed train metrics from `logs/train_steps.csv`.
  - Default score weights in `training/posthoc_selection.py::DEFAULT_SELECTION_WEIGHTS`:
    - reward z-score: `0.35`
    - coverage z-score: `0.25`
    - success-rate z-score: `0.20`
    - episode-length z-score: `-0.10`
    - repeat-visit-ratio z-score: `-0.10`
  - Up to `posthoc_final_probe_topk` selected candidates receive held-out final probes.
  - Final probe ranking order is `success_rate`, then `coverage`, then `reward`.
  - The winner checkpoint payload is promoted to `checkpoints/best.pt`.
  - `checkpoints/last.pt` remains the training endpoint checkpoint. It receives a held-out final-probe row only if it is selected into the post-hoc top-k candidate set.
- Artifact writing:
  - `training.posthoc_selection.write_posthoc_final_artifacts()` writes post-hoc final selection JSON summaries.
  - `training.formal_artifacts.write_formal_run_artifacts()` writes metric, benchmark, config, reproducibility, artifact-index, and text-summary artifacts.

Legacy lifecycle:

- `formal_best_checkpoint_v3` remains a compatibility protocol.
- Legacy periodic validation writes `logs/model_select_eval.csv`.
- Legacy top-k recheck writes `logs/best_recheck_eval.csv`.
- Legacy model-selection checkpoints are stored under `checkpoints/model_select/*.pt`.

The training README contains historical model-selection wording in places. The lifecycle statements above are based on inspected code in `train_q_agent.py`, `training/posthoc_selection.py`, and `training/checkpointing.py`.

## Generated Artifact Interface

The training system writes lightweight CSV, JSON, and text artifacts under each run directory. These files describe training progress, selection metadata, runtime settings, and artifact inventory. Model checkpoint files are binary training artifacts and are not stored in `Automatic_2`.

Confirmed structured artifacts:

| Artifact | Status | Written by | Contents |
| --- | --- | --- | --- |
| `logs/metric_snapshot.json` | confirmed by code | `training/formal_artifacts.py` | Unified metric snapshot containing recent train, final probe, checkpoint selection summary, diagnostic last checkpoint summary, best-vs-last diagnostics, training dynamics summary, train-final consistency fields, and insufficient-evidence flags. |
| `logs/benchmark_summary.json` | confirmed by code | `training/formal_artifacts.py` | Runtime summary, run mode, runtime/performance switches, timing switch status, optional timing summaries, protocol revision, budget mode, best/last env steps, and runtime support fields. |
| `logs/config_snapshot.json` | confirmed by code | `training/formal_artifacts.py` | Serialized `TrainConfig`, git branch/SHA, comparability groups, runtime-only fields, observed run contract, evaluation contract, reward semantics, and structured artifact references. |
| `logs/reproducibility_contract.json` | confirmed by code | `training/formal_artifacts.py` | Launch/runtime determinism contract, environment variables, backend flags, seed policies, sanitized argv, and `contract_verdict`. |
| `logs/artifact_index.json` | confirmed by code | `training/formal_artifacts.py` | Inventory of expected CSVs, checkpoints, structured summaries, optional plots, and optional trajectories, with existence and required/optional status. |
| `logs/posthoc_candidate_scores.csv` | confirmed by code | `training/posthoc_selection.py` | Train-window candidate scores, z-scores, selection ranks, candidate checkpoint paths, and validity flags for post-hoc selection. |
| `logs/posthoc_selection_summary.json` | confirmed by code | `training/posthoc_selection.py` | Candidate range, candidate count, selected candidate steps, score weights, all candidate summaries, and diagnostic payload. |
| `logs/final_probe_summary.json` | confirmed by code | `training/posthoc_selection.py` | Top-k held-out final-probe rows, winner row, winner checkpoint path, `best.pt` path, `last.pt` path, seed base, and episode count. |
| `logs/best_vs_last_gap_summary.json` | confirmed by code | `training/posthoc_selection.py` | Diagnostic gap between the selected winner and the last checkpoint, using a held-out last candidate row when present or the recent train endpoint summary otherwise. |
| `logs/formal_selection_manifest.json` | confirmed by code | `training/posthoc_selection.py` | Protocol name, candidate range, checkpoint interval, selected candidate steps, final probe episode count, seed base, winner step/checkpoint, `best.pt`, `last.pt`, and related artifact paths. |
| `logs/training_summary.txt` | confirmed by code | `training/formal_artifacts.py` | Human-readable lightweight summary built from metric and benchmark snapshots. |

Confirmed CSV artifacts:

| Artifact | Status | Contents |
| --- | --- | --- |
| `logs/train_steps.csv` | confirmed by code | Recent train snapshots by env-step or final logging cadence. It is also the source for post-hoc candidate scoring. |
| `logs/train_episodes.csv` | confirmed by code | Per-episode train records, including seeds/fingerprints when fixed train episode seeds are enabled. |
| `logs/final_probe.csv` | confirmed by code | Held-out final probe rows. Under the current post-hoc protocol, it includes probed top-k candidates and marks the selected winner. |
| `logs/model_select_eval.csv` | confirmed by code, legacy-compatible | Periodic validation rows for `formal_best_checkpoint_v3`; not active in the current default post-hoc protocol. |
| `logs/best_recheck_eval.csv` | confirmed by code, legacy-compatible | Top-k recheck rows for `formal_best_checkpoint_v3`; not active in the current default post-hoc protocol. |
| `logs/eval_metrics.csv` | documented by artifact index as legacy diagnostic | Historical/legacy diagnostic CSV if present. Not part of the current default post-hoc training path. |

Confirmed checkpoint artifacts:

| Artifact | Status | Contents |
| --- | --- | --- |
| `checkpoints/best.pt` | confirmed by code | Selected winner checkpoint payload promoted from an already-saved candidate. The payload includes `online_state_dict`, step metadata, serialized config, and selection metadata. |
| `checkpoints/last.pt` | confirmed by code | Training endpoint checkpoint payload. The payload includes optimizer state by default. |
| `checkpoints/ckpt_step_<env_steps>.pt` | confirmed by code | Train-side post-hoc candidate checkpoint payload. Optimizer state is disabled by default in `save_periodic_checkpoint()`. |
| `checkpoints/model_select/*.pt` | legacy-compatible | Legacy model-selection candidate checkpoint payloads under `formal_best_checkpoint_v3`. |

Optional generated artifacts:

- `plots/**`
  - Generated only when plot export is enabled.
- `trajectories/**`
  - Generated only when trajectory export is enabled.
- Training repository `.gitignore` excludes `outputs/`, `perf_runs/`, and `tmp_plot_smoke/`.

## Configuration Surface

This section lists confirmed configuration fields and their engineering role. It does not recommend values.

Core runtime and output fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `device` / `--device` | `cuda` | Torch device request. |
| `seed` / `--seed` | `0` | Global Python, NumPy, and Torch seed input before model/system initialization. |
| `output_root` / `--output-root` | `outputs` | Parent directory for run outputs. |
| `run_name` / `--run-name` | `ddqn_explore_vscode_stage5` | Prefix used in timestamped run directory names. |
| `budget_mode` / `--budget-mode` | `env_steps` | Budget organization surface; `env_steps` and `episodes` are supported. |
| `total_env_steps` / `--total-env-steps` | `500000` | Env-step budget when `budget_mode=env_steps`. |
| `total_train_episodes` / `--total-train-episodes` | `600` | Episode budget when `budget_mode=episodes`. |
| `warmup_steps` / `--warmup-steps` | `4000` | Env-step warmup for `env_steps` mode. |
| `warmup_episodes` / `--warmup-episodes` | `0` | Episode warmup for `episodes` mode. |
| `collect_steps_per_iter` / `--collect-steps-per-iter` | `16` | Rollout collection chunk size. |
| `learner_updates_per_iter` / `--learner-updates-per-iter` | `2` | Learner updates attempted per collection chunk. |
| `train_every_env_steps` / `--train-every-env-steps` | `16` | Env-step spacing for learner-update attempts. |
| `log_interval` / `--log-interval` | `500` | Step-level train snapshot cadence. |
| `log_interval_episodes` / `--log-interval-episodes` | `10` | Episode-mode train snapshot cadence. |
| `recent_episode_window` / `--recent-episode-window` | `100` | Window size for recent train summaries. |

Environment and state fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `rows` / `--rows` | `40` | Map rows. |
| `cols` / `--cols` | `60` | Map columns. |
| `obs_size` / `--obs-size` | `6` | Obstacle-generation size parameter. |
| `scan_radius` / `--scan-radius` | `10` | Radar radius and advantage-canvas local scale. |
| `trajectory_history_steps` | `10` | Shared short-horizon length for trajectory canvas and recent revisit penalty horizon. No CLI flag was found for this field in `parse_args()`. |
| `obstacle_ratio` / `--obstacle-ratio` | `0.20` | Map obstacle density target. |
| `max_accessible_blocks` / `--max-accessible-blocks` | `16` | Value-state block cap. |
| `max_entries_per_block` / `--max-entries-per-block` | `8` | Value-state child-entry cap. |
| `max_episode_steps` | `600` | Per-episode step cap in `TrainConfig`; no direct CLI flag was found in `parse_args()`. |
| `coverage_stop_threshold` | `0.95` | Coverage success threshold in `TrainConfig`; no direct CLI flag was found in `parse_args()`. |

Replay and learner fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `replay_capacity` / `--replay-capacity` | `100000` | Uniform replay capacity. |
| `batch_size` / `--batch-size` | `128` | Learner batch size. |
| `min_replay_size` / `--min-replay-size` | `4000` | Minimum replay size before learner updates. |
| `n_step` / `--n-step` | `3` | N-step transition length. |
| `gamma` / `--gamma` | `0.99` | Discount factor. |
| `learning_rate` / `--learning-rate` | `1.0e-4` | Adam optimizer learning rate. |
| `weight_decay` | `0.0` | Adam optimizer weight decay in `TrainConfig`; no direct CLI flag was found in `parse_args()`. |
| `grad_clip_norm` / `--grad-clip-norm` | `10.0` | Gradient clipping norm. |
| `target_update_interval` / `--target-update-interval` | `1000` | Hard target sync interval. |
| `prefer_batch_replay_add` / `--prefer-batch-replay-add` | `true` | Replay write-path optimization. |
| `learner_debug_stats_every` / `--learner-debug-stats-every` | `8` | Learner debug-stat emission cadence. |

Exploration fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `epsilon_start` / `--epsilon-start` | `1.0` | Linear epsilon schedule start. |
| `epsilon_end` / `--epsilon-end` | `0.03` | Linear epsilon schedule end. |
| `epsilon_decay_steps` / `--epsilon-decay-steps` | `400000` | Linear epsilon schedule duration. |

Reward fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `reward_info_scale` / `--reward-info-scale` | `3.0` | Weighted information-gain reward scale. |
| `reward_obstacle_weight` / `--reward-obstacle-weight` | `0.25` | Obstacle reveal weighting inside information gain. |
| `reward_step_penalty` / `--reward-step-penalty` | `0.02` | Per-step penalty. |
| `reward_terminal_bonus` / `--reward-terminal-bonus` | `20.0` | Coverage-success terminal bonus. |
| `reward_revisit_penalty` / `--reward-revisit-penalty` | `0.10` | Recent revisit penalty. |
| `reward_turn_penalty_scale` / `--reward-turn-penalty-scale` | `0.05` | Total turn penalty scale. |
| `reward_turn_weight_45` / `--reward-turn-weight-45` | `0.0` | Turn angle weight. |
| `reward_turn_weight_90` / `--reward-turn-weight-90` | `1/3` | Turn angle weight. |
| `reward_turn_weight_135` / `--reward-turn-weight-135` | `2/3` | Turn angle weight. |
| `reward_turn_weight_180` / `--reward-turn-weight-180` | `1.0` | Turn angle weight. |
| `reward_timeout_penalty` / `--reward-timeout-penalty` | `8.0` | Timeout penalty. |

Current formal protocol fields:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `formal_protocol` / `--formal-protocol` | `formal_posthoc_trainselect_v1` | Current default protocol lane. |
| `final_greedy_episodes` / `--final-greedy-episodes` | `100` | Held-out final-probe episode count. |
| `use_fixed_train_episode_seeds` / `--use-fixed-train-episode-seeds` | `true` | Binds train episode index to fixed seed sequence. |
| `fixed_train_episode_seed_base` / `--fixed-train-episode-seed-base` | `20259323` | Train seed base. |
| `use_fixed_eval_seeds` / `--use-fixed-eval-seeds` | `true` | Legacy-named toggle controlling final-probe fixed seeds. |
| `fixed_final_probe_seed_base` / `--fixed-final-probe-seed-base` | `20261323` | Final-probe held-out seed base. |
| `periodic_checkpoint_interval_env_steps` / `--periodic-checkpoint-interval-env-steps` | `20000` | Post-hoc train-side candidate interval. |
| `posthoc_candidate_start_env_steps` / `--posthoc-candidate-start-env-steps` | `200000` | Start of candidate range. |
| `posthoc_candidate_end_env_steps` / `--posthoc-candidate-end-env-steps` | `0` | Candidate range end; `0` means use `total_env_steps`. |
| `posthoc_selection_window_env_steps` / `--posthoc-selection-window-env-steps` | `40000` | Train-window width for post-hoc scoring. |
| `posthoc_final_probe_topk` / `--posthoc-final-probe-topk` | `3` | Maximum selected candidates to final-probe. |

Runtime/debug fields:

- Runtime/performance flags:
  - `enable_amp`
  - `amp_dtype`
  - `enable_inference_amp`
  - `enable_torch_compile`
  - `compile_mode`
  - `enable_cudnn_benchmark`
  - `enable_tf32`
  - `enable_channels_last`
  - `fast_cuda`
- Strict reproducibility flags:
  - `strict_reproducibility`
  - `deterministic_warn_only`
- Timing/profiling flags:
  - `enable_collector_timing`
  - `enable_learner_timing`
  - `enable_replay_timing`
  - `enable_state_adapter_timing`
  - `enable_cummap_timing`
  - `enable_shared_semantic_timing`
  - `enable_advantage_state_timing`
  - `enable_value_state_timing`
  - `timing_log_interval`
  - `profile`
- Optional artifact export flags:
  - `save_train_representative_trajectories`
  - `save_train_special_trajectories`
  - `save_final_probe_trajectories`
  - `generate_plots_on_finish`
- Debug/stdout fields:
  - `debug_check_incremental_frontier`
  - `smoke`
  - `episode_print_interval`
  - `train_print_interval`
  - `train_print_interval_episodes`
- Special trajectory export gates:
  - `special_highcov_timeout_min_coverage`
  - `special_highcov_timeout_max_plots`
  - `special_long_success_gate_coverage`
  - `special_long_success_gate_window`
  - `special_long_success_min_length`
  - `special_long_success_percentile`
  - `special_long_success_max_plots`
  - `special_lowcov_gate_coverage`
  - `special_lowcov_gate_window`
  - `special_lowcov_absolute_threshold`
  - `special_lowcov_local_drop_margin`
  - `special_lowcov_max_plots`

Legacy compatibility fields:

- `enable_best_checkpoint_selection`
- `best_checkpoint_selection_start_env_steps`
- `best_checkpoint_selection_interval_env_steps`
- `best_checkpoint_validation_episodes`
- `best_checkpoint_topk_recheck`
- `best_checkpoint_recheck_episodes`
- `use_fixed_model_select_seeds`
- `fixed_model_select_seed_base`
- `formal_protocol=formal_best_checkpoint_v3`

Config snapshot classification:

- `training/formal_artifacts.py::ALLOWED_TUNING_FIELDS` currently contains:
  - `reward_turn_penalty_scale`
  - `reward_turn_weight_45`
  - `reward_turn_weight_90`
  - `reward_turn_weight_135`
  - `reward_turn_weight_180`
  - `reward_revisit_penalty`
- `training/formal_artifacts.py::MANUAL_REVIEW_FIELDS` currently contains:
  - `max_entries_per_block`
- `training/formal_artifacts.py::FROZEN_COMPARABILITY_FIELDS` contains many environment, protocol, replay, learner, exploration, reward, and state-cap fields. This is artifact metadata emitted by the training repository.

## Metric And Diagnostic Semantics

Neutral engineering semantics:

- `final_probe` is a held-out greedy evaluation output written to `logs/final_probe.csv`.
- Under `formal_posthoc_trainselect_v1`, `final_probe` rows are produced for the selected post-hoc top-k candidate checkpoints, and one row is marked as the winner.
- `winner_step` is the env-step value of the selected checkpoint recorded by the post-hoc final artifacts. It is not necessarily the terminal training step.
- `checkpoints/best.pt` is the promoted selected winner checkpoint payload.
- `checkpoints/last.pt` is the training endpoint checkpoint payload. It has held-out final-probe metrics only if the last checkpoint is among the selected post-hoc top-k candidates.
- `best_vs_last_gap_summary.json` records diagnostic differences between the selected winner and either a held-out last-candidate row or the recent train endpoint summary.
- `repeat_visit_ratio` is computed from cumulative visit counts as repeated visits divided by total visits.
- `stall_trigger_count` counts diagnostic stall-trigger events when consecutive zero-information steps reach `STALL_DIAGNOSTIC_WINDOW`.
- `zero_info_step_count` counts steps with zero newly observed empty and obstacle cells.
- `timeout_flag` records whether an episode ended at `max_episode_steps`.
- `value_*cap*` fields record value-state truncation and cap-hit diagnostics.
- Runtime and timing summaries describe operational behavior and profiling status.
- Generated plots and trajectories are optional visualization artifacts.
- Legacy `model_select_eval.csv`, `best_recheck_eval.csv`, historical final-probe lanes, historical channel layouts, and supplementary probes have different generation paths from the current default protocol.

## Runtime / Launch Modes

Confirmed launch modes:

- Standard CLI documented by the training README:
  - `python .\train_q_agent.py --device cuda`
- CPU smoke mode:
  - `python .\train_q_agent.py --smoke --device cpu`
- Profile mode:
  - `--profile` enables collector, learner, replay, state-adapter, cumulative-map, semantic, advantage-state, and value-state timing flags.
- Experimental fast CUDA mode:
  - `--fast-cuda` enables AMP, inference AMP, `torch.compile`, channels-last, cuDNN benchmark, and TF32.
- Strict reproducibility mode:
  - `--strict-reproducibility`
  - With `--no-deterministic-warn-only`, deterministic guard failures become hard failures where supported.
  - Stable launcher scripts set `PYTHONHASHSEED=0` and `CUBLAS_WORKSPACE_CONFIG=:4096:8`.
  - Stable launcher scripts disable AMP, inference AMP, compile, channels-last, TF32, and cuDNN benchmark.
- Stable same-seed A/B launch:
  - `scripts/launch_stable_same_seed_ab.ps1`
  - Runs A, checks A's reproducibility contract, then runs B if A and its contract check pass.

Confirmed nuance:

- `scripts/launch_formal_train_stable.ps1` exposes parameter defaults that differ from main CLI defaults:
  - `EpsilonDecaySteps = 320000` instead of CLI `epsilon_decay_steps = 400000`
  - `EpsilonEnd = 0.05` instead of CLI `epsilon_end = 0.03`
  - `MinReplaySize = 8000` instead of CLI `min_replay_size = 4000`

## Legacy And Non-Mainline Boundaries

The following are not part of the confirmed current algorithmic mainline:

- near/mid/token architecture files:
  - `encoders/local_encoder.py`
  - `encoders/global_encoder.py`
  - `heads/q_head.py`
  - `env/frontier_token_builder.py`
- legacy checkpoint-selection protocol:
  - `formal_best_checkpoint_v3`
  - `model_select_eval.csv`
  - `best_recheck_eval.csv`
  - `checkpoints/model_select/*.pt`
- historical output lanes and historical final-probe episode counts.
- supplementary checkpoint probes from `tools/supplementary_multi_checkpoint_probe.py`.
- local scheduler comparison logic under `scripts/experiment_scheduler.py` and `scripts/tuning/**`.
- historical/backfill tools under:
  - `tools/backfill_formal_run_artifacts.py`
  - `tools/generate_historical_baseline_summary.py`

The active mainline confirmed in this source inspection is the shared semantic dual-state architecture with `formal_posthoc_trainselect_v1`.

## Update Policy

This manifest becomes stale when source facts change in the training repository. Source-inspection updates are needed when any of the following change:

- training entry point or launch scripts
- `TrainConfig` fields or CLI flags
- current algorithmic mainline
- state tensor schema or channel layout
- shared semantic layer semantics
- model architecture, encoder, or decision head
- learning loop, replay, learner, evaluator, or exploration behavior
- reward and diagnostic event semantics
- formal protocol lifecycle
- checkpoint semantics for `best.pt`, `last.pt`, or candidate checkpoints
- generated artifact schemas or artifact names
- reproducibility contract fields
- legacy/current boundaries

Updates are source-inspection based. Runtime artifact existence and exact row contents remain per-run facts.

## Unverified Or Requires Future Audit

- No training run was executed for this manifest.
- No run outputs, checkpoints, model weights, full logs, full CSVs, plots, trajectories, or generated artifacts were copied or analyzed.
- Artifact names and writer paths were confirmed by source code, but actual per-run artifact existence and exact row contents remain run-specific.
- `outputs/` was intentionally not audited as evidence for this task.
- Optional plotting and trajectory output formats were not deeply audited.
- `scripts/tuning/**` and `scripts/experiment_scheduler.py` were inspected only enough to identify role and non-mainline boundary.
- Top-level non-mainline directories such as `demos/`, `docs/`, `run_picture/`, and `tmp/` were not audited as active training implementation.
- `tools/backfill_formal_run_artifacts.py` and `tools/generate_historical_baseline_summary.py` were not executed.
- Stable launcher scripts were inspected, but no stable launch or reproducibility contract check was run.
- Training README prose was inspected, but code-confirmed lifecycle facts take precedence in this manifest where README wording appears historical or legacy-oriented.
