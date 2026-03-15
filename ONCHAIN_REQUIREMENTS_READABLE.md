# Web2–Web3 Identity Verification System Requirements (Human-Readable)

## Purpose
R-1. Provide trust-minimized verification of Web2 attributes.
R-2. Preserve user privacy when binding Web2 identity to Web3 addresses.
R-3. Securely store verification results.
R-4. Prevent impersonation and replay attacks.
R-5. Correctly map proofs to providers and criteria sets.

## Binding Requirements
BR-1. A Web3 user can bind to different Web2 identities per `providerId`.
BR-1a. Binding multiplicity follows provider-specific policy.
BR-1b. For `Binance`, one Web3 user can have only one active bound nullifier at a time.
BR-1c. For providers other than `Binance` (including `GitHub`), one Web3 user can have multiple active bound nullifiers, subject to nullifier uniqueness constraints.
BR-2. Unbind operations are supported.
BR-3. Binding is represented by storing `identityNullifier` and `user`.
BR-4. A Web2 identity (via `identityNullifier`) can bind to only one Web3 user.
BR-5. Binding is enforced per provider (not per criteria).
BR-5a. `identityNullifier` is derived from `(providerId, providerUserId)`; therefore provider-scoped enforcement is correct by construction.
BR-6. After unbind, all historical data for that nullifier is wiped.
BR-7. Unbind is blocked if a cooldown window since the last bind or last result submission has not elapsed.
BR-8. Unbind is blocked if any downstream system has locked the result.
BR-9. Binding occurs only through successful on-chain verification result submission; no manual bind operation exists.
BR-10. When a verification result is submitted on-chain and all other requirements are met, the system automatically binds the `identityNullifier` to `proofUser` if it is not already bound.
BR-11. The system provides a contract call to notify that an `identityNullifier` is in use or no longer in use by downstream services.
BR-12. The downstream usage notification call is restricted to whitelisted callers.
BR-13. The contract treats `inUse` as a downstream-defined lock signal that prevents unbind; it does not interpret business meaning beyond lock/unlock.
BR-14. Multiple downstream services may use the same verification result concurrently.
BR-15. An `identityNullifier` is considered locked if any whitelisted downstream service reports `inUse = true` for that nullifier.
BR-16. A verification result can be reused by multiple downstream services without regenerating proofs.
BR-17. Downstream locks do not use TTL-based auto-expiry; unlock is explicit by the locking service.
BR-18. The system provides a governed deadlock-resolution path for exceptional cases where downstream locks are not released.

## Security Requirements
SR-1. Proofs are user-bound by including `proofUser` in the zkVM output.
SR-2. Verification results are stored under `proofUser` regardless of transaction sender.
SR-3. Submissions are rejected where `identityNullifier` is already bound to another user.
SR-4. The system validates `providerCriteriaId` (on-chain or off-chain registry).
SR-5. Replay enforcement uses `timestamp` and `nonce` as replay-prevention fields.
SR-6. The contract rejects reused timestamps and reused nonces within replay domain scope.
SR-6a. `proofHash` is not required for replay protection.
SR-7. The contract rejects proofs older than the latest stored result for the same user and criteria.
SR-8. Relayer-submitted transactions are allowed while still enforcing that results are stored under `proofUser`.
SR-9. There is no manual bind operation; binding occurs only via successful verification result submission.
SR-10. Downstream lock/unlock notifications are accepted only from whitelisted callers to prevent malicious locking or unlocking of bindings.
SR-11. The contract defines a canonical replay domain `ReplayDomain`.
SR-12. `ReplayDomain` includes at least `chainId`, `contractAddress`, `proofUser`, `identityNullifier`, and `providerCriteriaId`.
SR-13. For proxy-based deployments, `contractAddress` in `ReplayDomain` is the proxy address (`address(this)` at execution).
SR-14. Nonce replay checks use `ReplayDomain` and reject reuse only within the same `ReplayDomain`.
SR-15. Timestamp replay checks use `ReplayDomain` and reject replay only within the same `ReplayDomain`.

## Verification Result Requirements
VR-1. Each stored result includes: `providerCriteriaId`, `identityNullifier`, `providerOutput`, and any extra required fields.
VR-2. Users may have multiple results per `providerCriteriaId`.
VR-3. Only the latest result per `(user, providerCriteriaId)` is active.
VR-4. Multi-criteria proofs are supported; combined output is stored as an opaque blob, and downstream services are responsible for interpretation and usage.
VR-5. The on-chain contract treats `providerOutput` as opaque bytes even though zkVM defines explicit encoding/decoding for downstream interpretation.
VR-6. The system always stores the full `providerOutput` blob on-chain for downstream business-logic usage.
VR-7. The system enforces maximum `providerOutput` size limits to prevent calldata/storage abuse.
VR-8. The system supports versioned/schema-prefixed `providerOutput` envelopes and rejects malformed envelopes.
VR-9. Allowed `providerOutput` schema/version is bound to `providerCriteriaId`.
VR-10. Each verification result exposes `timestamp` as part of stored and queryable result data.

## Functional Requirements
FR-1. Implement `unbindIdentity(user, identityNullifier)`.
FR-2. `unbindIdentity` fails if the cooldown window since last bind or last result submission has not elapsed.
FR-3. `unbindIdentity` fails if any downstream system has locked the result.
FR-4. `unbindIdentity` is callable only by `proofUser`.
FR-5. `unbindIdentity` wipes all historical data for the nullifier upon success.
FR-5a. Implement `reportDownstreamUsage(identityNullifier, inUse)` to lock/unlock unbind based on downstream usage.
FR-5b. `reportDownstreamUsage` is callable only by whitelisted addresses.
FR-5c. `reportDownstreamUsage` only toggles the unbind lock state; any semantic meaning of `inUse` is defined by the downstream service.
FR-5d. For lock creation (`inUse = true`), the contract verifies `identityNullifier` is actively bound.
FR-5e. The system tracks locks by `(service, identityNullifier)` and allows unlock only by the same service for that nullifier.
FR-5f. Unbind is blocked if any active downstream lock exists for the same `identityNullifier`.
FR-5g. User signature per downstream service is not required for lock creation.
FR-5h. Lock policy is `whitelisted service + bound identityNullifier`.
FR-5i. Repeated `inUse = true` calls from the same service for the same `identityNullifier` are idempotent.
FR-5j. The contract does not apply timeout-based or TTL-based automatic unlock.
FR-5k. Unlock (`inUse = false`) validates lock ownership and lock existence for `identityNullifier` only.
FR-5l. On successful unbind, lock state for that `identityNullifier` is cleared together with binding/result data to prevent orphan locks.
FR-5m. Any privileged wipe or migration path that removes result/binding data also clears corresponding downstream lock state.
FR-5n. Implement a governance-controlled emergency lock-resolution operation for exceptional deadlock cases.
FR-5o. Emergency lock resolution is restricted by governance authorization policy and requires an explicit reason code.
FR-5p. Emergency lock resolution emits auditable events with caller, reason, affected `identityNullifier` lock, and timestamp.
FR-5q. The system exposes lock-state observability (events and/or queries) sufficient to identify long-lived locks and operational deadlock risk.
FR-5r. Each service has at most one lock per `identityNullifier`.
FR-6. Implement `submitVerificationResult(proof)`.
FR-7. `submitVerificationResult` verifies the zkVM proof.
FR-8. `submitVerificationResult` validates `providerCriteriaId` if it is maintained on-chain.
FR-9. `submitVerificationResult` enforces impersonation resistance and replay protection.
FR-10. `submitVerificationResult` stores results under `proofUser`.
FR-11. `submitVerificationResult` updates the latest-result pointer per `(user, providerCriteriaId)`.
FR-12. `submitVerificationResult` fails if:
FR-12a. the `identityNullifier` is bound to another user, or
FR-12b. the `providerCriteriaId` is unregistered (if on-chain), or
FR-12c. the proof is stale or replayed, or
FR-12d. the proof is invalid.
FR-13. `submitVerificationResult` auto-binds `identityNullifier` to `proofUser` if no binding exists and all other requirements are met.
FR-14. The contract uses deterministic, standardized error codes/messages for all rejection paths.
FR-15. Implement whitelist governance functions: `addWhitelistedService`, `removeWhitelistedService`, and `setWhitelistedServiceStatus`.
FR-16. Whitelist governance actions are restricted to governance-authorized roles.
FR-17. Whitelisted service status supports at least `ACTIVE`, `PAUSED`, and `REVOKED`.
FR-18. `reportDownstreamUsage` rejects callers whose whitelist status is not `ACTIVE`.
FR-19. The governance framework supports emergency revoke for compromised services.
FR-20. Whitelist governance actions emit auditable events with actor, target service, old/new status, reason, and timestamp.
FR-21. Revoked services do not create new locks.
FR-22. Policy for existing locks from revoked services is governance-resolvable and auditable.
FR-23. `submitVerificationResult` validates `providerOutput` length against configured limits.
FR-24. `submitVerificationResult` validates `providerOutput` envelope structure, including prefix/version/schema and payload length consistency.
FR-25. `submitVerificationResult` enforces schema/version allowlist mapping for the given `providerCriteriaId`.
FR-26. Auto-binding on `submitVerificationResult` enforces provider-specific binding multiplicity policy for the corresponding provider.

## Query Requirements
QR-1. Implement `getLatestResult(user, providerCriteriaId)`.
QR-2. `getLatestResult` returns the latest result for the user and criteria.
QR-3. The system may implement `getAllResults(user, providerCriteriaId)`.
QR-4. If `getAllResults` is not implemented, event-based indexing is supported.
QR-5. The system provides at least one full-history retrieval path for `(user, providerCriteriaId)`: on-chain `getAllResults` or event-based indexing.

## Constraints and Invariants
CI-1. One Web2 identity maps to only one Web3 user.
CI-2. No raw Web2 IDs are stored on-chain.
CI-3. Only `identityNullifier` is stored on-chain.
CI-4. `identityNullifier` is deterministic.
CI-5. `providerCriteriaId` is deterministic.
CI-6. zkVM outputs are deterministic.
CI-7. The system produces audit artifacts for external review, including logs and evidence of domain separation and replay protection checks.

## Non-Functional Requirements
NFR-1. The system is gas-efficient.
NFR-2. The system is upgradeable.
NFR-3. The system uses a modular design.
NFR-4. The system defines governance for provider and criteria changes, including approval authority and versioning to prevent fragmentation.
NFR-5. The governance framework defines deadlock-resolution policy, escalation workflow, and accountability for emergency lock actions.
NFR-6. The governance framework defines whitelist operational controls, including review cadence, escalation, and emergency response procedures.

## Performance Evaluation Matrix
PM-1. Metrics include gas cost per operation (`bind` via submit, `unbind`, `submitVerificationResult`, `getLatestResult`, and downstream usage reporting), storage growth per result, and query latency (on-chain view call and off-chain indexing).
PM-2. Measurements are reported for typical and worst-case inputs (including multi-criteria proofs and large `providerOutput` blobs).
PM-3. The report identifies primary gas/storage cost drivers and optimization opportunities.
PM-4. The report compares results against at least one baseline (prior version or alternative implementation) and explains deltas.

## Acceptance Criteria
AC-1. Identity Binding (Provider-Level)
AC-1.1. Binding correctness: When a verification result is submitted on-chain, the contract stores the binding only if:
AC-1.1a. the `identityNullifier` is not already bound to another user.
AC-1.2. Unbinding correctness: After `unbindIdentity(user, identityNullifier)`, the user must:
AC-1.2a. no longer have an active binding for that nullifier, and
AC-1.2b. have all historical verification results for that nullifier wiped out.
AC-1.3. Nullifier determinism: For the same `(providerId, providerUserId)`, the zkVM always outputs the same `identityNullifier`.
AC-1.4. Auto-bind on submit: When a verification result is submitted on-chain and passes all checks, the contract auto-binds the `identityNullifier` to `proofUser` if no binding exists.
AC-1.5. Downstream usage signaling: The contract exposes a lock/unlock notification call for `identityNullifier` that marks it as in-use or no longer in-use.
AC-1.6. Whitelist enforcement: Only whitelisted callers can lock or unlock downstream usage, and unauthorized calls are rejected.
AC-1.7. Nullifier validation (lock path): For `inUse = true`, `reportDownstreamUsage` requires `identityNullifier` and rejects nullifiers that are not actively bound.
AC-1.8. Ownership validation: Locks are owned by `(service, identityNullifier)` and can only be unlocked by the same service for that nullifier.
AC-1.9. Lock policy: User per-service signatures are not required; lock creation succeeds only with whitelisted caller and bound `identityNullifier`.
AC-1.10. No TTL auto-unlock: Locks remain active until explicitly unlocked by the same service owner for that nullifier.
AC-1.11. Unlock lifecycle: Unlock succeeds based on existing owned lock state for `identityNullifier`, and fails if no owned lock exists.
AC-1.12. Unbind cleanup: Successful unbind clears corresponding downstream locks so no orphan lock can block future operations.
AC-1.13. Deadlock resolution: Governance-authorized emergency lock resolution can clear stuck locks and unblock unbind in exceptional cases.
AC-1.14. Emergency controls: Emergency lock resolution rejects unauthorized callers and requires explicit reason metadata.
AC-1.15. Observability: Lock-state telemetry supports detection and review of long-lived locks and emergency actions.
AC-1.16. Provider-specific multiplicity: For `Binance`, a user can have at most one active bound nullifier; attempts to bind another Binance nullifier are rejected until prior unbind.
AC-1.17. Provider-specific multiplicity: For providers other than `Binance` (including `GitHub`), a user can hold multiple active bound nullifiers if each nullifier is free to bind.
AC-1.18. Multi-provider concurrency: A user can hold active bindings for different providers at the same time (for example, Binance + GitHub).
AC-2. Verification Result Submission
AC-2.1. Proof-user attribution: When a proof is submitted, the verification result is stored under `proofUser` (the user inside the proof), not `msg.sender`.
AC-2.2. Binding validation: The contract rejects proofs where `identityNullifier` is bound to a different user.
AC-2.3. Provider-criteria consistency: The contract rejects proofs where `providerCriteriaId` is not valid.
AC-2.4. Multiple results: The system allows multiple verification results for the same `(user, providerCriteriaId)` pair.
AC-2.5. Latest pointer: The system correctly updates the latest-result pointer.
AC-2.6. Full blob storage: The full `providerOutput` blob is persisted on-chain for accepted proofs.
AC-2.7. Size limits: Proofs with oversized `providerOutput` are rejected deterministically.
AC-2.8. Envelope validity: Proofs with malformed or inconsistent `providerOutput` envelope fields are rejected deterministically.
AC-2.9. Schema binding: Proofs are rejected when `providerOutput` schema/version is not permitted for the submitted `providerCriteriaId`.
AC-3. zkVM Output Validation
AC-3.1. Required fields: The contract rejects proofs missing any of:
AC-3.1a. `proofUser`
AC-3.1b. `providerCriteriaId`
AC-3.1c. `identityNullifier`
AC-3.1d. `providerOutput`
AC-3.1e. `timestamp` and `nonce`
AC-3.2. Nullifier correctness: The zkVM proves `identityNullifier = hash(providerId, providerUserId)`.
AC-3.3. Criteria correctness: The zkVM proves that the evaluation result corresponds to the ruleset represented by `providerCriteriaId`.
AC-3.4. Timestamp exposure: Accepted verification results expose `timestamp` in stored/queryable result data.
AC-4. Anti-Impersonation Guarantees
AC-4.1. No proof theft: If Alice submits a proof generated for Bob, the result is stored under Bob, not Alice.
AC-4.2. No identity hijacking: Alice cannot bind Bob's `identityNullifier` to herself.
AC-4.3. No cross-provider impersonation: A nullifier from provider A is not accepted for provider B.
AC-5. Replay Attack Resistance
AC-5.1. Freshness enforcement: The contract rejects proofs if:
AC-5.1a. the timestamp is older than the latest stored result, or
AC-5.1b. the timestamp is replayed within the same replay domain, or
AC-5.1c. the nonce is replayed within the same replay domain.
AC-5.2. Rebinding invalidation: After a user unbinds and rebinds to a new `identityNullifier`, all proofs tied to the old nullifier are rejected.
AC-5.3. Nonce scope: The same nonce may be reused across different `ReplayDomain` values, and is rejected only when reused within the same `ReplayDomain`.
AC-5.4. Timestamp scope: The same timestamp may be reused across different `ReplayDomain` values, and is rejected only when replayed within the same `ReplayDomain`.
AC-5.5. Proxy scope: For proxy deployments, replay-domain `contractAddress` uses the proxy address.
AC-6. Querying
AC-6.1. Latest result: `getLatestResult(user, providerCriteriaId)` returns the most recent result, or empty if none exists.
AC-6.2. Full history (on-chain option): If implemented, `getAllResults(user, providerCriteriaId)` returns all stored results in chronological order.
AC-6.3. Full history (indexing option): If `getAllResults` is not implemented, event-based indexing provides full chronological history for `(user, providerCriteriaId)`.
AC-7. Privacy Guarantees
AC-7.1. No raw Web2 IDs: The contract does not store or emit raw Web2 user IDs.
AC-7.2. Nullifier-only identity: All identity references use `identityNullifier`.
AC-8. Determinism and Consistency
AC-8.1. Deterministic nullifier: Same `(providerId, providerUserId)` always produces the same `identityNullifier`.
AC-8.2. Deterministic providerCriteriaId: Same ruleset always produces the same `providerCriteriaId`.
AC-8.3. Deterministic zkVM output: zkVM produces identical outputs for identical inputs.
AC-8.4. Audit artifacts exist for external review, including logs and evidence of domain separation and replay protection checks.
AC-9. Error signaling: Rejection paths emit standardized, deterministic error codes/messages that downstream systems can rely on.
AC-10. Whitelist governance: Unauthorized whitelist management actions are rejected, and authorized actions are event-auditable.
AC-11. Service status enforcement: Only `ACTIVE` services can call `reportDownstreamUsage`; `PAUSED` and `REVOKED` services are rejected.
AC-12. Emergency revoke: Governance can revoke a compromised service immediately, and the action is logged with reason metadata.
AC-13. Revoked-lock handling: Existing locks from revoked services are handled via governed, auditable resolution workflow.

## Standard Error Codes
E-1. `ERR_NULLIFIER_BOUND`: `identityNullifier` is already bound to another user.
E-2. `ERR_PROVIDER_CRITERIA_UNREGISTERED`: `providerCriteriaId` is not registered (when on-chain registry is used).
E-3. `ERR_PROOF_REPLAY`: proof is stale, timestamp is older than latest, or timestamp/nonce is replayed within replay domain scope.
E-4. `ERR_PROOF_INVALID`: zkVM proof verification failed.
E-5. `ERR_UNBIND_COOLDOWN`: unbind cooldown window has not elapsed.
E-6. `ERR_DOWNSTREAM_LOCKED`: downstream usage lock is active.
E-7. `ERR_UNAUTHORIZED`: caller is not authorized (for example, non-whitelisted or not `proofUser`).
E-8. `ERR_INPUT_MALFORMED`: input data or required fields are missing or malformed.
E-9. `ERR_NULLIFIER_NOT_BOUND`: `identityNullifier` is not actively bound.
E-10. `ERR_DOWNSTREAM_UNLOCK_OWNER_MISMATCH`: unlock attempted by a different service owner for the same `identityNullifier`.
E-11. `ERR_DOWNSTREAM_NOT_LOCKED`: unlock attempted when no owned active lock exists.
E-12. `ERR_GOVERNANCE_REQUIRED`: operation requires governance authorization.
E-13. `ERR_SERVICE_NOT_ACTIVE`: caller service is not in `ACTIVE` whitelist status.
E-14. `ERR_PROVIDER_OUTPUT_TOO_LARGE`: `providerOutput` exceeds configured size limit.
E-15. `ERR_PROVIDER_OUTPUT_MALFORMED`: `providerOutput` envelope or length fields are invalid.
E-16. `ERR_PROVIDER_OUTPUT_SCHEMA_UNSUPPORTED`: `providerOutput` schema/version is not allowed for `providerCriteriaId`.

## Auditability Requirements
AR-1. Emit events for downstream lock create/update/release operations with `service`, `identityNullifier`, and `inUse`.
AR-2. Keep audit records sufficient to reconstruct active-lock state and lock ownership history per `identityNullifier`.
AR-3. Emit dedicated events for emergency lock-resolution actions, including governance actor, reason code, and affected `identityNullifier` lock.
AR-4. Emit dedicated events for whitelist governance actions, including actor, service, status transition, reason, and timestamp.

## Replay Scope Rationale
RSR-1. Global nonce uniqueness is not required because unrelated users and criteria may generate proofs concurrently.
RSR-2. Domain-scoped timestamp/nonce checks prevent false rejection across independent users/criteria while preserving replay protection within the same verification context.
