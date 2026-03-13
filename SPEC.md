# Web2–Web3 Identity Verification System Spec

## Overview
This document defines the data model, identifiers, and interfaces for a zkVM-verified identity attestation system that links Web2 attributes to Web3 wallet addresses.

## Terminology
- Provider: A Web2 platform where identity attributes originate. Identifier: `providerId`. Example: GitHub, Binance, Twitter.
- Provider Criteria: A specific verification rule set under a provider. Identifier: `providerCriteriaId`.
- Web2 User Unique ID: The platform-specific account ID (e.g., `github_id`, `steam_id`). Not stored on-chain and not revealed in zkVM output.
- Identity Nullifier: A masked, irreversible representation of the Web2 user ID, tied to the provider (not to criteria). Identifier: `identityNullifier`.
- Web3 User: An EVM wallet address.
- Verification Result: A zkVM-verified output containing `proofUser`, `providerCriteriaId`, `identityNullifier`, provider-specific outputs, and replay-prevention fields.

## Identifier Derivations
- `providerCriteriaId` is deterministically derived from `(providerId, criteriaDefinition)`.
- `identityNullifier` is deterministically derived from `(providerId, web2UserUniqueId)` and is provider-scoped.
- `providerCriteriaId` is provider-scoped because `providerId` is part of its derivation, so it does not collide across providers except by hash collision.

## zkVM Output Schema
The zkVM output includes:
- `proofUser`: Web3 user address
- `providerCriteriaId`
- `identityNullifier`
- `providerOutput`: provider/criteria-specific evaluation results
- `timestamp` and/or `nonce` and/or `proofHash`

## On-Chain Storage Model
- Identity binding:
  - Mapping of `(providerId, identityNullifier) -> user`.
- Results:
  - Mapping of `(user, providerCriteriaId) -> [results]` (append-only list).
  - Mapping of `(user, providerCriteriaId) -> latestResultId` (or latest index).
- Replay protection:
  - Sets for used `timestamp`, `nonce`, and `proofHash` as required by the proof format.
- Unbind control:
  - `lastBindTime` per `(identityNullifier)` or `(user, providerId)` as needed.
  - `lastResultSubmissionTime` per `(user, providerCriteriaId)` or `(user, providerId)` as needed.
  - `cooldownWindow` configuration.
  - Optional lock registry or flag indicating downstream lock status for a nullifier or result.

## Interfaces
Function signatures:
- `unbindIdentity(user, identityNullifier)`
- `submitVerificationResult(proof)`
- `getLatestResult(user, providerCriteriaId)`
- `getAllResults(user, providerCriteriaId)` (optional)

## Unbind Rules
- `unbindIdentity` is callable only by `proofUser` (the bound user).
- Unbind is blocked if the cooldown window since last bind or last result submission has not elapsed.
- Unbind is blocked if any downstream system has locked the result.
- On successful unbind, all historical data for the nullifier is wiped (binding and associated results).

## Relayer Support
- Transactions may be submitted by relayers.
- Storage is always keyed by `proofUser` from the zkVM output, not by transaction sender.

## Binding Behavior
- There is no manual bind operation; binding occurs only via successful on-chain verification result submission.
- On a successful `submitVerificationResult`, if no binding exists, the contract auto-binds `identityNullifier` to `proofUser` (provider-scoped).

## Result Selection
- The latest stored result per `(user, providerCriteriaId)` is considered the active result.
- Event-based indexing can be used to retrieve full history if `getAllResults` is omitted.

## Privacy and Determinism Guarantees
- No raw Web2 user IDs are stored or emitted on-chain.
- All identity references use `identityNullifier`.
- `identityNullifier` and `providerCriteriaId` are deterministic, and zkVM outputs are deterministic for identical inputs.
