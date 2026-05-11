# Document Authoring Principles

Use this file to keep repository docs compact, single-role, interface-first, positive, directive, structurally clean, and stable.

## 1. Single-Role Documents

- Use one document for one role.
- State what the document does; keep authority inside that role.
- Keep GPT-facing docs as task interfaces, Codex skills as execution contracts, manifests as implementation facts, and current/history JSON as evidence.

## 2. Interface-First Wording

- Prefer directive fields, labels, tables, and checklists.
- Use positive verbs: `Use`, `Include`, `Preserve`, `Validate`, `Record`, `Report`.
- Keep task interfaces field-based: inputs, outputs, writes, validation, final report.
- Prefer output allowlists and required operations before prohibitions.
- Write stable present-tense material; preserve native task names, fields, path labels, and interface labels.
- Keep construction notes, change logs, compatibility history, broad role-negation lists, and before/after wording out of reusable docs.
- Keep examples brief.

## 3. Structural Cleanup

- When a task asks to slim, clean, rewrite, refactor, or apply these principles, prefer structural cleanup over surface wording edits.
- Replace explanatory sections with execution contracts, task interfaces, tables, labels, and checklists.
- Remove redundant role introductions, duplicate prompt skeletons, repeated negative warnings, and human-facing explanations that do not guide execution.
- Preserve semantics first; reduce prose second.
- Keep required schema keys, status enums, artifact blockers, evidence roles, paths, command metadata labels, and validation contracts intact.

## 4. Cross-Document References

- Reference another document only for the current function.
- Use exact paths, skill names, and invocation blocks.
- Do not restate another document's full procedure.
- Do not compensate for missing routing or missing skill content inside an unrelated document.
- If a concrete skill owns the detailed procedure, reference or invoke it instead of duplicating it.

## 5. Execution Documents

- GPT-facing prompt documents expose selected-task interfaces, required fields, skill invocation, outputs, validation, and final-report expectations.
- Codex-facing skill documents expose execution contracts: inputs, extraction targets, output schema, validation, and blockers.
- Execution documents instruct what to inspect, extract, record, write, validate, and report.
- Keep execution documents out of explanatory essay form.

## 6. Necessary Blockers

- State allowed outputs and required operations first.
- Use hard blockers only for write scope, source integrity, artifact copying, privacy, validation failure, and decision-scope violations.
- Consolidate repeated blockers into category-level blockers where possible.
- Keep blockers short, specific, and task-relevant.
- Keep decision authority inside the role that owns the decision.

## 7. Artifact And Path Discipline

- `Automatic_2` must not store checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts.
- Store tracked paths as repository-relative paths or stable labels where possible.
- Keep private local absolute paths out of reusable docs and tracked history unless local execution requires them.
- Record artifact existence, metadata, and stable labels instead of copying source artifacts.

## 8. Reserved Interfaces

- Reserved interfaces are acceptable as stable intended contracts.
- A reserved interface is not callable until an accepted local `SKILL.md` exists and a concrete invocation is recorded.
- Use the same field shape as callable interfaces where practical.
- Keep reserved interfaces free of construction notes, compatibility history, and placeholder procedures.

## 9. Edit Scope

- Bound edits to the requested file set.
- Preserve unrelated files and user changes.
- Keep edits role-preserving.
- Update README or indexes only when explicitly requested.
