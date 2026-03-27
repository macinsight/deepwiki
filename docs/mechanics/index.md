# Mechanics

## Heat

Heat represents your visibility to the Agency — DeepNet's surveillance and enforcement arm. Every hack increases your heat, and high heat makes everything harder.

### Heat Levels

| Level | Range | Effects |
|-------|-------|---------|
| **COLD** | 0–49 | Best dealer prices, lowest risk |
| **WARM** | 50–99 | Slightly reduced sell prices |
| **HOT** | 100–149 | Worse deals, harder targets, increased Agency attention |
| **BURNING** | 150–200 | Worst prices, agency raids, **permanent notoriety increase** |

### Heat Gain and Decay

- Each hack adds heat (amount varies by world state — during CRISIS, target stress adds +3 per hack)
- Heat decays passively at **5 per hour**, regardless of VPN route
- The `wipe` command during breach reduces exposure but not heat directly
- The **CLEANER** pathway (hidden layer) makes `wipe` also remove 25% of current heat
- The **OPSEC** pathway reduces heat gain by x0.85 per reinforcement level

!!! warning "Never Reach BURNING"
    Reaching BURNING heat increases your **notoriety**, which is permanent and never decays. Higher notoriety means worse loot, slower scans, and more Agency attention. Managing heat is the single most important survival skill.

### Agency Raids

At high heat levels, the Agency may conduct a **force_logout** — an involuntary disconnection that kills your session. Officer Vance is the Agency's public-facing operative. Keep your heat low to avoid their attention.

---

## Exposure

Exposure is your per-session trace signature. Unlike heat (which persists across sessions), exposure accumulates during active operations and resets when you cool down.

- Exposure increases during hacks and breaches
- At **100% exposure**: forced disconnect
- **Sanctuary VPN** accelerates exposure decay (but blocks offensive operations)
- Other VPN routes cool exposure at their standard rate

### Managing Exposure

The typical cycle is: hack until exposure is high → switch to Sanctuary → cool down → switch back to an offensive route → hack again. Balancing hack attempts with cooling periods is essential for sustained operations.

---

## Notoriety

Notoriety is a **permanent** stat that increases every time your heat reaches BURNING (150+). It never decays.

Higher notoriety results in:

- Worse loot drops
- Slower scan times
- Increased Agency attention and raids

Check your notoriety with the `traces` command, which shows your threat actor dossier including classification, exposure level, and burn count.

---

## VPN Routes

VPN routes determine your network profile during operations. Each route has different trade-offs.

| Route | Grade Req. | Offense | Effects |
|-------|-----------|---------|---------|
| **Public** | Any | ✅ | Balanced, no modifiers |
| **Shadowmesh** | 10+ | ✅ | Lower trace pressure, slower breach |
| **Overclock** | 20+ | ✅ | Faster breach, louder telemetry trail |
| **Relaygrid** | 25+ | ✅ | Better host visibility, noisier routing |
| **Sanctuary** | Any | ❌ | Protected lane, offensive traffic blocked — cooling only |

Switch routes with `vpn -connect <route>.dn` and check your current route with `vpn -status`.

!!! tip "Route Strategy"
    Use Shadowmesh or Public for hacking, then switch to Sanctuary between hack cycles to cool exposure faster. Sanctuary blocks all offensive operations (hacking, scanning) but accelerates recovery.

---

## Trust

Trust represents your relationship with individual NPCs in the game.

**Levels:** UNKNOWN → NOTICED → KNOWN → TRUSTED → INNER

Higher trust unlocks:

- Better dealer sell prices
- Access to exclusive contracts
- NPC outreach and tips
- Bounty board access (requires trust with specific NPCs)

Build trust through consistent interaction — selling to the same dealer, completing contracts, and maintaining a clean reputation.

---

## Recognition

Recognition is how visible you are on The Net as a whole.

**Levels:** NOBODY → HANDLE → NAME → REPUTATION → LEGEND

Higher recognition means:

- NPC mentions and news articles about you
- Board posts referencing your activity
- Access to higher-tier bounties
- Greater influence in the game world

Recognition increases through successful operations, breach count, and overall activity.
