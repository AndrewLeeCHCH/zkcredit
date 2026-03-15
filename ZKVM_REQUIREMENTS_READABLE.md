# zkVM Requirements (Human-Readable)

## Purpose
- The zkVM shall take public input and private input into the circuit and generate a verification result that is on-chain verifiable and satisfies the on-chain requirements.
## Terminology
- Criteria shall be a formal object with fields including `providerCriteriaId`, `evaluationLogic`, and `encodingRules`.
## External Dependencies
- The zkVM shall follow the input data schema defined in https://github.com/primus-labs/DVC-Brevis/tree/brevis.
- The zkVM shall verify input data using https://github.com/primus-labs/zktls-att-verification.
- The zkVM circuit shall be designed and implemented using Pico (https://github.com/brevis-network/pico).
## Threat Model and Trust Boundaries
- The zkVM shall document assumed adversaries including malicious provers, malformed attestations, and colluding parties.
- The zkVM shall define trust boundaries for input acquisition, attestation verification, proof generation, and on-chain verification.
## Functional Requirements
- The zkVM shall support multiple criteria.
- The zkVM shall generate verification results based on the specified criteria (for example, KYC status or GitHub account age).
  - The zkVM shall support multi-criteria proofs that combine multiple criteria in a single proof (for example, KYC status + GitHub account age).
- The zkVM shall define explicit encoding and decoding for the on-chain verification result format.
- The zkVM shall validate input structure and schema version to prevent malformed or out-of-spec inputs.
- The zkVM shall ensure output encoding is unambiguous and matches the on-chain decoder.
- The zkVM shall bind the evaluated criteria to `providerCriteriaId` to prevent criteria mis-binding.
- The zkVM shall include freshness binding (timestamp/nonce/proofHash) consistent with on-chain replay protection.
- The zkVM shall use domain separation for all hashes to avoid cross-context collisions.
- The zkVM shall verify zkTLS attestations completely and reject unverifiable or replayed attestations.
- The zkVM shall minimize outputs to avoid leaking private or sensitive intermediate data.
- The zkVM shall enforce all arithmetic and boolean constraints required for correctness (no missing constraints).
- The zkVM shall enforce schema and version compatibility with on-chain verification.
- The zkVM shall cryptographically bind any off-chain preprocessing inputs that affect the proof statement.
- The zkVM shall include a complete performance evaluation of proving costs to inform future design decisions.
- The zkVM shall define deterministic error signaling for malformed inputs and verification failures suitable for on-chain handling.
  - Error signaling shall use enumerated error codes (for example, `ERR_SCHEMA_MISMATCH`, `ERR_ATTESTATION_INVALID`) rather than generic rejection.
  - The zkVM shall fail fast during proof generation; if any error is detected, proof generation shall abort and no proof shall be produced.
- The zkVM shall ensure proof generation is reproducible across environments for identical inputs and versions.
- The zkVM shall provide audit artifacts sufficient to review proof generation and circuit configuration.
  - Audit artifacts shall include, at minimum: circuit constraints, hash domain separation documentation, attestation verification logs, and proof-generation configuration (versions, parameters, and flags).
- The zkVM shall define upgrade paths for new schema or criteria versions without breaking verification of existing proofs.
- The zkVM shall define governance for schema and criteria changes, including who approves changes and how criteria definitions are versioned.
## Performance Evaluation Matrix
- Metrics shall include proving time (p50/p95), peak memory, proof size, verification cost (gas or CPU), constraint/gate count, and witness generation time.
- Measurements shall be reported per criteria (for example, KYC status, GitHub account age).
- Measurements shall cover input sizes (small, typical, worst-case) including attestation size when applicable.
- Measurements shall be captured across hardware tiers (developer laptop, standard server, high-end server) or explicitly note the tested hardware.
- The report shall identify primary cost drivers and the dominant stage in the pipeline.
- The report shall compare results against at least one baseline (prior version or alternative zkVM) and explain deltas.
