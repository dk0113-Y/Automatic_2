---
name: training-analysis-archive
description: Archive paired training-analysis records into Automatic_2 history. For single-run archives, preserve Codex factual analysis JSON and a GPT-provided tuning_review.md digest, update history_index.json, and never reanalyze training outputs or provide tuning decisions.
---

# Training Analysis Archive

## 1. Purpose

This skill archives one paired single-run training-analysis record into `Automatic_2` history. A paired single-run record contains:

- source factual analysis JSON
- GPT-provided `tuning_review.md` digest

Pairing metadata is stored in `training_results/history/history_index.json`.

This skill does not reanalyze training outputs and does not provide tuning decisions.

## 2. Authority Boundary

- Source analysis JSON owns Codex factual report content.
- GPT-provided `tuning_review_md_digest` owns tuning rationale content.
- Codex validates and writes the supplied digest without summarizing, expanding, compressing, rewriting, reinterpreting, reanalyzing metrics, changing `recommendation_type`, or changing tuning meaning.
- Codex must not provide tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, method-level conclusions, or paper-level conclusions.

## 3. Required Input

Expected single-run task input:

- `archive_type`: `single_run`
- `archive_id`
- `source_analysis_json`
- `tuning_review_md_digest` supplied in the task prompt between marker lines:
  - `BEGIN_TUNING_REVIEW_MD_DIGEST`
  - `END_TUNING_REVIEW_MD_DIGEST`
- `history_index_path`, default `training_results/history/history_index.json`

For `single_run`:

- `source_analysis_json` normally points to `training_results/current/current_training_run_analysis.json`.
- Archived factual analysis output is `training_results/history/single_runs/<archive_id>/training_run_analysis.json`.
- Archived tuning review output is `training_results/history/single_runs/<archive_id>/tuning_review.md`.

If required inputs are missing or contradictory, report `blocked_insufficient_input` and do not invent records.

## 4. Archive Directory Contract

Single-run archive:

```text
training_results/history/single_runs/<archive_id>/training_run_analysis.json
training_results/history/single_runs/<archive_id>/tuning_review.md
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
- Preserve the source analysis content without reinterpretation.
- Copy the parsed JSON object to `training_run_analysis.json`.
- Do not add tuning rationale into the factual analysis JSON.
- Do not remove or rewrite source analysis fields unless explicitly authorized.
- Do not include private absolute paths if source already contains sanitized paths; do not add new private paths.

If `source_analysis_json` is invalid, report `parse_failed` and do not archive.

## 6. Tuning Review Markdown Digest Contract

`tuning_review_md_digest` is GPT-owned content supplied by the task prompt.

Extraction requirements:

- Extract digest content only from text between `BEGIN_TUNING_REVIEW_MD_DIGEST` and `END_TUNING_REVIEW_MD_DIGEST`.
- Both markers must appear exactly once and on their own unindented lines.
- Write only the Markdown between the markers to `tuning_review.md`.
- Do not write the `BEGIN_TUNING_REVIEW_MD_DIGEST` or `END_TUNING_REVIEW_MD_DIGEST` markers into `tuning_review.md`.
- Do not wrap the whole digest in a Markdown fenced code block.
- The digest may contain its own fenced PowerShell command block.

Preservation requirements:

- Preserve the supplied digest bytes as Markdown content after marker removal, except for normal file newline handling.
- Do not summarize, expand, compress, rewrite, or reinterpret the digest.
- Do not infer missing hypothesis, evidence, recommendation, command, or boundary content.
- If required content is missing or malformed, report `digest_validation_failed` with the specific issue and do not archive.

Required headings:

- `# GPT Tuning Review Digest`
- `## 1. Summary`
- `## 2. Evidence Admission`
- `## 3. Prior Validation`
- `## 4. Current Evidence Digest`
- `## 5. Next Action`
- `## 6. Boundaries`

Required fields:

- Summary: `source_run_name`, `source_archive_id`, `recommendation_type`, `prior_validation_status`, `recommended_next_run_name`, `decision_summary`
- Evidence Admission: `reproducible_launch_status`, `factual_summary_status`, `admission_result`, `note`
- Prior Validation: `prior_hypothesis`, `validation_status`, `judgement`, `key_evidence`, `remaining_uncertainty`
- Current Evidence Digest: `train_side_primary`, `posthoc_context`, `final_probe_supplemental`, `main_limitation`
- Next Action: `target_uncertainty`, `next_hypothesis`, `rationale`, `command`, `expected_validation_focus`
- Boundaries: `reference_baseline_note`, `final_probe_boundary`, `posthoc_boundary`, `redesign_boundary`

Enum requirements:

- `recommendation_type` must be one of:
  - `next_run_plan`
  - `hold_current_baseline`
  - `requires_more_evidence`
  - `repeat_or_repair_run`
  - `multi_run_comparison_needed`
  - `method_redesign_discussion_only`
- `prior_validation_status` and `validation_status` must be one of:
  - `supported`
  - `partially_supported`
  - `refuted`
  - `not_verifiable`
  - `not_applicable`

Command validation:

- If `recommendation_type` is `next_run_plan`, the digest must contain a fenced PowerShell command block.
- The launcher-only PowerShell command must start with `.\scripts\launch_formal_train_stable.ps1`.
- The command block must not include `cd <source_training_repo>;`, local absolute paths, or working-directory placeholders.
- The digest must not contain private local absolute paths.
- The digest must not request method-level redesign execution.

## 7. history_index.json Contract

`history_index.json` is the only pairing and index metadata file. If `history_index.json` does not exist, create it with:

```json
{
  "schema_version": "1.0",
  "index_type": "training_analysis_history_index",
  "entries": []
}
```

For single-run archive operations, append or update one compact entry:

```json
{
  "archive_id": "<archive_id>",
  "archive_type": "single_run",
  "run_name_or_group": "<run name>",
  "created_by": "codex",
  "tuning_review_generated_by": "gpt",
  "pairing_status": "paired",
  "training_analysis_path": "training_results/history/single_runs/<archive_id>/training_run_analysis.json",
  "tuning_review_path": "training_results/history/single_runs/<archive_id>/tuning_review.md",
  "source_current_analysis_path": "training_results/current/current_training_run_analysis.json",
  "source_report_type": "training_run_factual_analysis",
  "tuning_review_report_type": "gpt_tuning_review_digest",
  "recommendation_type": "<recommendation_type>",
  "prior_validation_status": "<prior_validation_status>",
  "recommended_next_run_name": "<recommended_next_run_name>",
  "validation": {
    "source_analysis_json_parse": "passed",
    "tuning_review_md_digest_validation": "passed",
    "no_training_repo_modification": true,
    "no_source_run_output_modification": true,
    "no_forbidden_artifacts_copied": true
  }
}
```

Requirements:

- Use repository-relative paths in `history_index.json`.
- Use `tuning_review_report_type: gpt_tuning_review_digest`.
- Use `validation.tuning_review_md_digest_validation: passed`.
- Do not store private absolute paths.
- Do not duplicate full digest content, key evidence, next hypothesis, validation focus, full command, full evidence details, historical trajectory, or current evidence interpretation in `history_index.json`.
- Record `recommendation_type`, `prior_validation_status`, and `recommended_next_run_name` only as compact values when present.
- If an entry with the same `archive_id` already exists, do not silently overwrite it.
- Existing archive replacement requires explicit prompt authorization.
- Without replacement authorization, report `archive_id_conflict` and stop.

## 8. Multi-Run Handling

Multi-run archive support is pending separate alignment. Do not introduce a multi-run Markdown digest contract in this skill. For `archive_type: multi_run`, require an explicitly supplied accepted contract or report `invalid_archive_type`.

## 9. Missingness And Conflict Handling

Use neutral statuses:

- `archive_ready`
- `archived`
- `blocked_insufficient_input`
- `parse_failed`
- `digest_validation_failed`
- `archive_id_conflict`
- `invalid_archive_type`
- `forbidden_scope_detected`

Stop without archiving when:

- Digest markers are missing, duplicated, indented, or malformed.
- Required headings or fields are missing.
- Digest enum values are invalid.
- Required launcher-only command validation fails.
- `source_analysis_json` is invalid.
- `archive_id` already exists and replacement is not explicitly authorized.
- Forbidden scope or artifact-copying risk is detected.

## 10. Forbidden Actions

- No training-output reanalysis.
- No training code modification.
- No source run output modification.
- No checkpoint, model weight, log, CSV, plot, trajectory, or binary artifact copying.
- No `archive_manifest.json` creation.
- No tuning recommendations by Codex.
- No next hyperparameters invented by Codex.
- No accepted-baseline decision.
- No stop/continue/branch decision.
- No method-level or paper-level conclusion.
- No rewriting GPT digest meaning.

## 11. Validation For Skill Use

When this skill is used in a real task, record:

- files read
- files written
- `source_analysis_json` parse check
- `tuning_review_md_digest` validation
- required Markdown headings and fields
- `recommendation_type` enum
- `prior_validation_status` enum
- `validation_status` enum
- launcher-only command check when applicable
- `history_index.json` update result
- `git diff` scope
- `git diff --check`
- `git diff --cached --check`
- training repository status if accessible
- source run output modification check if relevant
- forbidden artifact check
- no tuning decision by Codex

## 12. Suggested Final Report Fields

Final report should include:

- `archive_type`
- `archive_id`
- files written
- history index path
- source analysis preserved
- `tuning_review_md_digest` preserved
- digest validation result
- whether BEGIN/END markers were excluded from `tuning_review.md`
- whether no `archive_manifest.json` was created
- whether no forbidden artifacts were copied
- whether no tuning decision was introduced by Codex
- validation results
- commit/push status when execution task authorizes commit/push
