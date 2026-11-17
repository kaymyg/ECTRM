# Consensus-Based Theft Recovery Mechanism (ECTRM): a bounded, rule-based approach to recovering stolen ETH

**Suggested venue:** ethresear.ch (Execution Layer Research) or Ethereum Magicians (Primordial Soup)
**Status:** Early research, Draft EIP opened for technical criticism

---

## TL;DR

Once stolen assets leave a victim's control, Ethereum has no protocol-level way to respond, even when the theft is conclusively proven. This post explores whether a narrowly scoped, consensus-enforced mechanism could redirect proven stolen ETH from designated addresses into a victim escrow, capped at the verified stolen amount, with a public challenge and reversal process, and without changing anything for ordinary users.

I know this touches one of Ethereum's most sensitive topics. I'm not asking anyone to support it yet. I'm asking whether it is technically feasible, and where it breaks.

---

## The problem

Ethereum's security ecosystem (audits, bug bounties, simulation, real-time exploit detection) has become very good at *detecting* theft. It has almost no way to *recover* it.

From the protocol's perspective, an attacker holding stolen keys or exploit proceeds is fully authorized to spend them. Recovery today depends on exchanges, stablecoin issuers, law enforcement or the attacker's goodwill. The result is an asymmetry: an exploit can be confirmed beyond reasonable doubt while the funds keep moving freely through intermediary wallets, mixers and bridges.

## The idea in brief

1. **Recovery registry.** A consensus-recognized registry records *recovery orders*. Each order covers an incident, the designated addresses, the asset, a verified stolen amount, an escrow destination and an expiry.
2. **Authorization is separate from block consensus.** Orders are approved through a threshold of independent reviewers, with a quorum, a supermajority, a public challenge period and a timelock. Validators don't judge guilt. They only verify that a registry update is validly authorized and then apply deterministic rules.
3. **Recovery path at execution.** When a designated address sends value, the protocol redirects the eligible amount to the incident's escrow instead of the recipient, until the order's outstanding balance reaches zero. Gas is charged and paid exactly as today.
4. **Bounded by design.** An order can never recover more than its verified outstanding amount, and it expires when that amount is met or its time limit passes.
5. **Escrow and claims are separate.** Recovered funds sit in a per-incident escrow and are released only through a transparent claims process, which can run on its own timeline.
6. **Reversible.** Wrong designations can be challenged, corrected and refunded.
7. **Ordinary users are unaffected.** No universal screening, no permission to transact, no change to fees or validator rewards for unmarked accounts.

Because this changes the state transition function, it would require a hard fork. Optional validator policy cannot enforce it consistently.

## How this differs from EIP-867 and EIP-999

Recovery has been proposed before, and those proposals were broadly rejected. I've tried to take the objections seriously:

| | EIP-999 / EIP-867 style | ECTRM |
|---|---|---|
| Mechanism | Irregular state change per incident, decided case by case | Fixed rules, applied deterministically by all clients |
| Scope | Rewrite balances or restore code | Redirects only outgoing value from designated addresses |
| Upper bound | Whatever the fork specifies | Capped at verified outstanding stolen amount |
| Reversibility | A new fork | Built-in challenge, correction and refund |
| Effect on others | Depends on the case | None for unmarked accounts |

The objection I expect to remain is that *any* registry able to redirect a designated account's funds is protocol-level censorship, whatever its safeguards. I think that is the right question to argue about, and I'd rather have it argued in public than avoided.

## Known hard problems (where I need the most help)

**1. The timing gap.** Attackers typically launder funds within minutes. A careful authorization process (review, challenge period, timelock) takes days. A slow process may never catch funds that matter, and a fast one weakens the safeguards. Is there a useful middle ground, for example a short provisional hold with mandatory review, or is this gap fatal?

**2. Attribution of fungible ETH.** ETH has no per-unit history. If a designated address holds stolen and legitimate funds, or receives legitimate deposits later, what rule decides which outgoing ETH is "stolen"? Options include an incident-specific encumbrance ledger with first-in-first-out, last-in-first-out or pro-rata semantics. All have edge cases.

**3. Who authorizes.** Security firm threshold, onchain governance or a hybrid? Every option creates a target for capture, coercion or legal pressure. How is that authorization set chosen, rotated and held accountable?

**4. Execution paths.** Checking `tx.origin` or `msg.sender` is not enough. Designated accounts can move value through `CALL` with value, `SELFDESTRUCT` beneficiaries, smart wallets, EIP-7702 delegation and contract-mediated transfers. Can all value-transfer paths be covered without breaking EVM semantics or contract assumptions?

**5. Intermediaries.** What happens when stolen ETH reaches an exchange hot wallet, a bridge, a DEX pool or a mixer, where it is commingled with many innocent users' funds? I lean toward never designating shared contracts, but that leaves an obvious escape route.

**6. ERC-20s and other assets.** Token balances live in contract storage, not protocol state. Recovery there probably needs a token-level standard rather than a consensus rule. Is native-ETH-only even worth a fork?

**7. Client complexity and DoS.** The registry check has to be cheap and deterministic, and must not open new denial-of-service vectors.

## What I'm asking for

- Is a native-ETH-only version technically sound as a research prototype?
- Which of the hard problems above do you consider fatal, and which are solvable?
- Are there prior designs (academic or otherwise) I should be building on?
- Would anyone be interested in collaborating on a devnet prototype on a modified execution client?

I've opened a Draft EIP so the structure is concrete enough to critique, with the unresolved parts clearly marked as Open. It is a starting point for discussion, not a finished specification.

*Draft EIP pull request:* link to follow in a reply below.

Thanks for reading, and I welcome blunt criticism.
