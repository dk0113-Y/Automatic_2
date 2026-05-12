# GPT-to-Codex Prompting

Use this file to generate minimal English Codex task prompts that invoke the selected skill and pass task-instance fields.

## 1. Prompt Fields

Include only task-instance fields: Task title; Task type; Skill invocation; Working context; Inputs; Outputs; Writes; Validation; Commit/push; Final report.

Rules:
- Copy Skill invocation exactly from the selected task interface.
- Use the selected skill for execution details.
- Keep prompts short; do not restate skill procedures, extraction contracts, schema contracts, or artifact lists.
- Writes define task-instance write scope.
- Validation points to the selected skill contract plus task-instance diff scope and commit/push checks.
- Commit and push after validation passes and only allowed files changed, unless explicitly disabled.
- Final report includes files changed, validation result, commit/push status, commit hash, push target, and remaining risks.

### Final Prompt Format

When GPT outputs a Codex prompt, output exactly one `text` fenced block and no prose outside it. Keep the final Codex prompt operational only; do not include copy-safety explanations, self-check lists, invalid-path examples, or valid/invalid path demonstrations. Inside that block, do not use nested fenced code blocks or literal triple-backtick sequences. Write shell/PowerShell commands as indented plain text. Preserve Windows paths exactly, especially `Automatic_2\.agents`. For digest tasks, keep `BEGIN_TUNING_REVIEW_MD_DIGEST` and `END_TUNING_REVIEW_MD_DIGEST` on their own unindented lines and write command lines as indented plain text inside the digest.

## 2. Task Interfaces

### 2.1 training_run_factual_analysis_task
Skill invocation:
  Use [$training-run-factual-analysis](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-run-factual-analysis\SKILL.md)
Inputs: `source_run_dir`; `run_name`
Outputs: `training_results/current/current_training_run_analysis.json`
Writes: current factual analysis JSON only
Validation: selected_skill_contract; json_parse; required_top_level_keys; factual_summary_status; tuning_recommendation_provided_false; diff_scope; commit_push

### 2.2 single_run_analysis_archive_task
Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)
Inputs: current factual JSON; GPT-provided strict `tuning_review_md_digest` generated from the immediately preceding GPT tuning review answer; `archive_id`
Outputs: single-run archive factual JSON; single-run archive `tuning_review.md`; `training_results/history/history_index.json`
Writes: archive pair plus history index
Validation: selected_skill_contract; source_json; tuning_review_md_digest; archive_id; history_index; diff_scope; commit_push
Digest block rule: Include the supplied digest between the required `BEGIN_TUNING_REVIEW_MD_DIGEST` and `END_TUNING_REVIEW_MD_DIGEST` markers owned by Final Prompt Format.
Digest rule: Codex preserves the supplied GPT digest without re-summarizing, reanalyzing metrics, changing `recommendation_type`, or changing tuning meaning.

### 2.3 multi_run_analysis_archive_task
Skill invocation:
  Use [$training-analysis-archive](C:\Users\Dk\Desktop\SCI\Automatic_2\.agents\skills\training-analysis-archive\SKILL.md)
Inputs: current multi-run factual JSON; `tuning_review_payload` or `tuning_review_json_source`; `archive_id`
Outputs: multi-run archive factual JSON; multi-run archive tuning review JSON; `training_results/history/history_index.json`
Writes: multi-run archive pair plus history index
Validation: selected_skill_contract; source_json; tuning_review_json; archive_id; history_index; portable command metadata labels `source_training_repo`, `<source_training_repo>`, `command_arguments`; diff_scope; commit_push

### 2.4 multi_run_script_generation_task
Reserved skill: `$multi-run-script-generation`
Expected skill file: `.agents/skills/multi-run-script-generation/SKILL.md`
Inputs: GPT parameter plan; script or command target; execution constraints
Outputs: generated script or launch-command file
Writes: explicitly authorized target only
Validation: future_skill_contract; diff_scope; commit_push

### 2.5 multi_run_factual_analysis_task
Reserved skill: `$multi-run-factual-analysis`
Expected skill file: `.agents/skills/multi-run-factual-analysis/SKILL.md`
Inputs: source run entries; run labels; comparison scope
Outputs: `training_results/current/current_multi_run_analysis.json`
Writes: current multi-run factual analysis JSON only
Validation: future_skill_contract; per_run_reproducibility; json_parse; diff_scope; commit_push

## 3. Update Policy

- Reserved interfaces become callable after an accepted local `SKILL.md` exists and the interface records the concrete Skill invocation.
- Keep interface shape stable.
- Callable skill invocations use the exact local Markdown-link recorded in the task interface.
