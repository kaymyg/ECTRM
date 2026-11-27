# Help wanted

ECTRM: Consensus-Based Theft Recovery Mechanism is open to contributions. Below is the hardening work that would make it stronger, roughly in order of impact. Pick anything, comment on (or open) an issue so others know you are on it, and see [CONTRIBUTING.md](CONTRIBUTING.md) to get started.

Found a vulnerability instead? Please report it privately, see [SECURITY.md](SECURITY.md).


### Attack the griefing model

**High impact.** How cheaply can someone file false theft claims to freeze honest funds? Write up concrete attacks and the bonds or limits that stop them.

### Validator collusion economics

**High impact.** Model what it would cost a colluding set of validators to approve a fraudulent recovery, and compare it to the value at risk.

### Worked historical cases

**Medium.** Walk real exploits (bridge hacks, key compromises) through the proposal and show where it would and would not have helped.

### Precise state transition spec

**Medium.** Turn the prose rules into a precise specification that a client team could implement and test against.
