# Document Authoring Principles

Use this file to keep repository docs compact, single-role, interface-first, positive, directive, signal-preserving, and structurally clean.

## 1. Single-Role Documents

- Use one document for one role.
- State what the document does; keep authority inside that role.
- Keep GPT-facing docs as task interfaces, Codex skills as execution contracts, manifests as implementation facts, and current/history JSON as compact evidence.

## 2. Interface-First Wording

- Prefer directive fields, labels, tables, and checklists.
- Use positive verbs: `Use`, `Include`, `Preserve`, `Validate`, `Record`, `Report`.
- Keep task interfaces field-based: inputs, outputs, writes, validation, final report.
- Prefer output allowlists and required operations before prohibitions.
- Write stable present-tense material; preserve native task names, fields, path labels, and interface labels.
- Keep construction notes, change logs, compatibility history, broad role-negation lists, and before/after wording out of reusable docs.
- Keep examples brief.

## 3. Structural Cleanup

- When a task asks to slim, clean, rewrite, refactor, restructure, compact, or apply these principles, perform structural cleanup by default.
- Structural cleanup may reorganize sections, merge duplicate blocks, convert prose to tables/checklists, rename headings, and delete redundant material.
- Use minimal patching only when the task explicitly asks for a small fix, repair, or single-line change.
- Preserve role, schema, evidence roles, status enums, paths, command metadata labels, validation contracts, artifact blockers, and decision boundaries.

## 4. Evidence And Report Density

- Preserve decision-useful signal; remove process noise.
- Full evidence stays in source artifacts; tracked reports store compact digests.
- Retain fields that support execution, validation, evidence admission, tuning review, archive integrity, or safety boundaries.
- Summarize command traces, artifact inventories, headers, raw rows, broad numeric ranges, backend/seed detail, and repeated audit prose.
- Prefer status labels, counts, key metrics, deltas, compact summaries, and stable identifiers.
- Remove fields that do not help the target role act, validate, decide, or preserve evidence integrity.
- Retain a field or paragraph only if it supports the document role: execute, validate, admit evidence, review tuning, archive, preserve safety, or define a stable interface.

## 5. Cross-Document References

- Reference another document only for the current function.
- Use exact paths, skill names, and invocation blocks.
- Do not restate another document's full procedure.
- Do not compensate for missing routing or missing skill content inside an unrelated document.
- If a concrete skill owns the detailed procedure, reference or invoke it instead of duplicating it.

## 6. Execution Documents

- GPT-facing docs stay as task interfaces and skill-handshake prompts.
- Codex-facing skills stay as execution contracts: inputs, extraction targets, output schema, validation, and blockers.
- Current/history JSON stays as compact evidence, not source-artifact dumps.
- Execution documents instruct what to inspect, extract, record, write, validate, and report.
- Reports and JSON should be review-facing digests unless the task explicitly asks for audit detail.

## 7. Necessary Blockers

- State allowed outputs and required operations first.
- Use hard blockers only for write scope, source integrity, artifact copying, privacy, validation failure, and decision-scope violations.
- Consolidate repeated blockers into category-level blockers where possible.
- Keep blockers short, specific, and task-relevant.
- Keep decision authority inside the role that owns the decision.

## 8. Artifact And Path Discipline

- `Automatic_2` must not store checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts.
- Store tracked paths as repository-relative paths or stable labels where possible.
- Keep private local absolute paths out of reusable docs and tracked history unless local execution requires them.
- Record artifact existence, metadata, and stable labels instead of copying source artifacts.

## 9. Reserved Interfaces

- Reserved interfaces are acceptable as stable intended contracts.
- A reserved interface is not callable until an accepted local `SKILL.md` exists and a concrete invocation is recorded.
- Use the same field shape as callable interfaces where practical.
- Keep reserved interfaces free of construction notes, compatibility history, and placeholder procedures.

## 10. Edit Scope

- Bound edits to the requested file set.
- Preserve unrelated files and user changes.
- Keep edits role-preserving.
- Update README or indexes only when explicitly requested.
