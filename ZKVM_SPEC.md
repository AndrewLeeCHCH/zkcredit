# zkVM Identity Verification Spec

## Overview
This document defines zkVM-specific data definitions and output semantics for generating on-chain verifiable identity verification results.

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

## Output Schema
The zkVM output includes:
- `proofUser`: Web3 user address.
- `providerCriteriaId`.
- `identityNullifier`.
- `providerOutput`: provider/criteria-specific evaluation results. This may contain multi-criteria output encoded as an opaque blob for on-chain storage.
- `timestamp` and/or `nonce` and/or `proofHash`.

## Replay Inputs and Scope
- Nonce uniqueness is enforced per replay domain, not globally.
- The replay domain includes at least `proofUser`, `identityNullifier`, and `providerCriteriaId`, plus chain/contract context.
- Reuse of the same nonce is rejected only when reused within the same replay domain.

## Encoding and Decoding Contract
- zkVM defines explicit encoding/decoding rules for `providerOutput`.
- On-chain contracts treat `providerOutput` as opaque bytes and do not interpret business semantics.
- Downstream services decode and interpret `providerOutput` according to zkVM-defined rules.

## Determinism Guarantees
- For identical inputs and versions, zkVM output is deterministic.
- `identityNullifier` and `providerCriteriaId` are deterministic for identical source inputs.
