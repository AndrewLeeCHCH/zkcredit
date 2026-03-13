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
- A Web2 identity (via `identityNullifier`) can bind to only one Web3 user.
- Binding is stored as `identityNullifier -> user`.
- Binding is enforced per provider, not per criteria.
- Unbind is supported and removes all historical data tied to that nullifier.
- Unbind is blocked if the cooldown window since the last bind or last result submission has not elapsed.
- Unbind is blocked if any downstream system has locked the result.

## Security Requirements
- Proofs must include `proofUser` and results must be stored under `proofUser`, not `msg.sender`.
- The system must reject submissions where `identityNullifier` is already bound to another user.
- `providerCriteriaId` must be validated (on-chain or off-chain registry).
- Proofs must include replay-prevention fields: `timestamp` or `nonce` or `proofHash` (or a combination).
- The contract must reject reused timestamps, reused nonces, and reused proof hashes.
- The contract must reject proofs older than the latest stored result for the same user and criteria.
- Relayers are allowed, but results still belong to `proofUser`.

## Verification Result Requirements
- Each stored result includes `providerCriteriaId`, `identityNullifier`, `providerOutput`, and any other required fields.
- Users can have multiple results for the same `providerCriteriaId`.
- Only the latest result per `(user, providerCriteriaId)` is active.

## Functional Requirements
- Implement `bindIdentity(user, identityNullifier)`.
- Reject `bindIdentity` if the `identityNullifier` is already bound to a different user.
- Implement `unbindIdentity(user, identityNullifier)`.
- Reject `unbindIdentity` if the cooldown window since last bind or last result submission has not elapsed.
- Reject `unbindIdentity` if any downstream system has locked the result.
- Allow `unbindIdentity` only for `proofUser`.
- Wipe all historical data for the nullifier on successful unbind.
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

## Non-Functional Requirements
- Gas-efficient.
- Upgradeable.
- Modular design.

## Acceptance Criteria (User-Focused)
### Identity Binding (Provider-Level)
- Binding correctness: `bindIdentity(user, identityNullifier)` succeeds only if the nullifier is not bound to another user and the user is not bound to a different nullifier.
- Unbinding correctness: After `unbindIdentity(user, identityNullifier)`, there is no active binding and all historical results for that nullifier are wiped.
- Nullifier determinism: For the same `(providerId, web2UserId)`, zkVM always outputs the same `identityNullifier`.

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
