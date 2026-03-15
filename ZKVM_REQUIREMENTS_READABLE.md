# zkVM Requirements (Human-Readable)

## Purpose
The zkVM takes public and private inputs, evaluates criteria, and produces a verification result that is verifiable on-chain and satisfies the on-chain requirements.

## Terminology
- Criteria is a formal object with fields including `providerCriteriaId`, `evaluationLogic`, and `encodingRules`.

## External Dependencies
- Follow the input data schema defined in https://github.com/primus-labs/DVC-Brevis/tree/brevis.
- Verify input data using https://github.com/primus-labs/zktls-att-verification.
- Design and implement the zkVM circuit using Pico (https://github.com/brevis-network/pico).

## Threat Model and Trust Boundaries
- Document assumed adversaries including malicious provers, malformed attestations, and colluding parties.
- Define trust boundaries for input acquisition, attestation verification, proof generation, and on-chain verification.

## Functional Requirements
- Support multiple criteria.
- Generate verification results based on criteria (for example, KYC status or GitHub account age).
- Support multi-criteria proofs that combine multiple criteria in a single proof (for example, KYC status + GitHub account age).
- Define explicit encoding and decoding for the on-chain verification result format.
- Validate input structure and schema version to prevent malformed or out-of-spec inputs.
- Ensure output encoding is unambiguous and matches the on-chain decoder.
- Bind the evaluated criteria to `providerCriteriaId` to prevent criteria mis-binding.
- Include freshness binding (timestamp/nonce/proofHash) consistent with on-chain replay protection.
- Use domain separation for all hashes to avoid cross-context collisions.
- Fully verify zkTLS attestations and reject unverifiable or replayed attestations.
- Minimize outputs to avoid leaking private or sensitive intermediate data.
- Enforce all arithmetic and boolean constraints required for correctness (no missing constraints).
- Enforce schema and version compatibility with on-chain verification.
- Cryptographically bind any off-chain preprocessing inputs that affect the proof statement.
- Include a complete performance evaluation of proving costs to inform future design decisions.
- Define deterministic error signaling for malformed inputs and verification failures suitable for on-chain handling.
- Use enumerated error codes (for example, `ERR_SCHEMA_MISMATCH`, `ERR_ATTESTATION_INVALID`) rather than generic rejection.
- Fail fast during proof generation; if any error is detected, proof generation aborts and no proof is produced.
- Ensure proof generation is reproducible across environments for identical inputs and versions.
- Provide audit artifacts sufficient to review proof generation and circuit configuration.
- Audit artifacts include, at minimum: circuit constraints, hash domain separation documentation, attestation verification logs, and proof-generation configuration (versions, parameters, and flags).
- Define upgrade paths for new schema or criteria versions without breaking verification of existing proofs.
- Define governance for schema and criteria changes, including who approves changes and how criteria definitions are versioned.

## Acceptance Criteria
- The zkVM produces a proof whose output is verifiable on-chain and meets the on-chain requirements.
- Inputs conforming to the DVC-Brevis schema are accepted; non-conforming inputs are rejected.
- Input data is validated using zktls-att-verification.
- Multiple criteria are supported and results are correctly tied to the requested criteria.
- Output results reflect the requested criteria (for example, KYC status or GitHub account age).
- Multi-criteria proofs are supported and output results for each requested criterion in a single proof.
- The on-chain verification result format has documented encoding and decoding rules.
- Inputs that do not match the required schema or version are rejected.
- On-chain decoding of zkVM outputs is deterministic and matches the circuit encoding without ambiguity.
- The proof output is bound to the correct `providerCriteriaId` and criteria evaluation.
- Freshness primitives in the proof output are enforced and align with on-chain replay protection.
- Hashes in the circuit use explicit domain separation.
- zkTLS attestations are fully verified; unverifiable or replayed attestations are rejected.
- Outputs do not reveal private inputs or sensitive intermediate values beyond required fields.
- Constraint completeness tests or audits show no missing arithmetic/boolean constraints for correctness.
- Schema and version compatibility is enforced between zkVM output and on-chain verification.
- Any off-chain preprocessing inputs that influence the proof are bound in-circuit.
- A performance evaluation report exists covering proving time, memory usage, and cost drivers for the supported criteria.
- Error signaling is deterministic and maps to defined on-chain failure modes.
- Enumerated error codes are documented and used for all rejection paths.
- Proof generation halts on first error and does not emit a proof for invalid inputs.
- Proofs generated on different environments with identical inputs and versions are byte-for-byte reproducible or produce equivalent verifiable outputs.
- Audit artifacts are available to review circuit configuration, inputs, and proof generation parameters.
- Audit artifacts include circuit constraints, hash domain separation documentation, attestation verification logs, and proof-generation configuration.
- Upgrade procedures are documented and preserve verification of existing proofs.
- Governance processes for schema and criteria changes are documented, with versioning and approval roles clearly defined.
- The zkVM circuit is implemented in Pico.

## Performance Evaluation Matrix
- Metrics include proving time (p50/p95), peak memory, proof size, verification cost (gas or CPU), constraint/gate count, and witness generation time.
- Measurements are reported per criteria (for example, KYC status, GitHub account age).
- Measurements cover input sizes (small, typical, worst-case) including attestation size when applicable.
- Measurements are captured across hardware tiers (developer laptop, standard server, high-end server) or explicitly note the tested hardware.
- The report identifies primary cost drivers and the dominant stage in the pipeline.
- The report compares results against at least one baseline (prior version or alternative zkVM) and explains deltas.
