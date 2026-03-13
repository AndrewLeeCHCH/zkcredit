# Web2–Web3 Identity Verification System Requirements (Human-Readable)

## Purpose
The system provides trust-minimized verification of Web2 attributes, preserves user privacy when binding Web2 identity to Web3 addresses, securely stores verification results, prevents impersonation and replay attacks, and maps proofs to the correct provider and criteria set.

## Core Concepts (Plain Language)
- **Provider**: A Web2 platform (for example, a social network or exchange).
- **Criteria**: A ruleset for verification (for example, account age or KYC tier).
- **providerCriteriaId**: A deterministic ID that uniquely represents the provider + criteria.
- **identityNullifier**: A deterministic hash derived from `(providerId, web2UserId)` so raw Web2 IDs never appear on-chain.
- **proofUser**: The Web3 address embedded inside the proof; this is the owner of the result, regardless of who submits the transaction.
- **Latest result**: For each `(user, providerCriteriaId)` pair, only the most recent result is active.

## Binding Requirements
- A Web3 user can bind different Web2 identities per `providerId`.
- A Web3 user can have multiple bindings across providers and within the same provider, because the nullifier does not reveal the provider.
- A Web2 identity (via `identityNullifier`) can bind to only one Web3 user.
- Binding is stored as `identityNullifier -> user`.
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
- A nullifier is considered locked if any whitelisted downstream service reports `inUse = true`.

## Security Requirements
- Proofs must include `proofUser` and results must be stored under `proofUser`, not `msg.sender`.
- The system must reject submissions where `identityNullifier` is already bound to another user.
- `providerCriteriaId` must be validated (on-chain or off-chain registry).
- Proofs must include replay-prevention fields: `timestamp` or `nonce` or `proofHash` (or a combination).
- The contract must reject reused timestamps, reused nonces, and reused proof hashes.
- The contract must reject proofs older than the latest stored result for the same user and criteria.
- Relayers are allowed, but results still belong to `proofUser`.
- There is no manual bind flow; binding can only occur via a successful verification result.
- Downstream lock/unlock notifications are accepted only from whitelisted callers to prevent malicious locking or unlocking of bindings.

## Verification Result Requirements
- Each stored result includes `providerCriteriaId`, `identityNullifier`, `providerOutput`, and any other required fields.
- Users can have multiple results for the same `providerCriteriaId`.
- Only the latest result per `(user, providerCriteriaId)` is active.
- Multi-criteria proofs are supported; their combined output is stored as an opaque blob, and downstream services handle interpretation and usage.
- The on-chain contract treats `providerOutput` as opaque bytes even though zkVM defines explicit encoding/decoding for downstream interpretation.

## Functional Requirements
- Implement `unbindIdentity(user, identityNullifier)`.
- Reject `unbindIdentity` if the cooldown window since last bind or last result submission has not elapsed.
- Reject `unbindIdentity` if any downstream system has locked the result.
- Allow `unbindIdentity` only for `proofUser`.
- Wipe all historical data for the nullifier on successful unbind.
- Implement `reportDownstreamUsage(identityNullifier, inUse)` to lock/unlock unbind based on downstream usage.
- Restrict `reportDownstreamUsage` to whitelisted addresses.
- `reportDownstreamUsage` only toggles the unbind lock state; any semantic meaning of `inUse` is defined by the downstream service.
- The system tracks downstream usage per whitelisted service and treats the nullifier as locked if any service reports `inUse = true`.
- A downstream service can only unlock (`inUse = false`) if it previously locked (`inUse = true`).
- Repeated `inUse = true` calls from the same service are idempotent.
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

## Query Requirements
- Implement `getLatestResult(user, providerCriteriaId)` that returns the latest result or empty if none.
- Optionally implement `getAllResults(user, providerCriteriaId)` for full history.
- If `getAllResults` is not implemented, provide event-based indexing.

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

## Performance Evaluation Matrix
- Metrics include gas cost per operation (bind via submit, unbind, submitVerificationResult, getLatestResult, downstream usage reporting), storage growth per result, and query latency (on-chain view call and off-chain indexing).
- Measurements are reported for typical and worst-case inputs (including multi-criteria proofs and large `providerOutput` blobs).
- The report identifies primary gas/storage cost drivers and optimization opportunities.
- The report compares results against at least one baseline (prior version or alternative implementation) and explains deltas.

## Acceptance Criteria (User-Focused)
### Identity Binding (Provider-Level)
- Binding correctness: Binding is created only by a successful on-chain verification result and only if the nullifier is not bound to another user.
- Unbinding correctness: After `unbindIdentity(user, identityNullifier)`, there is no active binding and all historical results for that nullifier are wiped.
- Nullifier determinism: For the same `(providerId, web2UserId)`, zkVM always outputs the same `identityNullifier`.
- Downstream usage signaling: The contract exposes a lock/unlock notification call that marks an `identityNullifier` as in-use or no longer in-use.
- Whitelist enforcement: Only whitelisted callers can lock or unlock downstream usage, and unauthorized calls are rejected.

### Verification Result Submission
- Proof-user attribution: Results are stored under `proofUser`, not `msg.sender`.
- Binding validation: Proofs are rejected if the nullifier is bound to a different user.
- Provider–criteria consistency: Proofs are rejected if `providerCriteriaId` is invalid.
- Multiple results: Multiple results are allowed for the same `(user, providerCriteriaId)`.
- Latest pointer: The latest-result pointer is updated correctly.

### zkVM Output Validation
- Required fields: Proofs without `proofUser`, `providerCriteriaId`, `identityNullifier`, `providerOutput`, and replay-prevention fields are rejected.
- Nullifier correctness: zkVM proves `identityNullifier = hash(providerId, web2UserId)`.
- Criteria correctness: zkVM proves the evaluation matches the ruleset represented by `providerCriteriaId`.

### Anti-Impersonation Guarantees
- No proof theft: Submitting someone else’s proof stores the result under the original `proofUser`.
- No identity hijacking: A user cannot bind another person’s `identityNullifier`.
- No cross-provider impersonation: A nullifier from provider A is never accepted for provider B.

### Replay Attack Resistance
- Freshness enforcement: The contract rejects proofs with old timestamps, reused nonces, or reused proof hashes.
- Rebinding invalidation: After unbind and rebind, proofs tied to the old nullifier are rejected.
- Error signaling: Rejection paths emit standardized, deterministic error codes/messages that downstream systems can rely on.

### Querying
- Latest result: `getLatestResult(user, providerCriteriaId)` returns the most recent result or empty.
- Full history: `getAllResults(user, providerCriteriaId)` returns results in chronological order (if implemented).

### Privacy Guarantees
- No raw Web2 IDs on-chain.
- Identity references use `identityNullifier` only.

### Determinism and Consistency
- Deterministic nullifier for the same `(providerId, web2UserId)`.
- Deterministic `providerCriteriaId` for the same ruleset.
- Deterministic zkVM output for identical inputs.
- Audit artifacts exist for external review, including logs and evidence of domain separation and replay protection checks.

## Standard Error Codes
- `ERR_NULLIFIER_BOUND`: `identityNullifier` is already bound to another user.
- `ERR_PROVIDER_CRITERIA_UNREGISTERED`: `providerCriteriaId` is not registered (when on-chain registry is used).
- `ERR_PROOF_REPLAY`: proof is stale, timestamp is older than latest, nonce reused, or `proofHash` reused.
- `ERR_PROOF_INVALID`: zkVM proof verification failed.
- `ERR_UNBIND_COOLDOWN`: unbind cooldown window has not elapsed.
- `ERR_DOWNSTREAM_LOCKED`: downstream usage lock is active.
- `ERR_UNAUTHORIZED`: caller is not authorized (e.g., non-whitelisted or not `proofUser`).
- `ERR_INPUT_MALFORMED`: input data or required fields are missing or malformed.
- `ERR_DOWNSTREAM_NOT_LOCKED`: downstream service attempted to unlock without a prior lock.
