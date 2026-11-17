---
eip: <to be assigned>
title: Consensus-Based Theft Recovery Mechanism
description: Redirects verified stolen ETH from designated addresses to a per-incident escrow under bounded, reversible rules
author: Paul K (@kaymyg)
discussions-to: <URL of your Ethereum Magicians thread>
status: Draft
type: Standards Track
category: Core
created: 2026-11-17
---

## Abstract

This EIP introduces a consensus-enforced recovery registry and an execution-layer rule that redirects outgoing ETH from designated addresses to a per-incident recovery escrow. Designations are created only through an authorization process that is separate from block consensus, and every designation is bounded by a verified outstanding stolen amount, has an expiry, and can be challenged and reversed. Accounts that are not designated, gas accounting and validator compensation are unchanged.

## Motivation

Once an exploit succeeds and assets are moved to attacker-controlled addresses, the protocol treats the attacker as fully authorized to spend them. Recovery today depends on centralized intermediaries, asset issuers, law enforcement or voluntary cooperation from the attacker. Even when a theft is conclusively proven, Ethereum has no general mechanism to respond.

Previous recovery proposals (EIP-999, EIP-867) relied on irregular, case-by-case state changes and were broadly rejected. This EIP explores a different design: fixed, deterministic rules that redirect only outgoing value from designated accounts, are capped at the verified stolen amount, and include built-in reversal.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

> **Note:** Sections marked **Open** are unresolved and are the subject of ongoing discussion. This draft is not implementable until they are resolved.

### Constants

| Name | Value |
|---|---|
| `FORK_TIMESTAMP` | TBD |
| `RECOVERY_REGISTRY_ADDRESS` | TBD (system contract) |
| `MIN_CHALLENGE_PERIOD` | TBD |
| `MAX_ORDER_LIFETIME` | TBD |

### Recovery orders

A recovery order is an entry in the recovery registry with the following fields:

| Field | Type | Description |
|---|---|---|
| `incident_id` | `bytes32` | Unique identifier of the theft incident |
| `addresses` | `address[]` | Designated addresses subject to the order |
| `asset` | enum | `NATIVE_ETH` in this version. Other assets are out of scope |
| `verified_amount` | `uint256` | Verified stolen amount, in wei |
| `recovered_amount` | `uint256` | Amount recovered so far, in wei |
| `escrow` | `address` | Per-incident recovery escrow |
| `evidence_hash` | `bytes32` | Commitment to the published evidence bundle |
| `attestations` | `bytes` | Authorization attestations (see Authorization) |
| `activation_time` | `uint64` | Earliest time the order takes effect |
| `expiry_time` | `uint64` | Time after which the order has no effect |
| `status` | enum | `PENDING`, `ACTIVE`, `SATISFIED`, `EXPIRED`, `REVOKED` |

The outstanding amount of an order is `verified_amount - recovered_amount`.

### Recovery registry

The registry is a system contract at `RECOVERY_REGISTRY_ADDRESS`. It MUST:

1. Accept a new order only if its `attestations` satisfy the authorization rules.
2. Set `status` to `PENDING` on creation and to `ACTIVE` only once `activation_time` is reached and no successful challenge has been recorded.
3. Reject any order where `activation_time - creation_time < MIN_CHALLENGE_PERIOD` or `expiry_time - activation_time > MAX_ORDER_LIFETIME`.
4. Allow an order to be moved to `REVOKED` through the challenge process.
5. Emit a log for every state change.

Execution clients read active orders from registry storage at the start of each block.

### Authorization (Open)

Authorization is separate from block consensus. Validators MUST NOT judge whether an address is involved in theft. They only verify that a registry update carries valid attestations.

The following are unresolved:

- The composition of the authorizing set (independent security organizations, onchain governance or a hybrid).
- The quorum and supermajority thresholds.
- How members are selected, rotated and removed.
- The attestation signature scheme.

### Execution rule

From `FORK_TIMESTAMP`, whenever value `v > 0` is transferred out of an account `A` that is listed in one or more `ACTIVE` orders, by any of the following paths:

- the top-level transaction value,
- a `CALL` or `CALLCODE` with non-zero value,
- a `SELFDESTRUCT` beneficiary transfer,
- any other path that debits `A`'s balance in favor of another account, apart from gas payment,

the client MUST:

1. Compute `r = min(v, recoverable(A))`, where `recoverable(A)` is defined under Attribution.
2. Credit `r` to the `escrow` of the applicable order instead of the recipient.
3. Credit `v - r` to the recipient as normal.
4. Increase that order's `recovered_amount` by `r`, and set `status` to `SATISFIED` if the outstanding amount reaches zero.
5. Emit a recovery log containing `incident_id`, `A`, the intended recipient and `r`.

If `A` appears in several active orders, they MUST be applied in a deterministic order (**Open**: proposed ordering is by `activation_time`, then `incident_id`).

Gas purchase, refunds, base fee burn and priority fees MUST be computed exactly as without this EIP. The redirected amount MUST NOT be paid to the block proposer.

A block that does not apply this rule correctly is invalid.

### Attribution (Open)

ETH is fungible, so `recoverable(A)` needs an attribution model. Candidate models include an incident-specific encumbrance ledger with first-in-first-out, last-in-first-out or pro-rata semantics. The model MUST guarantee that the total amount recovered for an order never exceeds `verified_amount`, and SHOULD minimize impact on legitimate funds received by `A` after designation.

Treatment of shared contracts (exchanges, bridges, DEX pools, mixers) is unresolved. This draft proposes that such contracts MUST NOT be designated.

### Recovery escrow

Each order's escrow MUST:

1. Accept only credits made by the execution rule for its `incident_id`, plus refunds from the challenge process.
2. Release funds only through a published claims process. The claims process is out of scope for this EIP.
3. Expose its full accounting onchain.

### Challenge and reversal (Open)

Any party MAY challenge an order during its challenge period or while it is `ACTIVE`. A successful challenge MUST move the order to `REVOKED` and MUST make any amount recovered from a wrongly designated address refundable to that address from the escrow. The challenge process, its adjudicators and its timelines are unresolved.

## Rationale

**Separation of authorization and consensus.** Attestations and finality establish agreement on state, not facts about theft. Keeping the judgment outside consensus and the enforcement inside consensus keeps validators neutral and makes enforcement uniform.

**Consensus enforcement rather than validator policy.** If recovery were optional, a single non-participating proposer could include transfers that others reject, causing inconsistent state. The rule therefore has to be part of the state transition function.

**Separate accounting from gas.** Mixing recovery with fee accounting would complicate EIP-1559 mechanics and create proposer incentives. Gas stays gas, and recovery stays restitution.

**Bounded, expiring orders.** Capping recovery at the verified amount and expiring orders prevents any address from carrying an open-ended confiscation rule.

**Native ETH only.** ERC-20 and NFT balances live in contract storage. Recovering them is better handled by a separate application-level standard.

## Backwards Compatibility

This EIP changes the state transition function and requires a hard fork. Contracts that assume a value transfer always credits the full amount to the recipient may behave differently when the sender is designated. The extent of this risk is an open question.

## Security Considerations

- **Timing.** Attackers usually move funds within minutes, while authorization with a challenge period takes much longer. The mechanism may rarely reach funds before they are laundered or bridged.
- **Capture of the authorizing set.** A compromised or coerced authorizing set could designate innocent addresses. Distribution, transparency, challenge and reversal are the main mitigations.
- **Legal and political pressure.** A working recovery registry could attract demands to use it beyond theft, for example sanctions enforcement. The authorization rules need to resist scope creep.
- **Evasion.** Funds can be moved through contracts, smart wallets, EIP-7702 delegations and shared pools. Incomplete coverage of value-transfer paths would make the mechanism trivially avoidable.
- **Denial of service.** Registry lookups and attribution bookkeeping MUST have bounded cost per value transfer.
- **Contract assumptions.** Partial crediting of value may break invariants in contracts that receive funds from designated accounts.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
