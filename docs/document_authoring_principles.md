# Document Authoring Principles

This document defines repository-wide authoring principles for creating and editing `Automatic_2` documentation and Codex skill files.

This document is not a user-request router, a GPT tuning workflow, a Codex execution skill, a training-system manifest, a factual analysis report, a history archive, or a decision policy.

## 1. Single-Responsibility Documents

Each document must perform its own role only.

- Engineering manifests describe implementation facts.
- GPT-to-Codex prompt documents describe prompt packaging.
- Codex skills describe bounded Codex execution procedures.
- Current analysis JSON files store the latest structured factual analysis.
- History files store accepted archive records.

This principle does not define workflow routing or tuning decisions.

## 2. No Cross-Document Compensation

One document must not compensate for, reinterpret, patch, or restate another document's full role.

A document may call or reference another document only when needed for its own direct function. Skill invocation should be minimal and format-correct. A prompt-packaging document may invoke a skill by path without restating the full skill procedure. A skill may require passive context from a manifest without becoming a manifest.

## 3. Native Stable Wording

Documents should be written as stable project materials, not as construction notes. Avoid patch-style or construction-phase wording unless explicitly requested by the user.

Avoid wording that frames content as transitional, including:

- prior-state adverbs such as "pre<!-- -->viously" or "now"
- reversal phrases such as "no lon<!-- -->ger"
- change-result phrases such as "after this cha<!-- -->nge" or "this fi<!-- -->xes"
- status labels such as "temporary" or "workaround"
- change-history labels such as "migration no<!-- -->te" or "patch no<!-- -->te"
- format-transition labels such as "old for<!-- -->mat" or "new for<!-- -->mat"

Reserved interfaces are allowed when they are intentional stable interface contracts, not temporary construction notes.

## 4. Scope Boundaries

Documents must not introduce extra authority beyond their role.

- Codex skills must not provide GPT tuning decisions unless explicitly assigned a decision role.
- Routine Codex skills must not invent next hyperparameters.
- Engineering manifests must not become tuning guides.
- Prompt-packaging documents must not become data-analysis methods.
- Archive documents and archive skills must not reanalyze training outputs.

## 5. Artifact And Privacy Discipline

`Automatic_2` must not store checkpoints, model weights, full logs, full CSVs, raw outputs, plots, or trajectories.

Tracked records should use repository-relative paths or stable repository labels when possible. Private local absolute paths should not be stored in tracked history or reusable documentation unless required for local execution examples.

## 6. Reserved Interfaces

Reserved interfaces may be documented when they are part of the intended native construction direction.

- A reserved interface is acceptable when it defines a stable intended contract.
- A reserved interface should not include change-history notes, historical explanations, or temporary compatibility language.
- A reserved interface is not a callable skill until it has a concrete local `SKILL.md` path and a canonical skill invocation.

## 7. Editing Discipline

Edits should be minimal, role-preserving, and bounded to the requested files.

New documents should not trigger `README.md` updates by default during active construction unless the task explicitly requests index maintenance.
