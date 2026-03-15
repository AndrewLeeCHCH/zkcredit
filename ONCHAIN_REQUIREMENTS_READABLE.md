# Web2–Web3 Identity Verification System Requirements (Human-Readable)

## Purpose
The system provides trust-minimized verification of Web2 attributes, preserves user privacy when binding Web2 identity to Web3 addresses, securely stores verification results, prevents impersonation and replay attacks, and maps proofs to the correct provider and criteria set.

## Core Concepts (Plain Language)
- **Provider**: A Web2 platform (for example, a social network or exchange).
- **Criteria**: A ruleset for verification (for example, account age or KYC tier).
- **providerCriteriaId**: A deterministic ID that uniquely represents the provider + criteria.
- **identityNullifier**: A deterministic hash derived from `(providerId, providerUserId)` so raw Web2 IDs never appear on-chain.
- **proofUser**: The Web3 address embedded inside the proof; this is the owner of the result, regardless of who submits the transaction.
- **Latest result**: For each `(user, providerCriteriaId)` pair, only the most recent result is active.

## Binding Requirements
- A Web3 user can bind different Web2 identities per `providerId`.
- Binding multiplicity follows provider-specific policy.
- For `Binance`, one Web3 user has only one active bound nullifier at a time.
- For providers other than `Binance` (including `GitHub`), one Web3 user may have multiple active bound nullifiers, subject to nullifier uniqueness constraints.
- A Web2 identity (via `identityNullifier`) can bind to only one Web3 user.
- Binding is stored as `identityNullifier -> user`.
- `identityNullifier` is derived from `(providerId, providerUserId)`, so provider-scoped enforcement is correct by construction.
- Binding is enforced per provider, not per criteria.
- Binding happens only via a successful on-chain verification result submission; there is no manual bind flow.
- Unbind is supported and removes all historical data tied to that nullifier.
- Unbind is blocked if the cooldown window since the last bind or last result submission has not elapsed.
- Unbind is blocked if any downstream system has locked the result.
- When a verification result is submitted on-chain and all other requirements are met, the system automatically binds the `identityNullifier` to `proofUser` if it is not already bound.
- The contract provides a downstream usage notification call to mark an `identityNullifier` as in use or no longer in use.
- The downstream usage notification call is restricted to whitelisted callers.
- The contract treats `inUse` as a downstream-defined lock signal that prevents unbind; it does not interpret business meaning beyond lock/unlock.
- Multiple downstream services may use the same verification result concurrently.
- A nullifier is considered locked if any whitelisted downstream service reports `inUse = true` for that nullifier.
- A verification result can be reused by multiple downstream services without regenerating proofs.
- Downstream locks do not use TTL-based auto-expiry; unlock is explicit by the locking service.
- The system provides a governed deadlock-resolution path for exceptional cases where downstream locks are not released.

## Security Requirements
- Proofs must include `proofUser` and results must be stored under `proofUser`, not `msg.sender`.
- The system must reject submissions where `identityNullifier` is already bound to another user.
- `providerCriteriaId` must be validated (on-chain or off-chain registry).
- Replay enforcement uses `timestamp` and `nonce` as replay-prevention fields.
- The contract rejects reused timestamps and reused nonces within replay domain scope.
- `proofHash` is not required for replay protection.
- The contract must reject proofs older than the latest stored result for the same user and criteria.
- Relayers are allowed, but results still belong to `proofUser`.
- There is no manual bind flow; binding can only occur via a successful verification result.
- Downstream lock/unlock notifications are accepted only from whitelisted callers to prevent malicious locking or unlocking of bindings.
- The contract defines a canonical replay domain `ReplayDomain`.
- `ReplayDomain` includes at least `chainId`, `contractAddress`, `proofUser`, `identityNullifier`, and `providerCriteriaId`.
- For proxy-based deployments, `contractAddress` in `ReplayDomain` is the proxy address (`address(this)` at execution).
- Nonce replay checks use `ReplayDomain` and reject reuse only within the same `ReplayDomain`.
- Timestamp replay checks use `ReplayDomain` and reject replay only within the same `ReplayDomain`.

## Verification Result Requirements
- Each stored result includes `providerCriteriaId`, `identityNullifier`, `providerOutput`, and any other required fields.
- Users can have multiple results for the same `providerCriteriaId`.
- Only the latest result per `(user, providerCriteriaId)` is active.
- Multi-criteria proofs are supported; their combined output is stored as an opaque blob, and downstream services handle interpretation and usage.
- The on-chain contract treats `providerOutput` as opaque bytes even though zkVM defines explicit encoding/decoding for downstream interpretation.
- The full `providerOutput` blob is always stored on-chain for downstream business-logic usage.
- `providerOutput` size limits are enforced to prevent calldata/storage abuse.
- `providerOutput` uses a versioned/schema-prefixed envelope, and malformed envelopes are rejected.
- Allowed `providerOutput` schema/version is bound to `providerCriteriaId`.
- Each verification result exposes `timestamp` as part of stored/queryable result data.

## Functional Requirements
- Implement `unbindIdentity(user, identityNullifier)`.
- Reject `unbindIdentity` if the cooldown window since last bind or last result submission has not elapsed.
- Reject `unbindIdentity` if any downstream system has locked the result.
- Allow `unbindIdentity` only for `proofUser`.
- Wipe all historical data for the nullifier on successful unbind.
- Implement `reportDownstreamUsage(identityNullifier, inUse)` to lock/unlock unbind based on downstream usage.
- Restrict `reportDownstreamUsage` to whitelisted addresses.
- `reportDownstreamUsage` only toggles the unbind lock state; any semantic meaning of `inUse` is defined by the downstream service.
- For lock creation (`inUse = true`), the contract verifies `identityNullifier` is actively bound.
- Locks are tracked by `(service, identityNullifier)`, and only the same service can unlock for that nullifier.
- Unbind is blocked if any active downstream lock exists for the same `identityNullifier`.
- User signatures per downstream service are not required for lock creation.
- Lock policy is `whitelisted service + bound identityNullifier`.
- Repeated `inUse = true` calls from the same service for the same `identityNullifier` are idempotent.
- The contract does not apply timeout-based or TTL-based automatic unlock.
- Unlock (`inUse = false`) validates lock ownership and lock existence for `identityNullifier` only.
- Successful unbind clears lock state for that `identityNullifier` together with binding/result data to prevent orphan locks.
- Any privileged wipe or migration path that removes result/binding data also clears corresponding downstream lock state.
- Implement a governance-controlled emergency lock-resolution operation for exceptional deadlock cases.
- Emergency lock resolution is restricted by governance authorization policy and requires an explicit reason code.
- Emergency lock resolution emits auditable events with caller, reason, affected `identityNullifier` lock, and timestamp.
- Expose lock-state observability (events and/or queries) sufficient to identify long-lived locks and operational deadlock risk.
- Each service has at most one lock per `identityNullifier`.
- Implement `submitVerificationResult(proof)`.
- Verify zkVM proofs.
- Validate `providerCriteriaId` if the registry is on-chain.
- Enforce impersonation resistance and replay protection.
- Store results under `proofUser`.
- Update the latest-result pointer per `(user, providerCriteriaId)`.
- Reject `submitVerificationResult` if `identityNullifier` is bound to another user.
- Reject `submitVerificationResult` if `providerCriteriaId` is unregistered (when on-chain).
- Reject `submitVerificationResult` if the proof is stale or replayed.
- Reject `submitVerificationResult` if the proof is invalid.
- Auto-bind `identityNullifier` to `proofUser` on successful submission when no binding exists.
- Use deterministic, standardized error codes/messages for all rejection paths.
- Implement whitelist governance functions: `addWhitelistedService`, `removeWhitelistedService`, and `setWhitelistedServiceStatus`.
- Restrict whitelist governance actions to governance-authorized roles.
- Support whitelist statuses: `ACTIVE`, `PAUSED`, and `REVOKED`.
- Reject `reportDownstreamUsage` when caller status is not `ACTIVE`.
- Support emergency revoke for compromised services.
- Emit auditable whitelist-governance events with actor, service, old/new status, reason, and timestamp.
- Prevent revoked services from creating new locks.
- Handle existing locks from revoked services through governed, auditable resolution workflow.
- Validate `providerOutput` length against configured limits in `submitVerificationResult`.
- Validate `providerOutput` envelope structure in `submitVerificationResult`, including prefix/version/schema and payload length consistency.
- Enforce schema/version allowlist mapping for the submitted `providerCriteriaId`.
- Enforce provider-specific binding multiplicity policy during auto-binding on `submitVerificationResult`.

## Query Requirements
- Implement `getLatestResult(user, providerCriteriaId)` that returns the latest result or empty if none.
- Optionally implement `getAllResults(user, providerCriteriaId)` for full history.
- If `getAllResults` is not implemented, provide event-based indexing.
- Provide at least one full-history retrieval path for `(user, providerCriteriaId)`: on-chain `getAllResults` or event-based indexing.

## Constraints and Invariants
- One Web2 identity maps to only one Web3 user.
- No raw Web2 IDs are stored on-chain.
- Only `identityNullifier` is stored on-chain.
- `identityNullifier` is deterministic.
- `providerCriteriaId` is deterministic.
- zkVM outputs are deterministic.
- Audit artifacts exist for external review, including logs and evidence of domain separation and replay protection checks.

## Non-Functional Requirements
- Gas-efficient.
- Upgradeable.
- Modular design.
- Governance exists for provider and criteria changes, including approval authority and versioning to prevent fragmentation.
- Governance also defines deadlock-resolution policy, escalation workflow, and accountability for emergency lock actions.
- Governance also defines whitelist operational controls, including review cadence, escalation, and emergency response procedures.

## Performance Evaluation Matrix
- Metrics include gas cost per operation (bind via submit, unbind, submitVerificationResult, getLatestResult, downstream usage reporting), storage growth per result, and query latency (on-chain view call and off-chain indexing).
- Measurements are reported for typical and worst-case inputs (including multi-criteria proofs and large `providerOutput` blobs).
- The report identifies primary gas/storage cost drivers and optimization opportunities.
- The report compares results against at least one baseline (prior version or alternative implementation) and explains deltas.

## Acceptance Criteria (User-Focused)
### Identity Binding (Provider-Level)
- Binding correctness: Binding is created only by a successful on-chain verification result and only if the nullifier is not bound to another user.
- Unbinding correctness: After `unbindIdentity(user, identityNullifier)`, there is no active binding and all historical results for that nullifier are wiped.
- Nullifier determinism: For the same `(providerId, providerUserId)`, zkVM always outputs the same `identityNullifier`.
- Downstream usage signaling: The contract exposes a lock/unlock notification call for `identityNullifier` that marks it as in-use or no longer in-use.
- Whitelist enforcement: Only whitelisted callers can lock or unlock downstream usage, and unauthorized calls are rejected.
- Nullifier validation (lock path): For `inUse = true`, `reportDownstreamUsage` requires `identityNullifier` and rejects nullifiers that are not actively bound.
- Ownership validation: Locks are owned by `(service, identityNullifier)` and can only be unlocked by the same service for that nullifier.
- Lock policy: User per-service signatures are not required; lock creation succeeds only with whitelisted caller and bound `identityNullifier`.
- No TTL auto-unlock: Locks remain active until explicitly unlocked by the same service owner for that nullifier.
- Unlock lifecycle: Unlock succeeds based on existing owned lock state for `identityNullifier`, and fails if no owned lock exists.
- Unbind cleanup: Successful unbind clears corresponding downstream locks so no orphan lock can block future operations.
- Deadlock resolution: Governance-authorized emergency lock resolution can clear stuck locks and unblock unbind in exceptional cases.
- Emergency controls: Emergency lock resolution rejects unauthorized callers and requires explicit reason metadata.
- Observability: Lock-state telemetry supports detection and review of long-lived locks and emergency actions.
- Provider-specific multiplicity: For `Binance`, a user can have at most one active bound nullifier; attempts to bind another Binance nullifier are rejected until prior unbind.
- Provider-specific multiplicity: For providers other than `Binance` (including `GitHub`), a user can hold multiple active bound nullifiers if each nullifier is free to bind.
- Multi-provider concurrency: A user can hold active bindings for different providers at the same time (for example, Binance + GitHub).

### Verification Result Submission
- Proof-user attribution: Results are stored under `proofUser`, not `msg.sender`.
- Binding validation: Proofs are rejected if the nullifier is bound to a different user.
- Provider–criteria consistency: Proofs are rejected if `providerCriteriaId` is invalid.
- Multiple results: Multiple results are allowed for the same `(user, providerCriteriaId)`.
- Latest pointer: The latest-result pointer is updated correctly.
- Full blob storage: The full `providerOutput` blob is persisted on-chain for accepted proofs.
- Size limits: Proofs with oversized `providerOutput` are rejected deterministically.
- Envelope validity: Proofs with malformed or inconsistent `providerOutput` envelope fields are rejected deterministically.
- Schema binding: Proofs are rejected when `providerOutput` schema/version is not permitted for the submitted `providerCriteriaId`.

### zkVM Output Validation
- Required fields: Proofs without `proofUser`, `providerCriteriaId`, `identityNullifier`, `providerOutput`, and replay-prevention fields are rejected.
- Nullifier correctness: zkVM proves `identityNullifier = hash(providerId, providerUserId)`.
- Criteria correctness: zkVM proves the evaluation matches the ruleset represented by `providerCriteriaId`.
- Timestamp exposure: Accepted verification results expose `timestamp` in stored/queryable result data.

### Anti-Impersonation Guarantees
- No proof theft: Submitting someone else’s proof stores the result under the original `proofUser`.
- No identity hijacking: A user cannot bind another person’s `identityNullifier`.
- No cross-provider impersonation: A nullifier from provider A is never accepted for provider B.

### Replay Attack Resistance
- Freshness enforcement: The contract rejects proofs with old timestamps, reused nonces, or reused proof hashes.
- Freshness enforcement: The contract rejects proofs with old timestamps and replayed timestamps/nonces within the same replay domain.
- Rebinding invalidation: After unbind and rebind, proofs tied to the old nullifier are rejected.
- Error signaling: Rejection paths emit standardized, deterministic error codes/messages that downstream systems can rely on.
- Nonce scope: The same nonce may be reused across different `ReplayDomain` values, and is rejected only when reused within the same `ReplayDomain`.
- Timestamp scope: The same timestamp may be reused across different `ReplayDomain` values, and is rejected only when replayed within the same `ReplayDomain`.
- Proxy scope: For proxy deployments, replay-domain `contractAddress` uses the proxy address.

### Querying
- Latest result: `getLatestResult(user, providerCriteriaId)` returns the most recent result or empty.
- Full history (on-chain option): If implemented, `getAllResults(user, providerCriteriaId)` returns results in chronological order.
- Full history (indexing option): If `getAllResults` is not implemented, event-based indexing provides full chronological history for `(user, providerCriteriaId)`.

### Privacy Guarantees
- No raw Web2 IDs on-chain.
- Identity references use `identityNullifier` only.

### Determinism and Consistency
- Deterministic nullifier for the same `(providerId, providerUserId)`.
- Deterministic `providerCriteriaId` for the same ruleset.
- Deterministic zkVM output for identical inputs.
- Audit artifacts exist for external review, including logs and evidence of domain separation and replay protection checks.
- Whitelist governance: Unauthorized whitelist management actions are rejected, and authorized actions are event-auditable.
- Service status enforcement: Only `ACTIVE` services can call `reportDownstreamUsage`; `PAUSED` and `REVOKED` services are rejected.
- Emergency revoke: Governance can revoke a compromised service immediately, and the action is logged with reason metadata.
- Revoked-lock handling: Existing locks from revoked services are handled via governed, auditable resolution workflow.

## Standard Error Codes
- `ERR_NULLIFIER_BOUND`: `identityNullifier` is already bound to another user.
- `ERR_PROVIDER_CRITERIA_UNREGISTERED`: `providerCriteriaId` is not registered (when on-chain registry is used).
- `ERR_PROOF_REPLAY`: proof is stale, timestamp is older than latest, or timestamp/nonce is replayed within replay domain scope.
- `ERR_PROOF_INVALID`: zkVM proof verification failed.
- `ERR_UNBIND_COOLDOWN`: unbind cooldown window has not elapsed.
- `ERR_DOWNSTREAM_LOCKED`: downstream usage lock is active.
- `ERR_UNAUTHORIZED`: caller is not authorized (e.g., non-whitelisted or not `proofUser`).
- `ERR_INPUT_MALFORMED`: input data or required fields are missing or malformed.
- `ERR_NULLIFIER_NOT_BOUND`: `identityNullifier` is not actively bound.
- `ERR_DOWNSTREAM_UNLOCK_OWNER_MISMATCH`: unlock attempted by a different service owner for the same `identityNullifier`.
- `ERR_DOWNSTREAM_NOT_LOCKED`: unlock attempted when no owned active lock exists.
- `ERR_GOVERNANCE_REQUIRED`: operation requires governance authorization.
- `ERR_SERVICE_NOT_ACTIVE`: caller service is not in `ACTIVE` whitelist status.
- `ERR_PROVIDER_OUTPUT_TOO_LARGE`: `providerOutput` exceeds configured size limit.
- `ERR_PROVIDER_OUTPUT_MALFORMED`: `providerOutput` envelope or length fields are invalid.
- `ERR_PROVIDER_OUTPUT_SCHEMA_UNSUPPORTED`: `providerOutput` schema/version is not allowed for `providerCriteriaId`.

## Auditability Requirements
- Emit events for downstream lock create/update/release operations with `service`, `identityNullifier`, and `inUse`.
- Keep audit records sufficient to reconstruct active-lock state and lock ownership history per `identityNullifier`.
- Emit dedicated events for emergency lock-resolution actions, including governance actor, reason code, and affected `identityNullifier` lock.
- Emit dedicated events for whitelist governance actions, including actor, service, status transition, reason, and timestamp.

## Replay Scope Rationale
- Global nonce uniqueness is not required because unrelated users and criteria may generate proofs concurrently.
- Domain-scoped timestamp/nonce checks prevent false rejection across independent users/criteria while preserving replay protection within the same verification context.
