# Web2–Web3 Identity Verification System Requirements

## Purpose
R-1. The system shall provide trust-minimized verification of Web2 attributes.
R-2. The system shall preserve user privacy when binding Web2 identity to Web3 addresses.
R-3. The system shall securely store verification results.
R-4. The system shall prevent impersonation and replay attacks.
R-5. The system shall correctly map proofs to providers and criteria sets.

## Binding Requirements
BR-1. A Web3 user shall be able to bind to different Web2 identities per `providerId`.
BR-2. The system shall support unbind operations.
BR-3. Binding shall be represented by storing `identityNullifier` and `user`.
BR-4. A Web2 identity (via `identityNullifier`) shall bind to only one Web3 user.
BR-5. Binding shall be enforced per provider (not per criteria).
BR-6. After unbind, all historical data for that nullifier shall be wiped.
BR-7. Unbind shall be blocked if a cooldown window since the last bind or last result submission has not elapsed.
BR-8. Unbind shall be blocked if any downstream system has locked the result.

## Security Requirements
SR-1. Proofs shall be user-bound by including `proofUser` in the zkVM output.
SR-2. Verification results shall be stored under `proofUser` regardless of transaction sender.
SR-3. The system shall reject submissions where `identityNullifier` is already bound to another user.
SR-4. The system shall validate `providerCriteriaId` (on-chain or off-chain registry).
SR-5. Proofs shall include replay-prevention fields: `timestamp` or `nonce` or `proofHash` (or a combination).
SR-6. The contract shall reject reused timestamps, reused nonces, and reused proof hashes.
SR-7. The contract shall reject proofs older than the latest stored result for the same user and criteria.
SR-8. The system shall allow relayer-submitted transactions while still enforcing that results are stored under `proofUser`.

## Verification Result Requirements
VR-1. Each stored result shall include: `providerCriteriaId`, `identityNullifier`, `providerOutput`, and any extra required fields.
VR-2. Users may have multiple results per `providerCriteriaId`.
VR-3. Only the latest result per `(user, providerCriteriaId)` shall be active.

## Functional Requirements
FR-1. The system shall implement `bindIdentity(user, identityNullifier)`.
FR-2. `bindIdentity` shall fail if the `identityNullifier` is already bound to a different user.
FR-3. The system shall implement `unbindIdentity(user, identityNullifier)`.
FR-4. `unbindIdentity` shall fail if the cooldown window since last bind or last result submission has not elapsed.
FR-5. `unbindIdentity` shall fail if any downstream system has locked the result.
FR-6. `unbindIdentity` shall be callable only by `proofUser`.
FR-7. `unbindIdentity` shall wipe all historical data for the nullifier upon success.
FR-9. The system shall implement `submitVerificationResult(proof)`.
FR-10. `submitVerificationResult` shall verify the zkVM proof.
FR-11. `submitVerificationResult` shall validate `providerCriteriaId` if it is maintained on-chain.
FR-12. `submitVerificationResult` shall enforce impersonation resistance and replay protection.
FR-13. `submitVerificationResult` shall store results under `proofUser`.
FR-14. `submitVerificationResult` shall update the latest-result pointer per `(user, providerCriteriaId)`.
FR-15. `submitVerificationResult` shall fail if:
FR-15a. the `identityNullifier` is bound to another user, or
FR-15b. the `providerCriteriaId` is unregistered (if on-chain), or
FR-15c. the proof is stale or replayed, or
FR-15d. the proof is invalid.

## Query Requirements
QR-1. The system shall implement `getLatestResult(user, providerCriteriaId)`.
QR-2. `getLatestResult` shall return the latest result for the user and criteria.
QR-3. The system may implement `getAllResults(user, providerCriteriaId)`.
QR-4. If `getAllResults` is not implemented, event-based indexing shall be supported.

## Constraints and Invariants
CI-1. One Web2 identity shall map to only one Web3 user.
CI-2. No raw Web2 IDs shall be stored on-chain.
CI-3. Only `identityNullifier` shall be stored on-chain.
CI-4. `identityNullifier` shall be deterministic.
CI-5. `providerCriteriaId` shall be deterministic.
CI-6. zkVM outputs shall be deterministic.

## Non-Functional Requirements
NFR-1. The system shall be gas-efficient.
NFR-2. The system shall be upgradeable.
NFR-3. The system shall use a modular design.

## Acceptance Criteria
AC-1. Identity Binding (Provider-Level)
AC-1.1. Binding correctness: When a user calls `bindIdentity(user, identityNullifier)`, the contract must store the binding only if:
AC-1.1a. the `identityNullifier` is not already bound to another user, and
AC-1.1b. the user is not already bound to a different nullifier.
AC-1.2. Unbinding correctness: After `unbindIdentity(user, identityNullifier)`, the user must:
AC-1.2a. no longer have an active binding for that nullifier, and
AC-1.2b. have all historical verification results for that nullifier wiped out.
AC-1.3. Nullifier determinism: For the same `(providerId, web2UserId)`, the zkVM must always output the same `identityNullifier`.
AC-2. Verification Result Submission
AC-2.1. Proof-user attribution: When a proof is submitted, the verification result must be stored under `proofUser` (the user inside the proof), not `msg.sender`.
AC-2.2. Binding validation: The contract must reject proofs where `identityNullifier` is bound to a different user.
AC-2.3. Provider–criteria consistency: The contract must reject proofs where `providerCriteriaId` is not valid.
AC-2.4. Multiple results: The system must allow multiple verification results for the same `(user, providerCriteriaId)` pair.
AC-2.5. Latest pointer: The system must correctly update the latest-result pointer.
AC-3. zkVM Output Validation
AC-3.1. Required fields: The contract must reject proofs missing any of:
AC-3.1a. `proofUser`
AC-3.1b. `providerCriteriaId`
AC-3.1c. `identityNullifier`
AC-3.1d. `providerOutput`
AC-3.1e. `timestamp` and/or `nonce` and/or `proofHash`
AC-3.2. Nullifier correctness: The zkVM must prove `identityNullifier = hash(providerId, web2UserId)`.
AC-3.3. Criteria correctness: The zkVM must prove that the evaluation result corresponds to the ruleset represented by `providerCriteriaId`.
AC-4. Anti-Impersonation Guarantees
AC-4.1. No proof theft: If Alice submits a proof generated for Bob, the result must be stored under Bob, not Alice.
AC-4.2. No identity hijacking: Alice must not be able to bind Bob’s `identityNullifier` to herself.
AC-4.3. No cross-provider impersonation: A nullifier from provider A must not be accepted for provider B.
AC-5. Replay Attack Resistance
AC-5.1. Freshness enforcement: The contract must reject proofs if:
AC-5.1a. the timestamp is older than the latest stored result, or
AC-5.1b. the nonce has been used before, or
AC-5.1c. the `proofHash` has been used before.
AC-5.2. Rebinding invalidation: After a user unbinds and rebinds to a new `identityNullifier`, all proofs tied to the old nullifier must be rejected.
AC-6. Querying
AC-6.1. Latest result: `getLatestResult(user, providerCriteriaId)` must return the most recent result, or empty if none exists.
AC-6.2. Full history: `getAllResults(user, providerCriteriaId)` must return all stored results in chronological order.
AC-7. Privacy Guarantees
AC-7.1. No raw Web2 IDs: The contract must not store or emit raw Web2 user IDs.
AC-7.2. Nullifier-only identity: All identity references must use `identityNullifier`.
AC-8. Determinism and Consistency
AC-8.1. Deterministic nullifier: Same `(providerId, web2UserId)` must always produce the same `identityNullifier`.
AC-8.2. Deterministic providerCriteriaId: Same ruleset must always produce the same `providerCriteriaId`.
AC-8.3. Deterministic zkVM output: zkVM must produce identical outputs for identical inputs.
