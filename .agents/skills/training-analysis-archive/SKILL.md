---
name: training-analysis-archive
description: Archive paired training-analysis records into Automatic_2 history. For single-run archives, preserve Codex factual analysis JSON and a GPT-provided tuning_review.md digest, update history_index.json, and never reanalyze training outputs or provide tuning decisions.
---

# Training Analysis Archive

## 1. Purpose

- Archive paired training-analysis records into `Automatic_2` history.
- Preserve Codex factual JSON and GPT Markdown tuning digest.
- Update `training_results/history/history_index.json`.
- Do not reanalyze training outputs or provide tuning decisions.

## 2. Inputs

| Field | Required | Contract |
| --- | --- | --- |
| `archive_type` | yes | `single_run` or `multi_run` |
| `archive_id` | yes | Unique history archive id |
| `source_analysis_json` | yes | Path or supplied JSON object |
| `tuning_review_md_digest` | single_run | GPT digest between `BEGIN_TUNING_REVIEW_MD_DIGEST` and `END_TUNING_REVIEW_MD_DIGEST` |
| `history_index_path` | no | Default: `training_results/history/history_index.json` |

Single-run:
- Expected source `report_type`: `training_run_factual_analysis`
- Expected current source path: `training_results/current/current_training_run_analysis.json`

Multi-run: unchanged; pending separate alignment. Do not expand or redesign the multi-run archive contract in this skill.

## 3. Outputs

Single-run output allowlist:
- `training_results/history/single_runs/<archive_id>/training_run_analysis.json`
- `training_results/history/single_runs/<archive_id>/tuning_review.md`
- `training_results/history/history_index.json`

Multi-run: unchanged; pending separate alignment.

Never output:
- `archive_manifest.json`
- Raw artifacts, checkpoints, model weights, full logs, full CSVs, plots, trajectories, or binary artifacts

## 4. Execution Steps

1. Validate `archive_type` and required inputs.
2. Check `archive_id` conflict.
3. Parse `source_analysis_json`.
4. Validate source `report_type`.
5. Extract `tuning_review_md_digest` between `BEGIN/END` markers for `single_run`.
6. Validate digest headings, fields, enums, and command.
7. Create archive directory.
8. Write preserved `training_run_analysis.json`.
9. Write `tuning_review.md` from marker body only.
10. Append/update compact `history_index` entry.
11. Validate diff scope and forbidden artifacts.
12. Report result.

## 5. Digest Validation

Marker rules:

| Rule | Required |
| --- | --- |
| `BEGIN_TUNING_REVIEW_MD_DIGEST` exists | yes |
| `END_TUNING_REVIEW_MD_DIGEST` exists | yes |
| Markers are on their own unindented lines | yes |
| Exactly one digest block | yes, unless task explicitly authorizes otherwise |
| Whole digest is not wrapped in a Markdown fenced code block | yes |
| Literal triple-backtick sequences inside digest | no |

Preservation rules:
- Extract only marker body.
- Exclude `BEGIN/END` markers from `tuning_review.md`.
- Preserve GPT digest meaning.
- Do not summarize, expand, compress, rewrite, reinterpret, reanalyze, change `recommendation_type`, or invent hypotheses, evidence, commands, validation focus, or tuning decisions.

Required headings:
- `# GPT Tuning Review Digest`
- `## 1. Summary`
- `## 2. Evidence Admission`
- `## 3. Prior Validation`
- `## 4. Current Evidence Digest`
- `## 5. Next Action`
- `## 6. Boundaries`

Required fields:

| Section | Fields |
| --- | --- |
| Summary | `source_run_name`, `source_archive_id`, `recommendation_type`, `prior_validation_status`, `recommended_next_run_name`, `decision_summary` |
| Evidence Admission | `reproducible_launch_status`, `factual_summary_status`, `admission_result`, `note` |
| Prior Validation | `prior_hypothesis`, `validation_status`, `judgement`, `key_evidence`, `remaining_uncertainty` |
| Current Evidence Digest | `train_side_primary`, `posthoc_context`, `final_probe_supplemental`, `main_limitation` |
| Next Action | `target_uncertainty`, `next_hypothesis`, `rationale`, `command`, `expected_validation_focus` |
| Boundaries | `reference_baseline_note`, `final_probe_boundary`, `posthoc_boundary`, `redesign_boundary` |

Enum validation:
- `recommendation_type`: `next_run_plan`, `hold_current_baseline`, `requires_more_evidence`, `repeat_or_repair_run`, `multi_run_comparison_needed`, `method_redesign_discussion_only`
- `prior_validation_status`, `validation_status`: `supported`, `partially_supported`, `refuted`, `not_verifiable`, `not_applicable`

Command validation when `recommendation_type = next_run_plan`:
- The `command` field contains a launcher command line written as plain text or indented plain text.
- Command starts with `.\scripts\launch_formal_train_stable.ps1`.
- Command does not include `cd <source_training_repo>;`, local absolute paths, working-directory placeholders, or literal triple-backtick sequences.

Safety validation:
- No private local absolute paths.
- No method-level redesign execution request.
- No full logs, CSVs, checkpoints, weights, plots, trajectories, or binary artifact references as files to copy.

## 6. history_index Contract

Required index root if missing:

```json
{
  "schema_version": "1.0",
  "index_type": "training_analysis_history_index",
  "entries": []
}
```

Single-run entry fields:

| Field | Value |
| --- | --- |
| `archive_id` | `<archive_id>` |
| `archive_type` | `single_run` |
| `run_name_or_group` | `<run name>` |
| `created_by` | `codex` |
| `tuning_review_generated_by` | `gpt` |
| `pairing_status` | `paired` |
| `training_analysis_path` | `training_results/history/single_runs/<archive_id>/training_run_analysis.json` |
| `tuning_review_path` | `training_results/history/single_runs/<archive_id>/tuning_review.md` |
| `source_current_analysis_path` | `training_results/current/current_training_run_analysis.json` |
| `source_report_type` | `training_run_factual_analysis` |
| `tuning_review_report_type` | `gpt_tuning_review_digest` |
| `recommendation_type` | `<recommendation_type>` |
| `prior_validation_status` | `<prior_validation_status>` |
| `recommended_next_run_name` | `<recommended_next_run_name>` |
| `validation.source_analysis_json_parse` | `passed` |
| `validation.tuning_review_md_digest_validation` | `passed` |
| `validation.no_training_repo_modification` | `true` |
| `validation.no_source_run_output_modification` | `true` |
| `validation.no_forbidden_artifacts_copied` | `true` |

Index compactness:
- Use repository-relative paths only.
- Store no private absolute paths.
- Do not duplicate digest content.
- Do not store `key_evidence`, `next_hypothesis`, `expected_validation_focus`, full command, full evidence details, historical trajectory, or current evidence interpretation.
- Stop on existing `archive_id` unless explicit replacement authorization exists.

## 7. Stop Conditions

Statuses:
- `blocked_insufficient_input`
- `parse_failed`
- `digest_validation_failed`
- `archive_id_conflict`
- `invalid_archive_type`
- `forbidden_scope_detected`

Rules:
- Stop on missing required input.
- Stop on invalid `source_analysis_json`.
- Stop on malformed, missing, indented, duplicated, or invalid digest markers.
- Stop on missing required headings or fields, or invalid enum values.
- Stop on `archive_id` conflict unless explicit replacement authorization exists.
- Stop on forbidden write scope or forbidden artifact copying.

## 8. Forbidden Actions

- No training-output reanalysis.
- No training code modification.
- No source run output modification.
- No checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts copied.
- No `archive_manifest.json`.
- No tuning recommendations by Codex.
- No next hyperparameters invented by Codex.
- No accepted-baseline decision.
- No stop/continue/branch decision.
- No method-level or paper-level conclusion.
- No rewriting GPT digest meaning.

## 9. Final Report

Report fields:
- `archive_type`
- `archive_id`
- files written
- `history_index_path`
- `source_analysis_preserved`
- `tuning_review_md_digest_preserved`
- `digest_validation_result`
- `markers_excluded_from_tuning_review_md`
- `no_archive_manifest_created`
- `no_forbidden_artifacts_copied`
- `no_tuning_decision_by_codex`
- `validation_results`
- `commit_push_status`
- `commit_hash`
- `push_target`
- `remaining_risks`
