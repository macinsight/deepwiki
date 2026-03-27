# Economy & Progression

## RCH (Reputation Credits)

RCH is The Net's shadow currency. It's earned by selling exfiltrated data or mining compute cycles, and spent on hardware, tools, and services.

### Earning RCH

| Source | Description |
|--------|-------------|
| **Selling data** | Sell exfiltrated files to the dealer with `sell <filename>` |
| **Mining** | Passive income via `deepmine start` (~13 RCH/min at baseline) |
| **Contracts** | Fulfill data contracts for premium payouts |
| **Bounties** | Complete bounty objectives (requires reputation) |
| **Jobs** | Quick-window objectives from the job board (`jobs -list`) |

### Selling Files

Sell prices are affected by:

- **Heat level** — COLD gives the best prices. Selling at HOT or BURNING incurs a significant penalty.
- **Dealer trust** — Higher trust = better prices. Build trust through consistent interaction.
- **World state** — CRISIS provides a loot yield bonus (+12%) and a loot multiplier (x1.20)
- **File quality** — Files have quality tiers (grey, green, blue, etc.) affecting base value

!!! tip "Maximize Sell Value"
    Wait until your heat is COLD (0-49) before selling files. During CRISIS, the loot bonuses stack with COLD prices for maximum profit.

### File Types

Not all files are sellable. Know what to keep and what to sell:

| Extension | Type | Action |
|-----------|------|--------|
| `.ndf` | Training fragments | **KEEP** — pathway progression |
| `.ds` | DeepScript files | **KEEP** — tool compilation |
| `.ai` | AI implant files | **KEEP** — hack tools |
| `.rdat` | Intel Shards | **KEEP** — forge research data |
| `.df` | FORGE-MAT | **KEEP** — firewall compilation |
| `.log`, `.txt` | Log files | **SKIP** — worthless (0 RCH) |
| `.pdf`, `.json`, `.db`, `.csv`, `.tar.gz`, `.sql.gz`, `.zip`, `.kdbx`, `.pem`, `.key` | Data files | **SELL** — valuable |

### Contracts

Contracts are data requests from NPCs offering premium payouts above dealer rates.

- View contracts: `contracts list`
- Fulfill: `contracts fulfill <contract-id> <file-id>`
- Post your own: `contracts post`
- Files must match the contract category (e.g., finance files for a finance contract)

### Bounties

Bounties are objectives available from the bounty board. Access requires building trust with specific NPCs (e.g., Kira). During CRISIS, higher-level bounties become available with larger RCH pools.

- View: `bounties list`
- Claim: `bounties claim`

## Hardware

Your rig affects mining speed, hack capability, and system load. Components include:

| Component | Function |
|-----------|----------|
| **CPU** | Processing power for hacking |
| **GPU** | Compute for mining and AI operations |
| **RAM** | Memory for concurrent operations |
| **HDD** | Storage for files and tools |
| **Mainboard** | Base platform affecting all components |
| **Cooling** | Reduces power draw (negative wattage) |
| **PSU** | Power supply — must support total load |

Check your hardware with `hw` or `rig`. Upgrade components through the `broker` or `deepshop` commands.

!!! warning "Power Grid"
    If your system load exceeds PSU capacity, hacks will fail with "power grid unstable." Monitor load % and upgrade your PSU if needed.

## Groups

At grade 10+, you can join one of three hacker collectives:

| Group | Focus | Perks |
|-------|-------|-------|
| **SIGNAL** | Intelligence | Intel gathering, NPC contacts |
| **BLACKOUT** | Heat management | Cooling perks, stealth operations |
| **ARCHIVE** | Economics | Trading, market intel, RCH generation |

## Crews

Crews are player-formed groups with shared resources:

- Create or join a crew with the `crew` commands
- Shared RCH pool (deposit/withdraw)
- Weekly crew challenges for bonus rewards
- Crew reputation and rankings
