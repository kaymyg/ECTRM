# ECTRM: Consensus-Based Theft Recovery Mechanism

A research proposal for a bounded, transparent, consensus-enforced way for Ethereum to recover demonstrably stolen ETH, without changing anything for ordinary users.

**Status:** Early research and discussion draft. Not an accepted EIP.

## The idea in one paragraph

Today, once an exploit succeeds, the protocol treats the attacker as fully authorized to spend the stolen funds. ECTRM explores a recovery registry in which independently authorized *recovery orders* designate attacker addresses. Upgraded execution clients then redirect outgoing ETH from those addresses into a per-incident victim escrow, capped at the verified stolen amount, with a public challenge and reversal process. Gas, fees, validator rewards and every unmarked account stay exactly as they are.

## Submission status

This proposal is being prepared for the official [Ethereum Improvement Proposals repository (ethereum/EIPs)](https://github.com/ethereum/EIPs), following the process in [EIP-1](https://eips.ethereum.org/EIPS/eip-1).

| Step | Status |
|---|---|
| Full proposal written | Done |
| Draft in the official EIP template | Done, see [eip-draft_consensus_theft_recovery.md](eip-draft_consensus_theft_recovery.md) |
| Discussion thread on [Ethereum Magicians](https://ethereum-magicians.org/c/eips/eips-core/35) | Pending |
| Pull request to [ethereum/EIPs](https://github.com/ethereum/EIPs/pulls) | Pending, opens once the discussion thread is live |

## What's in this repo

| File | What it is |
|---|---|
| [PROPOSAL.md](PROPOSAL.md) | The full write-up: problem, mechanism, recovery levy, attribution, escrow, safeguards and roadmap |
| [eip-draft_consensus_theft_recovery.md](eip-draft_consensus_theft_recovery.md) | The proposal in the official EIP template, with unresolved parts marked **Open** |
| [ectrm-forum-post.md](ectrm-forum-post.md) | A condensed version for discussion on Ethereum Magicians or ethresear.ch |

## Key design principles

- **Ordinary users are unaffected.** No universal screening, no change to fees or validator rewards.
- **Authorization is separate from consensus.** Validators verify that an order is validly authorized and apply fixed rules. They never judge guilt.
- **Bounded.** An order can never recover more than its verified outstanding amount, and it expires.
- **Reversible.** Wrong designations can be challenged, corrected and refunded.
- **Separate accounting.** Gas stays gas, and recovery stays restitution.

## How this differs from past recovery proposals

EIP-999 and EIP-867 relied on irregular, case-by-case state changes and were broadly rejected by the community. ECTRM uses fixed, deterministic rules applied by all clients, redirects only outgoing value from designated addresses, and builds in challenge and refund. The core objection, that any such registry is a form of protocol-level censorship, still stands and is meant to be debated openly.

## Open problems

1. **Timing.** Attackers move funds in minutes, while careful authorization takes days.
2. **Attribution.** ETH is fungible, so separating stolen from legitimate funds needs a ledger model.
3. **Who authorizes**, and how that set resists capture and legal pressure.
4. **Execution paths**, including contract calls, smart wallets and EIP-7702 delegation.
5. **Intermediaries** such as exchanges, bridges, DEX pools and mixers.
6. **ERC-20s and NFTs**, which likely need a separate token-level standard.
7. **Client cost and denial of service.**

## Feedback

Technical criticism is very welcome. Please open an issue, or join the discussion thread once it's live on Ethereum Magicians.

## License

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

Author: Paul K ([@kaymyg](https://github.com/kaymyg))
