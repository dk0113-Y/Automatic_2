---
name: training-analysis-archive
description: Archive paired training-analysis records into Automatic_2 history. Preserve Codex factual analysis and GPT tuning-review JSON as a matched pair, update history_index.json, and never reanalyze training outputs or provide tuning decisions.
---

# Training Analysis Archive

## 1. Purpose

This skill archives one paired training-analysis record into `Automatic_2` history. A paired record contains:

- source factual analysis JSON
- GPT tuning-review JSON

Pairing metadata is stored in `training_results/history/history_index.json`.

This skill does not reanalyze training outputs and does not provide tuning decisions.

## 2. Authority Boundary

- Source analysis JSON owns Codex factual report content.
- GPT-provided tuning review payload owns tuning rationale content.
- Codex preserves and validates these records but does not reinterpret them.
- Codex may normalize the tuning-review payload into valid JSON shape only when instructed.
- Codex must not change the meaning of GPT's tuning rationale.
- Codex must not provide tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, or paper-level conclusions.

## 3. Required Input

Expected task input:

- `archive_type`: `single_run` or `multi_run`
- `archive_id`
- `source_analysis_json`
- `tuning_review_payload` or `tuning_review_json_source`
- `history_index_path`, default `training_results/history/history_index.json`

For `single_run`:

- `source_analysis_json` normally points to `training_results/current/current_training_run_analysis.json`.
- Archived factual analysis output is `training_results/history/single_runs/<archive_id>/training_run_analysis.json`.
- Archived tuning review output is `training_results/history/single_runs/<archive_id>/tuning_review.json`.

For `multi_run`:

- `source_analysis_json` normally points to `training_results/current/current_multi_run_analysis.json`.
- Archived factual analysis output is `training_results/history/multi_runs/<archive_id>/multi_run_analysis.json`.
- Archived tuning review output is `training_results/history/multi_runs/<archive_id>/tuning_review.json`.

If required inputs are missing or contradictory, report `blocked_insufficient_input` and do not invent records.

## 4. Archive Directory Contract

Single-run archive:

```text
training_results/history/single_runs/<archive_id>/training_run_analysis.json
training_results/history/single_runs/<archive_id>/tuning_review.json
```

Multi-run archive:

```text
training_results/history/multi_runs/<archive_id>/multi_run_analysis.json
training_results/history/multi_runs/<archive_id>/tuning_review.json
```

Global index:

```text
training_results/history/history_index.json
```

- Do not create `archive_manifest.json`.
- Per-archive directories contain content files only.
- Pairing metadata belongs to `history_index.json`.

## 5. Source Analysis Preservation

Required handling:

- Parse `source_analysis_json`.
- Verify `report_type` is compatible with `archive_type`.
- For `single_run`, expect `report_type` to be `training_run_factual_analysis`.
- For `multi_run`, validate the source report type against the declared multi-run analysis contract supplied by the task prompt.
- Preserve the source analysis content without reinterpretation.
- Copy the parsed JSON object to the archive content file.
- Do not add tuning rationale into the factual analysis JSON.
- Do not remove or rewrite source analysis fields unless explicitly authorized.
- Do not include private absolute paths if source already contains sanitized paths; do not add new private paths.

If `source_analysis_json` is invalid, report `parse_failed` and do not archive.

## 6. Tuning Review Payload Contract

`tuning_review_payload` is GPT-owned content supplied by the prompt or by a JSON source file.

Required minimum `tuning_review.json` shape:

```json
{
  "schema_version": "1.0",
  "report_type": "gpt_tuning_review",
  "generated_by": "gpt",
  "review_scope": "single_run_or_multi_run",
  "source_analysis_archive_id": "<archive_id>",
  "source_run_name_or_group": "<name>",
  "recommendation_type": "<next_run_plan|hold|requires_more_evidence|other>",
  "recommended_next_command_summary": {
    "working_directory": "source_training_repo",
    "launcher": "scripts/launch_formal_train_stable.ps1",
    "run_name": "<run name>",
    "stable_reproducible_mode_required": true,
    "command_template": "cd <source_training_repo>; .\\scripts\\launch_formal_train_stable.ps1 <arguments>",
    "command_arguments": {}
  },
  "tuning_commentary": "",
  "rationale": [],
  "expected_validation_focus": [],
  "limitations": []
}
```

Requirements:

- The final `tuning_review.json` must be valid JSON.
- Preserve GPT-provided tuning rationale.
- Do not judge whether the tuning rationale is correct.
- Do not add new tuning recommendations beyond the supplied payload.
- If optional fields are absent, keep the JSON minimal rather than inventing content.
- The skill may add `archive_id` and source path metadata when needed, but must not change the tuning meaning.
- Use portable repository labels or repository-relative paths for command metadata.
- Use `working_directory: source_training_repo` when command metadata references the external training repository.
- Use `<source_training_repo>` as the placeholder for the external training repository when a command template is recorded.
- Use `command_arguments` for concrete launcher parameters.
- Do not store private local absolute paths in tracked tuning-review history.

## 7. history_index.json Contract

`history_index.json` is the only pairing and index metadata file. If `history_index.json` does not exist, create it with:

```json
{
  "schema_version": "1.0",
  "index_type": "training_analysis_history_index",
  "entries": []
}
```

Each archive operation must append or update one entry with:

```json
{
  "archive_id": "<archive_id>",
  "archive_type": "single_run|multi_run",
  "run_name_or_group": "<run name or group id>",
  "created_by": "codex",
  "tuning_review_generated_by": "gpt",
  "pairing_status": "paired",
  "training_analysis_path": "<path to archived analysis JSON>",
  "tuning_review_path": "<path to archived tuning_review.json>",
  "source_current_analysis_path": "<source_analysis_json>",
  "source_report_type": "<report_type>",
  "tuning_review_report_type": "gpt_tuning_review",
  "recommendation_type": "<recommendation_type if present>",
  "recommended_next_run_name": "<if present>",
  "validation": {
    "source_analysis_json_parse": "passed",
    "tuning_review_json_parse": "passed",
    "no_training_repo_modification": true,
    "no_source_run_output_modification": true,
    "no_forbidden_artifacts_copied": true
  }
}
```

Requirements:

- Use repository-relative paths in `history_index.json`.
- Do not store private absolute paths.
- Do not duplicate full report content inside `history_index.json`.
- If an entry with the same `archive_id` already exists, do not silently overwrite it.
- Existing archive replacement requires explicit prompt authorization.
- Without replacement authorization, report `archive_id_conflict` and stop.

## 8. Missingness And Conflict Handling

Use neutral statuses:

- `archive_ready`
- `archived`
- `blocked_insufficient_input`
- `parse_failed`
- `archive_id_conflict`
- `invalid_archive_type`
- `forbidden_scope_detected`

Report missing files, invalid JSON, duplicate `archive_id`, and forbidden path risks.

## 9. Forbidden Actions

- No training-output reanalysis.
- No training code modification.
- No source run output modification.
- No checkpoint, model weight, log, CSV, plot, or trajectory copying.
- No `archive_manifest.json` creation.
- No tuning recommendations.
- No next hyperparameters invented by Codex.
- No accepted-baseline decision.
- No stop/continue/branch decision.
- No method-level or paper-level conclusion.

## 10. Validation For Skill Use

When this skill is used in a real task, record:

- files read
- files written
- JSON parse checks
- `history_index.json` update result
- `git diff` scope
- `git diff --check`
- `git diff --cached --check`
- training repository status if accessible
- source run output modification check if relevant
- forbidden artifact check
- no tuning decision by Codex

## 11. Suggested Final Report Fields

Final report should include:

- `archive_type`
- `archive_id`
- files written
- history index path
- validation results
- whether source analysis was preserved
- whether tuning-review payload was preserved
- whether no `archive_manifest.json` was created
- whether no forbidden artifacts were copied
- whether no tuning decision was introduced by Codex
