# Training System Manifest

## Document Role

This manifest is a control-plane engineering reference for GPT/Codex tuning work in `Automatic_2`. It summarizes the external DRL training implementation so future tuning analysis can reason from system design, state construction, model architecture, formal lifecycle, artifact schemas, and metric constraints.

This document is not a README clone, tuning report, run review, accepted-baseline decision, or recommendation document. It does not replace the real training code or real run artifacts.

Confirmed implementation facts are based on source inspection of the training repository. Items that were not fully proven by source inspection or by executing a run are listed under "Unverified Or Requires Future Audit".

## Source-of-Truth Boundary

`DRL-path-finding` / the local training repository remains the source of truth for:

- training implementation code
- command-line interface behavior
- runtime outputs
- logs
- checkpoints and model weights
- generated plots, trajectories, and other training artifacts
- exact artifact schemas at the time a run is produced

`Automatic_2` stores control-plane summaries and future sanitized reports only. It must not copy training source files, checkpoints, model weights, full logs, full CSV contents, plots, trajectories, or generated training artifacts.

The public training repository reference is `https://github.com/dk0113-Y/DRL-path-finding`. The local training checkout should still be inspected whenever implementation details may have changed.

## Current Algorithmic Mainline

Confirmed current mainline:

- Main training entry point: `train_q_agent.py`.
- Current policy network: `agents/q_value_agent.py::ExplorationQNetwork`.
- Current state adapter: `agents/q_value_agent.py::StateTensorAdapter`.
- Current state representation: shared semantic dual-state inputs:
  - `advantage_canvas`
  - `value_block_features`
  - `value_entry_features`
  - `value_block_mask`
  - `value_entry_mask`
- Current environment semantic layer: `env/shared_semantic_layer.py::SharedSemanticLayer`.
- Current decision architecture: dual-branch semantic dueling Q network:
  - advantage branch reads a local decision canvas
  - value branch reads an Accessible Unknown Block tree
  - `heads/semantic_dueling_head.py::SemanticDuelingHead` combines `V(s)` and `A(s,a)` into 8-action Q values
- Current learning algorithm: Double DQN with n-step transitions and uniform replay.

The active training path imports `AdvantageCanvasEncoder`, `ValueTreeEncoder`, and `SemanticDuelingHead`. It does not import the legacy near/mid/token encoder and split-head path for current formal training.

Confirmed legacy/non-mainline files include:

- `encoders/local_encoder.py`, explicitly documented as legacy near-map path.
- `encoders/global_encoder.py`, explicitly documented as legacy near/mid/token path.
- `heads/q_head.py`, explicitly documented as legacy split-input dueling head.
- `env/frontier_token_builder.py`, explicitly documented as legacy frontier-token builder.

## Training Entry Points

Confirmed training entry points:

- `train_q_agent.py`
  - Defines `TrainConfig`.
  - Defines CLI parsing through `parse_args()`.
  - Defines `build_system(cfg)`.
  - Defines `run_training(cfg, run_mode="cli")`.
  - Uses `if __name__ == "__main__"` as the CLI launch path.
- Documented formal CLI launch in the training README:
  - `python .\train_q_agent.py --device cuda`
- Smoke/debug launch:
  - `python .\train_q_agent.py --smoke --device cpu`
- Runtime profiling entry:
  - CLI `--profile` enables timing flags.
  - `build_profile_config()` provides a VSCode profiling preset.
- Experimental runtime acceleration entry:
  - CLI `--fast-cuda`.
  - `build_fast_cuda_config()` and related profile helpers.
  - These are runtime/performance paths, not algorithmic mainline changes.
- Stable reproducibility launch scripts:
  - `scripts/launch_formal_train_stable.ps1`
  - `scripts/launch_stable_same_seed_ab.ps1`
  - `scripts/check_reproducibility_contract.py`

Other local scripts/tools:

- `scripts/experiment_scheduler.py` and `scripts/tuning/**` implement a local sequential tuning scheduler with internal comparison rules. This is not an `Automatic_2` authority source and should not be treated as GPT decision authority without explicit task scope.
- `scripts/replay_posthoc_diagnostics.py` replays post-hoc candidate selection diagnostics from existing logs.
- `tools/supplementary_multi_checkpoint_probe.py` runs supplementary held-out checkpoint probes and includes compatibility support for historical 5/6/7 channel advantage layouts.
- `tools/backfill_formal_run_artifacts.py` and `tools/generate_historical_baseline_summary.py` are intended historical/backfill summary tools for existing outputs; they were not executed in this task.

## Repository Structure And Module Responsibilities

Confirmed source layout:

- `train_q_agent.py`
  - Main training assembly, CLI config, training loop, checkpoint lifecycle, post-hoc final selection, final probe, and formal artifact writing.
- `agents/`
  - `ExplorationQNetwork`, `StateTensorAdapter`, action masking helpers, greedy action selection.
- `encoders/`
  - Current mainline: `advantage_encoder.py`, `value_encoder.py`.
  - Legacy historical paths: `local_encoder.py`, `global_encoder.py`.
- `heads/`
  - Current mainline: `semantic_dueling_head.py`.
  - Legacy historical path: `q_head.py`.
- `env/`
  - Random map generation, grid topology, radar/local observation, cumulative belief map, shared semantic layer, advantage state builder, value state builder.
- `training/`
  - Collector, evaluator, replay buffer, DDQN learner, checkpointing, CSV logging, post-hoc selection, formal artifact writer, plotting, trajectory plotting, reward calculations.
- `scripts/`
  - Stable launches, reproducibility contract checking, post-hoc diagnostic replay, local experiment scheduler.
- `tools/`
  - Supplementary probes, historical backfill/generation, architecture/visualization helpers.
- `formal_artifacts/`
  - Contains tracked historical summary artifacts, not current training source code.
- `outputs/`
  - Generated run outputs. This is not source and should not be copied into `Automatic_2`.

Top-level `demos/`, `docs/`, `run_picture/`, `tmp/`, and generated/runtime directories were observed but were not audited as current formal training mainline in this task.

## Environment And State Construction Pipeline

Confirmed pipeline:

1. `RandomMapGenerator` builds bounded obstacle maps and valid 1x1 start states.
   - It adds border obstacles.
   - It creates rectangular obstacle structures.
   - It keeps the largest free component.
   - It can bind map generation to explicit seeds.
   - `compute_map_fingerprint()` records a compact map/start fingerprint.
2. `RadarSensor` defines the local observation footprint.
   - `scan_radius` controls local window shape `(2R+1, 2R+1)`.
   - Visibility uses disk-shaped line-of-sight templates.
   - The current sensor includes corner-occlusion logic controlled by `block_corner_peeking`.
3. `LocalObservationModel` renders local observations from the true map.
   - The policy does not receive the full true map.
   - The hot path uses `observe_fast()`.
4. `CumulativeBeliefMap` maintains agent-side knowledge.
   - Stores dynamic belief map, visit counts, frontier cache, coverage metrics, and analysis box.
   - Agent-side state construction uses cumulative belief (`self.map`, visits, frontier).
   - Effective coverage is a simulator-side metric and termination statistic.
   - The analysis box is a strict known-region bounding box, not a heuristic extension into unknown topology.
5. `SharedSemanticLayer` analyzes the cumulative belief map.
   - Extracts frontier clusters from the current frontier view.
   - Groups adjacent unknown components through frontier-first ownership.
   - Produces `UnknownBlock` objects as Accessible Unknown Blocks.
   - Attaches `FrontierCluster` children to each block.
   - Attaches `SupportGeometry` with local known-side obstacle-density summaries.
   - Exposes semantic metrics such as accessible block count, total accessible unknown area, frontier cluster count, and mean block area.
6. `AdvantageStateBuilder` builds the local decision canvas.
   - Canvas size is tied to the radar local observation window.
   - Confirmed channel layout:
     - `free`
     - `obstacle`
     - `frontier_block_area_map`
     - `visit_count_log_norm`
     - `recent_trajectory_decay`
   - It paints local frontier cluster cells with their parent block area ratio.
   - It includes cumulative revisit pressure and short-horizon trajectory decay.
7. `ValueStateBuilder` builds the block-tree tensor.
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
   - It emits diagnostic fields for truncation and cap-hit monitoring.

## Network / Encoding / Decision Head

Confirmed network mainline:

- `ExplorationQNetwork` validates feature dimensions against current builders before constructing modules.
- Advantage branch:
  - `AdvantageCanvasEncoder`
  - Convolutional residual backbone over the local decision canvas.
  - One action-specific state per move using directional spatial masks, landing-cell features, center features, and action embeddings.
  - Default action state dimension: `160`.
- Value branch:
  - `ValueTreeEncoder`
  - Encodes parent block tokens and child frontier-entry tokens.
  - Runs masked self-attention over sibling entries inside each block.
  - Uses parent-conditioned child aggregation.
  - Applies parent-grounded gated block updates.
  - Runs masked block self-attention across accessible blocks.
  - Uses learned state-query weighted pooling into the value state.
  - Default value state dimension: `192`.
- Decision head:
  - `SemanticDuelingHead`
  - Computes scalar `V(s)` from the value state.
  - Computes per-action `A(s,a)` from advantage states.
  - Outputs `Q(s,a) = V(s) + A(s,a) - mean_a A(s,a)`.
- Action space:
  - 8-neighbor grid movement from `env/grid_topology.py::ACTIONS_8`.
  - Legal action masks are applied during greedy selection and Double DQN target selection.

## Learning Pipeline

Confirmed learning pipeline:

- `TransitionCollector` owns environment interaction.
  - Resets episodes by generating a map/start, local observation, cumulative belief map, frontier, shared semantic snapshot, and initial state tensors.
  - Selects actions with epsilon-greedy policy over legal moves.
  - Random branches use Python `random`.
  - Greedy branches run model inference under legal-action masks.
  - Applies reward bookkeeping, coverage termination, timeout termination, stall diagnostics, zero-info diagnostics, repeat-visit metrics, and semantic monitoring.
  - Pushes transitions into n-step builder and replay buffer.
- `NStepTransitionBuilder`
  - Default `n_step=3`, `gamma=0.99`.
  - Builds n-step rewards and bootstrap discounts.
  - Flushes terminal prefixes at episode end.
- `ReplayBuffer`
  - Uniform replay by default.
  - Stores current state tensor structure plus action masks, action, reward, next state tensors, next action mask, done, and bootstrap discount.
  - `prioritized` is reserved but not active by default.
- `DDQNLearner`
  - Uses online and target networks.
  - Target action is selected by online Q over `next_action_mask`.
  - Target value is gathered from target network.
  - Uses Smooth L1 loss on TD error.
  - Uses Adam optimizer.
  - Uses hard target sync every `target_update_interval`.
  - Optional AMP affects runtime path, not metric definitions.
- `CSVMetricLogger`
  - Writes append-only CSVs:
    - `logs/train_episodes.csv`
    - `logs/train_steps.csv`
    - `logs/final_probe.csv`
    - legacy-compatible `logs/model_select_eval.csv`
    - legacy-compatible `logs/best_recheck_eval.csv`

## Formal Training Lifecycle

Confirmed current default protocol:

- `formal_protocol = formal_posthoc_trainselect_v1`.
- Budget modes:
  - `env_steps`: training stops at `total_env_steps`; warmup uses `warmup_steps`.
  - `episodes`: training stops at `total_train_episodes`; warmup uses `warmup_episodes`.
- Training loop:
  - Performs warmup collection.
  - Alternates rollout collection and learner updates.
  - Logs recent train snapshots by step or episode cadence.
  - Saves train-side periodic post-hoc checkpoints after `posthoc_candidate_start_env_steps` at `periodic_checkpoint_interval_env_steps`.
- Under the current post-hoc protocol:
  - No checkpoint validation runs during training.
  - No recheck runs during training.
  - No final probe runs during training.
  - Collector/learner are not paused for selection evaluation during training.
  - Periodic candidates are train-only checkpoint files matching `checkpoints/ckpt_step_<env_steps>.pt`.
- After training:
  - `checkpoints/last.pt` is saved as the diagnostic endpoint.
  - `training.posthoc_selection.select_posthoc_candidates()` reads train-side logs and candidate checkpoint filenames.
  - Candidate scoring uses windowed train metrics from `logs/train_steps.csv`.
  - Default score weights confirmed in code:
    - reward z-score: `0.35`
    - coverage z-score: `0.25`
    - success-rate z-score: `0.20`
    - episode-length z-score: `-0.10`
    - repeat-visit-ratio z-score: `-0.10`
  - Up to `posthoc_final_probe_topk` candidates receive held-out final probes.
  - Final winner ranking order is `success_rate`, then `coverage`, then `reward`.
  - The winner checkpoint is promoted to `checkpoints/best.pt`.
  - `best.pt` is the formal acceptance object.
  - `last.pt` remains diagnostic-only unless it was included in the post-hoc top-k candidates.
- Formal artifact writing:
  - `write_posthoc_final_artifacts()` writes post-hoc final selection JSON summaries.
  - `write_formal_run_artifacts()` writes metric, benchmark, config, reproducibility, artifact-index, and text-summary artifacts.

Legacy lifecycle:

- `formal_best_checkpoint_v3` remains a compatibility protocol.
- Legacy periodic validation writes `logs/model_select_eval.csv`.
- Legacy top-k recheck writes `logs/best_recheck_eval.csv`.
- Legacy model-selection flags should not be mixed with current post-hoc protocol evidence without explicit comparability review.

## Formal Artifact Interface

Future Codex evidence reports should inspect lightweight artifacts rather than model weights, full logs, plots, trajectories, or copied source.

Confirmed core artifacts:

| Artifact | Status | Intended contents and use |
| --- | --- | --- |
| `logs/metric_snapshot.json` | confirmed by code | Unified metric snapshot containing recent train, final probe, checkpoint selection summary, diagnostic last checkpoint summary, best-vs-last diagnostics, training dynamics summary, train-final consistency, and insufficient-evidence flags. |
| `logs/benchmark_summary.json` | confirmed by code | Runtime summary, run mode, runtime/performance switches, timing switch status, optional timing summaries, protocol revision, budget mode, best/last env steps, and runtime support evidence. |
| `logs/config_snapshot.json` | confirmed by code | Full serialized `TrainConfig`, git branch/SHA, comparability groups, runtime-only fields, observed run contract, evaluation contract, reward semantics, and structured artifact references. |
| `logs/reproducibility_contract.json` | confirmed by code | Launch/runtime determinism contract, environment variables, backend flags, seed policies, sanitized argv, and `contract_verdict`. |
| `logs/artifact_index.json` | confirmed by code | Inventory of expected CSVs, checkpoints, structured summaries, optional plots, and optional trajectories. It records existence and required/optional status. |
| `logs/posthoc_candidate_scores.csv` | confirmed by code | Train-window candidate scores, z-scores, selection ranks, candidate checkpoints, and validity flags for post-hoc selection. Lightweight CSV but should be inspected selectively, not copied wholesale into reports. |
| `logs/posthoc_selection_summary.json` | confirmed by code | Candidate range, candidate count, selected candidate steps, score weights, all candidate summaries, and diagnostic replay metadata. |
| `logs/final_probe_summary.json` | confirmed by code | Top-k held-out final-probe rows, winner row, winner checkpoint path, `best.pt` path, `last.pt` path, seed base, and episode count. |
| `logs/best_vs_last_gap_summary.json` | confirmed by code | Diagnostic gap between formal winner and last checkpoint, using either held-out last candidate if available or recent train endpoint summary. |
| `logs/formal_selection_manifest.json` | confirmed by code | Protocol name, candidate range, checkpoint interval, selected candidate steps, final probe episode count, seed base, winner step/checkpoint, `best.pt`, `last.pt`, and related artifact paths. |
| `logs/training_summary.txt` | confirmed by code | Human-readable lightweight summary built from metric and benchmark snapshots. |
| `logs/train_steps.csv` | confirmed by code | Recent train snapshots by env-step or final logging cadence. Source for training dynamics and post-hoc candidate scoring. |
| `logs/train_episodes.csv` | confirmed by code | Per-episode train records, including seeds/fingerprints when fixed train episode seeds are enabled. |
| `logs/final_probe.csv` | confirmed by code | Held-out final probe rows. Under current post-hoc protocol, includes top-k candidates and marks the formal winner. |

Checkpoint artifacts:

| Artifact | Status | Intended contents and use |
| --- | --- | --- |
| `checkpoints/best.pt` | confirmed by code | Formal primary checkpoint for the selected winner. Do not copy into `Automatic_2`. Inspect metadata only when explicitly needed and authorized. |
| `checkpoints/last.pt` | confirmed by code | Diagnostic endpoint checkpoint. Do not treat as formal winner unless selection artifacts prove it won or was promoted. Do not copy into `Automatic_2`. |
| `checkpoints/ckpt_step_<env_steps>.pt` | confirmed by code | Train-side post-hoc candidate checkpoints. Do not copy into `Automatic_2`. |
| `checkpoints/model_select/*.pt` | legacy-compatible | Legacy model-selection candidate checkpoints under `formal_best_checkpoint_v3`. Do not mix with current post-hoc evidence without comparability review. |

Legacy or optional artifacts:

| Artifact | Status | Intended contents and use |
| --- | --- | --- |
| `logs/model_select_eval.csv` | confirmed by code, legacy-compatible | Periodic validation rows for `formal_best_checkpoint_v3`; disabled in current post-hoc protocol. |
| `logs/best_recheck_eval.csv` | confirmed by code, legacy-compatible | Top-k recheck rows for `formal_best_checkpoint_v3`; disabled in current post-hoc protocol. |
| `logs/eval_metrics.csv` | documented by artifact index as legacy diagnostic | Historical/legacy diagnostic CSV if present. Not part of current default post-hoc training path. |
| `plots/**` | expected optional | Generated only when plot export is enabled. Supporting visualization only, not primary evidence. Do not copy into `Automatic_2` unless a future task explicitly allows sanitized visual artifacts. |
| `trajectories/**` | expected optional | Generated only when trajectory export is enabled. Do not copy into `Automatic_2` under this manifest task. |

## Tunable Configuration Surface

This section lists confirmed configuration fields and their engineering role. It does not recommend values.

Confirmed CLI/default fields from `TrainConfig` and `parse_args()`:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `budget_mode` / `--budget-mode` | `env_steps` | Current budget organization surface; `env_steps` and `episodes` are supported. |
| `total_env_steps` / `--total-env-steps` | `500000` | Formal env-step budget when `budget_mode=env_steps`. Comparability-sensitive. |
| `total_train_episodes` / `--total-train-episodes` | `600` | Episode budget when `budget_mode=episodes`. Comparability-sensitive. |
| `warmup_steps` / `--warmup-steps` | `4000` | Env-step warmup for `env_steps` mode. |
| `warmup_episodes` / `--warmup-episodes` | `0` | Episode warmup for `episodes` mode. |
| `batch_size` / `--batch-size` | `128` | Learner batch size. |
| `min_replay_size` / `--min-replay-size` | `4000` | Minimum replay size before learner updates. |
| `replay_capacity` / `--replay-capacity` | `100000` | Uniform replay capacity. |
| `n_step` / `--n-step` | `3` | n-step transition length. |
| `gamma` / `--gamma` | `0.99` | Discount factor. |
| `learning_rate` / `--learning-rate` | `1.0e-4` | Optimizer learning rate. |
| `target_update_interval` / `--target-update-interval` | `1000` | Hard target sync interval. |
| `epsilon_start` / `--epsilon-start` | `1.0` | Linear epsilon schedule start. |
| `epsilon_end` / `--epsilon-end` | `0.03` | Linear epsilon schedule end. |
| `epsilon_decay_steps` / `--epsilon-decay-steps` | `400000` | Linear epsilon schedule duration. |
| `rows` / `--rows` | `40` | Map rows. |
| `cols` / `--cols` | `60` | Map columns. |
| `obs_size` / `--obs-size` | `6` | Obstacle-generation size parameter. |
| `scan_radius` / `--scan-radius` | `10` | Radar radius and advantage-canvas local scale. |
| `obstacle_ratio` / `--obstacle-ratio` | `0.20` | Map obstacle density target. |
| `max_accessible_blocks` / `--max-accessible-blocks` | `16` | Value-state block cap. |
| `max_entries_per_block` / `--max-entries-per-block` | `8` | Value-state child-entry cap; config snapshot marks this as manual-review field. |
| `reward_info_scale` / `--reward-info-scale` | `3.0` | Weighted information-gain reward scale. |
| `reward_obstacle_weight` / `--reward-obstacle-weight` | `0.25` | Obstacle reveal weighting inside information gain. |
| `reward_step_penalty` / `--reward-step-penalty` | `0.02` | Per-step penalty. |
| `reward_terminal_bonus` / `--reward-terminal-bonus` | `20.0` | Coverage-success terminal bonus. |
| `reward_revisit_penalty` / `--reward-revisit-penalty` | `0.10` | Recent revisit penalty; config snapshot marks this as allowed tuning field. |
| `reward_turn_penalty_scale` / `--reward-turn-penalty-scale` | `0.05` | Total turn penalty scale; config snapshot marks this as allowed tuning field. |
| `reward_turn_weight_45` / `--reward-turn-weight-45` | `0.0` | Turn angle weight; config snapshot marks this as allowed tuning field. |
| `reward_turn_weight_90` / `--reward-turn-weight-90` | `1/3` | Turn angle weight; config snapshot marks this as allowed tuning field. |
| `reward_turn_weight_135` / `--reward-turn-weight-135` | `2/3` | Turn angle weight; config snapshot marks this as allowed tuning field. |
| `reward_turn_weight_180` / `--reward-turn-weight-180` | `1.0` | Turn angle weight; config snapshot marks this as allowed tuning field. |
| `reward_timeout_penalty` / `--reward-timeout-penalty` | `8.0` | Timeout penalty. |

Current formal protocol surface:

| Field / flag | Confirmed default | Role |
| --- | ---: | --- |
| `formal_protocol` / `--formal-protocol` | `formal_posthoc_trainselect_v1` | Current formal protocol lane. |
| `final_greedy_episodes` / `--final-greedy-episodes` | `100` | Held-out final-probe episode count. Comparability-sensitive. |
| `use_fixed_train_episode_seeds` / `--use-fixed-train-episode-seeds` | `true` | Binds train episode index to fixed seed sequence. |
| `fixed_train_episode_seed_base` / `--fixed-train-episode-seed-base` | `20259323` | Train seed base. |
| `use_fixed_eval_seeds` / `--use-fixed-eval-seeds` | `true` | Legacy-named toggle controlling final-probe fixed seeds. |
| `fixed_final_probe_seed_base` / `--fixed-final-probe-seed-base` | `20261323` | Final-probe held-out seed base. |
| `periodic_checkpoint_interval_env_steps` / `--periodic-checkpoint-interval-env-steps` | `20000` | Post-hoc train-side candidate interval. |
| `posthoc_candidate_start_env_steps` / `--posthoc-candidate-start-env-steps` | `200000` | Start of candidate range. |
| `posthoc_candidate_end_env_steps` / `--posthoc-candidate-end-env-steps` | `0` | Candidate range end; `0` means use total env steps. |
| `posthoc_selection_window_env_steps` / `--posthoc-selection-window-env-steps` | `40000` | Train-window width for post-hoc scoring. |
| `posthoc_final_probe_topk` / `--posthoc-final-probe-topk` | `3` | Maximum selected candidates to final-probe. |

Runtime/debug surface:

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
  - stable scripts set `PYTHONHASHSEED=0` and `CUBLAS_WORKSPACE_CONFIG=:4096:8`
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
- Debug/smoke flags:
  - `debug_check_incremental_frontier`
  - `smoke`
  - stdout interval flags such as `episode_print_interval` and `train_print_interval`

Legacy compatibility surface:

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
- `MANUAL_REVIEW_FIELDS` currently contains:
  - `max_entries_per_block`
- Many fields listed above are present CLI/config fields but are frozen/comparability-sensitive in `FROZEN_COMPARABILITY_FIELDS`. Changing them can move a run into a different comparability group and requires explicit review.

## Metric Interpretation Constraints For GPT

GPT review should use metrics in the context of the implementation described above.

- `final_probe` must not be interpreted in isolation from recent train support, train-final consistency, candidate-selection facts, and formal selection artifacts.
- High reward alone is insufficient if coverage, `success_rate`, repeat visit ratio (`RVR` / `repeat_visit_ratio`), episode length, stall indicators, zero-info indicators, or timeout indicators conflict.
- Lower `repeat_visit_ratio` is useful only when coverage, success, and episode efficiency remain healthy in the same evidence context.
- A strong post-hoc winner needs support from:
  - `logs/posthoc_selection_summary.json`
  - `logs/posthoc_candidate_scores.csv`
  - `logs/final_probe_summary.json`
  - `logs/best_vs_last_gap_summary.json`
  - `logs/formal_selection_manifest.json`
- `winner_step` must be interpreted as the selected checkpoint step, not automatically as the best training endpoint.
- `last.pt` is diagnostic-only unless selection artifacts prove it was in top-k and won or was promoted.
- `best_vs_last_gap_summary` is diagnostic support for winner-vs-training-end behavior. It is not by itself a tuning decision.
- Runtime and timing summaries are supporting operational evidence. They are not evidence of method-performance superiority.
- `stall_trigger_count`, `zero_info_step_count`, and `timeout_flag` are diagnostic constraints that can invalidate simple reward-only interpretations.
- Legacy `model_select_eval.csv`, `best_recheck_eval.csv`, historical final-probe lanes, historical channel layouts, and supplementary probes must not be mixed with current `formal_posthoc_trainselect_v1` outputs without explicit comparability review.
- Generated plots and trajectories may help inspect failure modes, but they should not replace structured metric and selection artifacts.
- A parameter setting should not be judged as good or bad from this manifest alone. Real run artifacts and current code inspection remain required for tuning decisions.

## Runtime / Launch Modes

Confirmed launch modes:

- Standard formal CLI documented by the training README:
  - `python .\train_q_agent.py --device cuda`
- CPU smoke mode:
  - `python .\train_q_agent.py --smoke --device cpu`
- Profile mode:
  - `--profile` enables collector, learner, replay, state-adapter, cumulative-map, semantic, advantage-state, and value-state timing flags.
- Experimental fast CUDA mode:
  - `--fast-cuda` enables AMP, inference AMP, torch compile, channels-last, cuDNN benchmark, and TF32.
  - Code and README mark this as an experimental runtime path, not the default formal mainline.
- Strict reproducibility mode:
  - `--strict-reproducibility`
  - With `--no-deterministic-warn-only`, strict runtime guard failures become hard failures where supported.
  - Stable launcher scripts also set required environment variables and disable AMP, inference AMP, compile, channels-last, TF32, and cuDNN benchmark.
- Stable same-seed A/B launch:
  - `scripts/launch_stable_same_seed_ab.ps1`
  - Runs A, checks A's reproducibility contract, then runs B only if A passes.

Important confirmed nuance:

- The stable launcher script exposes parameter defaults that differ from the main CLI defaults for some fields, including `epsilon_decay_steps`, `epsilon_end`, and `min_replay_size`. These are script defaults, not new recommendations in this manifest.

## Legacy And Non-Mainline Boundaries

Do not treat the following as current algorithmic mainline without fresh code verification:

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
- local scheduler decisions or recipe comparisons under `scripts/experiment_scheduler.py` and `scripts/tuning/**`.

The active mainline confirmed in this audit is the shared semantic dual-state architecture with `formal_posthoc_trainselect_v1`.

## GPT Tuning Usage Rules

GPT should use this manifest as:

- engineering context for tuning analysis
- a map of which source modules define state, model, learning, and artifact semantics
- a map of which lightweight artifacts to ask Codex to inspect
- a guardrail against metric-only reasoning
- a boundary document separating `Automatic_2` control-plane summaries from the training repository source of truth

GPT must still ask Codex to inspect the real training repository when:

- code, CLI flags, artifact schema, or protocol semantics may have changed
- a run output is inconsistent with this manifest
- a proposed tuning change depends on implementation details
- a proposed change touches state construction, model architecture, semantic layer, reward semantics, formal lifecycle, or artifact writers
- legacy and current outputs may be compared
- method-level redesign is requested
- exact command defaults matter for a run decision

This manifest does not replace real code, real checkpoints, real logs, or real run artifacts.

## Update Policy

Update this manifest whenever any of the following change in the training repository:

- training entry point or launch scripts
- `TrainConfig` fields or CLI flags
- current algorithmic mainline
- state tensor schema or channel layout
- shared semantic layer semantics
- model architecture, encoder, or decision head
- learning loop, replay, learner, or exploration behavior
- formal protocol lifecycle
- checkpoint semantics for `best.pt`, `last.pt`, or candidate checkpoints
- formal artifact schemas or artifact names
- reproducibility contract expectations
- legacy/current comparability boundaries

The update should be based on source inspection, not memory.

## Unverified Or Requires Future Audit

- No training run was executed for this manifest.
- No run outputs, checkpoints, model weights, full logs, full CSVs, plots, trajectories, or generated artifacts were copied or analyzed.
- Artifact names and writer paths were confirmed by source code, but actual per-run artifact existence and exact row contents must be verified from each completed run.
- `outputs/` was intentionally not audited as evidence for this task.
- Optional plotting and trajectory output formats were not deeply audited because they are not primary formal evidence.
- `scripts/tuning/**` and `scripts/experiment_scheduler.py` were inspected only enough to identify their role and non-authority boundary; their internal decision recipes were not adopted into this manifest.
- Top-level non-mainline directories such as `demos/`, `docs/`, `run_picture/`, and `tmp/` were not audited as active formal training implementation.
- The stable-launch script defaults were inspected, but no stable launch or reproducibility contract check was run.
