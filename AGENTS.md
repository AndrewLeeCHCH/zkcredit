# Agent Instructions

## Requirements Document Sync
- When any requirements change, update both the canonical requirements doc and its human-readable counterpart in the same change.
- Canonical ↔ human-readable pairs:
- `ONCHAIN_REQUIREMENTS.md` ↔ `ONCHAIN_REQUIREMENTS_READABLE.md`
- `ZKVM_REQUIREMENTS.md` ↔ `ZKVM_REQUIREMENTS_READABLE.md` (create the readable file if it does not exist when requirements change)
- Human-readable documents must preserve the same section order and hierarchy depth as their canonical requirements documents.
- Human-readable documents must preserve canonical requirement wording exactly (no paraphrasing), while removing identifier labels only.
- Human-readable documents must not flatten or reorder requirement levels.
- Human-readable formatting rule:
- Use basic markdown sections + bullets (similar to `SPEC.md` style).
- Top-level requirements use bullet lists.
- Child requirement levels use nested bullet lists under their parent requirement.
- Do not include requirement IDs in human-readable documents (remove labels such as `R-1`, `BR-1a`, `FR-12d`, `ZK-20b`).
- Do not present Acceptance Criteria (`AC-*`, `ZK-AC-*`) in human-readable documents; keep them only in canonical requirement docs.

## Spec Sync
- When requirements change, update corresponding spec documentation in the same change.
- Requirement ↔ spec mapping:
- `ONCHAIN_REQUIREMENTS.md` ↔ `SPEC.md`
- `ZKVM_REQUIREMENTS.md` ↔ `ZKVM_SPEC.md`
