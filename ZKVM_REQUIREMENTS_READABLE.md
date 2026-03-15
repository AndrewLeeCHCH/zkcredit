# zkVM Requirements (Human-Readable)

## Purpose
ZK-1. The zkVM circuit takes public and private inputs, then outputs a verification result that can be verified on-chain and satisfies on-chain requirements.

## Terminology
ZK-T-1. A `Criteria` object includes at least: `providerCriteriaId`, `evaluationLogic`, and `encodingRules`.

## External Dependencies
ZK-2. Input data must follow the DVC-Brevis schema: https://github.com/primus-labs/DVC-Brevis/tree/brevis.
ZK-3. Input data must be verified with zktls-att-verification: https://github.com/primus-labs/zktls-att-verification.
ZK-4. The circuit must be implemented with Pico: https://github.com/brevis-network/pico.

## Threat Model and Trust Boundaries
ZK-TM-1. The design must document adversaries, including malicious provers, malformed attestations, and colluding parties.
ZK-TM-2. The design must define trust boundaries for input collection, attestation verification, proof generation, and on-chain verification.

## Functional Requirements
ZK-5. The zkVM supports multiple criteria.
ZK-6. The zkVM generates results from the requested criteria (for example, KYC status or GitHub account age).
ZK-6a. The zkVM supports multi-criteria proofs in one proof (for example, KYC status + GitHub account age).
ZK-7. The on-chain result format has explicit encode/decode definitions.
ZK-9. The zkVM validates input structure and schema version, and rejects malformed or out-of-spec inputs.
ZK-10. Output encoding is unambiguous and matches the on-chain decoder.
ZK-11. Criteria evaluation is cryptographically bound to `providerCriteriaId`.
ZK-12. Freshness fields (timestamp/nonce/proofHash) are bound in a way that matches on-chain replay rules.
ZK-13. All hashes use domain separation to prevent cross-context collisions.
ZK-14. zkTLS attestation verification is complete; unverifiable or replayed attestations are rejected.
ZK-15. Outputs are minimized so private or sensitive intermediate data is not leaked.
ZK-16. All required arithmetic and boolean constraints are enforced (no missing constraints).
ZK-17. Schema and version compatibility with on-chain verification is enforced.
ZK-18. Any off-chain preprocessing inputs that affect the proof statement are cryptographically bound in-circuit.
ZK-19. A complete performance evaluation is required because proving cost affects design decisions.
ZK-20. Error signaling is deterministic for malformed inputs and verification failures, and suitable for on-chain handling.
ZK-20a. Errors use enumerated codes (for example, `ERR_SCHEMA_MISMATCH`, `ERR_ATTESTATION_INVALID`), not generic rejection.
ZK-20b. The system fails fast during proving: once an error is detected, proving aborts and no proof is produced.
ZK-21. Proof generation is reproducible across environments when inputs and versions are identical.
ZK-22. Audit artifacts are provided to review proof generation and circuit configuration.
ZK-22a. Minimum audit artifacts include: circuit constraints, hash-domain-separation documentation, attestation-verification logs, and proof-generation configuration (versions, parameters, flags).
ZK-23. Upgrade paths are defined for new schema/criteria versions without breaking verification of existing proofs.
ZK-24. Governance for schema/criteria changes is defined, including approvers and criteria versioning.

## Acceptance Criteria
ZK-AC-1. The zkVM output is verifiable on-chain and meets on-chain requirements.
ZK-AC-2. Inputs that match DVC-Brevis schema are accepted; non-conforming inputs are rejected.
ZK-AC-3. Input validation uses zktls-att-verification.
ZK-AC-4. Multiple criteria are supported and correctly bound to their criteria identifiers.
ZK-AC-5. Output matches requested criteria (for example, KYC status or GitHub account age).
ZK-AC-5a. Multi-criteria proofs output all requested criteria in one proof.
ZK-AC-6. Encoding/decoding rules for on-chain verification results are documented.
ZK-AC-8. Inputs with wrong schema/version are rejected.
ZK-AC-9. On-chain decoding is deterministic and unambiguous with circuit encoding.
ZK-AC-10. Proof output is bound to correct `providerCriteriaId` and evaluation logic.
ZK-AC-11. Freshness values are enforced and consistent with on-chain replay protection.
ZK-AC-12. Hash domain separation is explicit in-circuit.
ZK-AC-13. Unverifiable or replayed zkTLS attestations are rejected.
ZK-AC-14. Outputs reveal only required fields, not private inputs or sensitive intermediates.
ZK-AC-15. Tests/audits demonstrate no missing arithmetic/boolean constraints needed for correctness.
ZK-AC-16. Schema/version compatibility between zkVM output and on-chain verification is enforced.
ZK-AC-17. Off-chain preprocessing inputs that affect proofs are bound in-circuit.
ZK-AC-18. A performance report exists and covers proving time, memory usage, and key cost drivers by supported criteria.
ZK-AC-19. Error signaling is deterministic and maps to on-chain failure modes.
ZK-AC-19a. Enumerated error codes are documented and used across all rejection paths.
ZK-AC-19b. Proving halts on first error and produces no proof for invalid inputs.
ZK-AC-20. With identical inputs and versions, outputs are byte-identical or verifiably equivalent across environments.
ZK-AC-21. Audit artifacts are available for reviewing circuit config, inputs, and proving parameters.
ZK-AC-21a. Audit artifacts include constraints, domain-separation docs, attestation-verification logs, and proving configuration.
ZK-AC-22. Upgrade procedures are documented and preserve verification of existing proofs.
ZK-AC-23. Circuit implementation uses Pico.
ZK-AC-24. Governance for schema/criteria changes is documented, with clear roles and versioning.

## Performance Evaluation Matrix
ZK-PM-1. Metrics include proving time (p50/p95), peak memory, proof size, verification cost (gas or CPU), constraint/gate count, and witness generation time.
ZK-PM-2. Measurements are reported per criterion (for example, KYC status and GitHub account age).
ZK-PM-3. Measurements cover small, typical, and worst-case input sizes, including attestation size where relevant.
ZK-PM-4. Measurements cover hardware tiers (developer laptop, standard server, high-end server), or clearly state tested hardware.
ZK-PM-5. The report identifies dominant pipeline stages and primary cost drivers.
ZK-PM-6. The report compares against at least one baseline (previous version or alternative zkVM) and explains deltas.
