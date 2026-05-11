# Document Authoring Principles

Use this file to keep repository docs compact, single-role, interface-first, positive, directive, and stable.

## 1. Single-Role Documents

- Use one document for one role.
- State what the document does; keep authority inside that role.
- Keep GPT-facing docs as interfaces or workflows, Codex skills as bounded procedures, manifests as implementation facts, and current/history JSON as evidence.

## 2. Interface-First Wording

- Prefer directive fields, labels, tables, and checklists.
- Use positive verbs: `Use`, `Include`, `Preserve`, `Validate`, `Record`, `Report`.
- Keep task interfaces field-based: inputs, outputs, writes, validation, final report.
- Keep examples brief.

## 3. Cross-Document References

- Reference another document only for the current function.
- Use exact paths, skill names, and invocation blocks.
- Copy concrete skill invocation from the selected task interface or skill contract.
- Keep references minimal; do not repeat another document's procedure or compensate for missing content elsewhere.

## 4. Stable Native Wording

- Write stable present-tense repository material.
- Preserve native task names, fields, path labels, and interface labels.
- Avoid construction notes, change logs, compatibility history, historical explanations, and before/after wording.
- Avoid broad role-negation lists.

## 5. Necessary Blockers Only

- Retain hard blockers for artifacts, privacy, validation, and write scope.
- Use `Do not` wording for clear safety or artifact-control rules.
- Keep blockers short, specific, and task-relevant.
- Do not add tuning recommendations, next hyperparameters, accepted-baseline decisions, stop/continue decisions, branch decisions, or method/paper conclusions unless the role owns that decision.

## 6. Artifact And Path Discipline

- `Automatic_2` must not store checkpoints, model weights, full logs, full CSVs, raw outputs, plots, trajectories, or binary artifacts.
- Store tracked paths as repository-relative paths or stable labels where possible.
- Keep private local absolute paths out of reusable docs and tracked history unless local execution requires them.
- Record artifact existence, metadata, and stable labels instead of copying source artifacts.

## 7. Reserved Interfaces

- Reserved interfaces are acceptable as stable intended contracts.
- A reserved interface is not callable until an accepted local `SKILL.md` exists and a concrete invocation is recorded.
- Use the same field shape as callable interfaces where practical.
- Keep reserved interfaces free of construction notes, compatibility history, and placeholder procedures.

## 8. Edit Scope

- Bound edits to the requested file set.
- Preserve unrelated files and user changes.
- Keep edits minimal and role-preserving.
- Update README or indexes only when explicitly requested.
