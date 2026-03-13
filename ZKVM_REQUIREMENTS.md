# zkVM Requirements

## Purpose
ZK-1. The zkVM shall take public input and private input into the circuit and generate a verification result that is on-chain verifiable and satisfies the on-chain requirements.

## Terminology
ZK-T-1. Criteria shall be a formal object with fields including `providerCriteriaId`, `evaluationLogic`, and `encodingRules`.

## External Dependencies
ZK-2. The zkVM shall follow the input data schema defined in https://github.com/primus-labs/DVC-Brevis/tree/brevis.
ZK-3. The zkVM shall verify input data using https://github.com/primus-labs/zktls-att-verification.
ZK-4. The zkVM circuit shall be designed and implemented using Pico (https://github.com/brevis-network/pico).

## Threat Model and Trust Boundaries
ZK-TM-1. The zkVM shall document assumed adversaries including malicious provers, malformed attestations, and colluding parties.
ZK-TM-2. The zkVM shall define trust boundaries for input acquisition, attestation verification, proof generation, and on-chain verification.

## Functional Requirements
ZK-5. The zkVM shall support multiple criteria.
ZK-6. The zkVM shall generate verification results based on the specified criteria (for example, KYC status or GitHub account age).
ZK-6a. The zkVM shall support multi-criteria proofs that combine multiple criteria in a single proof (for example, KYC status + GitHub account age).
ZK-7. The zkVM shall define explicit encoding and decoding for the on-chain verification result format.
ZK-8. The zkVM shall not expose security risks.
ZK-9. The zkVM shall validate input structure and schema version to prevent malformed or out-of-spec inputs.
ZK-10. The zkVM shall ensure output encoding is unambiguous and matches the on-chain decoder.
ZK-11. The zkVM shall bind the evaluated criteria to `providerCriteriaId` to prevent criteria mis-binding.
ZK-12. The zkVM shall include freshness binding (timestamp/nonce/proofHash) consistent with on-chain replay protection.
ZK-13. The zkVM shall use domain separation for all hashes to avoid cross-context collisions.
ZK-14. The zkVM shall verify zkTLS attestations completely and reject unverifiable or replayed attestations.
ZK-15. The zkVM shall minimize outputs to avoid leaking private or sensitive intermediate data.
ZK-16. The zkVM shall enforce all arithmetic and boolean constraints required for correctness (no missing constraints).
ZK-17. The zkVM shall enforce schema and version compatibility with on-chain verification.
ZK-18. The zkVM shall cryptographically bind any off-chain preprocessing inputs that affect the proof statement.
ZK-19. The zkVM shall include a complete performance evaluation of proving costs to inform future design decisions.
ZK-20. The zkVM shall define deterministic error signaling for malformed inputs and verification failures suitable for on-chain handling.
ZK-20a. Error signaling shall use enumerated error codes (for example, `ERR_SCHEMA_MISMATCH`, `ERR_ATTESTATION_INVALID`) rather than generic rejection.
ZK-20b. The zkVM shall fail fast during proof generation; if any error is detected, proof generation shall abort and no proof shall be produced.
ZK-21. The zkVM shall ensure proof generation is reproducible across environments for identical inputs and versions.
ZK-22. The zkVM shall provide audit artifacts sufficient to review proof generation and circuit configuration.
ZK-22a. Audit artifacts shall include, at minimum: circuit constraints, hash domain separation documentation, attestation verification logs, and proof-generation configuration (versions, parameters, and flags).
ZK-23. The zkVM shall define upgrade paths for new schema or criteria versions without breaking verification of existing proofs.
ZK-24. The zkVM shall define governance for schema and criteria changes, including who approves changes and how criteria definitions are versioned.

## Acceptance Criteria
ZK-AC-1. The zkVM produces a proof whose output is verifiable on-chain and meets the on-chain requirements.
ZK-AC-2. The zkVM accepts inputs that conform to the DVC-Brevis schema and rejects inputs that do not.
ZK-AC-3. The zkVM validates input data using zktls-att-verification.
ZK-AC-4. The zkVM supports multiple criteria and correctly binds results to the corresponding criteria.
ZK-AC-5. The zkVM outputs results that reflect the requested criteria (for example, KYC status or GitHub account age).
ZK-AC-5a. The zkVM supports multi-criteria proofs and outputs results for each requested criterion in a single proof.
ZK-AC-6. The on-chain verification result format has documented encoding and decoding rules.
ZK-AC-7. The zkVM design and implementation do not introduce security risks in input handling, proof generation, or output encoding.
ZK-AC-8. The zkVM rejects inputs that do not match the required schema or version.
ZK-AC-9. On-chain decoding of zkVM outputs is deterministic and matches the circuit encoding without ambiguity.
ZK-AC-10. The proof output is bound to the correct `providerCriteriaId` and criteria evaluation.
ZK-AC-11. Freshness primitives in the proof output are enforced and align with on-chain replay protection.
ZK-AC-12. Hashes in the circuit use explicit domain separation.
ZK-AC-13. zkTLS attestations are fully verified; unverifiable or replayed attestations are rejected.
ZK-AC-14. Outputs do not reveal private inputs or sensitive intermediate values beyond required fields.
ZK-AC-15. Constraint completeness tests or audits show no missing arithmetic/boolean constraints for correctness.
ZK-AC-16. Schema and version compatibility is enforced between zkVM output and on-chain verification.
ZK-AC-17. Any off-chain preprocessing inputs that influence the proof are bound in-circuit.
ZK-AC-18. A performance evaluation report exists covering proving time, memory usage, and cost drivers for the supported criteria.
ZK-AC-19. Error signaling is deterministic and maps to defined on-chain failure modes.
ZK-AC-19a. Enumerated error codes are documented and used for all rejection paths.
ZK-AC-19b. Proof generation halts on first error and does not emit a proof for invalid inputs.
ZK-AC-20. Proofs generated on different environments with identical inputs and versions are byte-for-byte reproducible or produce equivalent verifiable outputs.
ZK-AC-21. Audit artifacts are available to review circuit configuration, inputs, and proof generation parameters.
ZK-AC-21a. Audit artifacts include circuit constraints, hash domain separation documentation, attestation verification logs, and proof-generation configuration.
ZK-AC-22. Upgrade procedures are documented and preserve verification of existing proofs.
ZK-AC-23. The zkVM circuit is implemented in Pico.
ZK-AC-24. Governance processes for schema and criteria changes are documented, with versioning and approval roles clearly defined.

## Performance Evaluation Matrix
ZK-PM-1. Metrics shall include proving time (p50/p95), peak memory, proof size, verification cost (gas or CPU), constraint/gate count, and witness generation time.
ZK-PM-2. Measurements shall be reported per criteria (for example, KYC status, GitHub account age).
ZK-PM-3. Measurements shall cover input sizes (small, typical, worst-case) including attestation size when applicable.
ZK-PM-4. Measurements shall be captured across hardware tiers (developer laptop, standard server, high-end server) or explicitly note the tested hardware.
ZK-PM-5. The report shall identify primary cost drivers and the dominant stage in the pipeline.
ZK-PM-6. The report shall compare results against at least one baseline (prior version or alternative zkVM) and explain deltas.
