# The World of DeepNet

## Factions

Factions are pledged using the `faction` command. This choice is **permanent** for each player and shapes your entire experience — affecting target difficulty, heat modifiers, and available opportunities.

### COVENANT

COVENANT is government-aligned and corporate-aligned. It works alongside the Agency to maintain institutional order.

**Gameplay effects:**

- Easier access to **corporate, finance, and government** targets (-1 FW)
- Harder access to underground and infrastructure targets
- During CRISIS: COVENANT-aligned nodes become **hardened** (+patch, +ICE)

### MERIDIAN

MERIDIAN is underground-aligned, opposing COVENANT and the Agency. It operates in the shadows of infrastructure and abandoned networks.

**Gameplay effects:**

- Easier access to **infrastructure, abandoned, and social** targets (-1 FW)
- Harder access to institutional and government targets
- During CRISIS: MERIDIAN-aligned nodes become **destabilized** (-patch, -ICE)

## World States

The game world cycles through different states that affect all players:

### CRISIS

An active conflict protocol — a high-value window for aggressive operators.

| Modifier | Effect |
|----------|--------|
| Target FW | +1 patch (hardened) |
| Defense | +15% (elevated) |
| Loot yield | +12% (above baseline) |
| Loot multiplier | x1.20 (World Arc) |
| Target stress | +3 heat per hack |
| Bounties | Level 4 bounties available (crisis only) |

CRISIS is a double-edged sword: targets are harder and generate more heat, but the loot bonuses make successful breaches significantly more profitable.

### Other States

The world cycles between conflict and stability. Check the current state with `sitrep`, which shows duration, trend (escalation or de-escalation), active events, and modifiers.

## The Agency

The Agency is The Net's surveillance and enforcement arm. It monitors operator heat, conducts sweeps, and assigns threat classifications to active operators.

- **Officer Vance** is the Agency's public-facing operative
- High heat triggers Agency raids (`force_logout` events)
- The `traces` command shows your Agency threat dossier

## NPCs

Several NPCs populate The Net and interact with players:

- **Kira** — Runs the bounty board. Requires trust to access. Occasionally drops subnet hints in board posts.
- **The Dealer** — Buys your exfiltrated data. Trust level affects prices.

## The Board

The board (`board`) is a social feed where players and NPCs post messages. It serves as both a communication channel and an intelligence source — NPC posts sometimes contain actionable information like subnet locations or faction intel.

## K-Mystery

A puzzle chain accessible via the `kmystery` command. Solving stages reveals deeper secrets about The Net's history and may unlock unique rewards.
