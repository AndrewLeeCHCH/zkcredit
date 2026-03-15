# Agent Instructions

## Requirements Document Sync
- When any requirements change, update both the canonical requirements doc and its human-readable counterpart in the same change.
- Canonical ↔ human-readable pairs:
- `ONCHAIN_REQUIREMENTS.md` ↔ `ONCHAIN_REQUIREMENTS_READABLE.md`
- `ZKVM_REQUIREMENTS.md` ↔ `ZKVM_REQUIREMENTS_READABLE.md` (create the readable file if it does not exist when requirements change)
- Human-readable documents must preserve the same section order and hierarchy depth as their canonical requirements documents.
- Human-readable documents may simplify wording, but must not flatten, reorder, or omit requirement levels (for example, `FR-5a`, `FR-5b` depth and ordering).

## Spec Sync
- When requirements change, update corresponding spec documentation in the same change.
- Requirement ↔ spec mapping:
- `ONCHAIN_REQUIREMENTS.md` ↔ `SPEC.md`
- `ZKVM_REQUIREMENTS.md` ↔ `ZKVM_SPEC.md`
