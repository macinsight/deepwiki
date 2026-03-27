# DeepAI

DeepAI is the AI forge system for crafting, upgrading, and managing exploit tools. It becomes available after completing **10 successful hacks**.

## Forge

The forge creates AI implant files (`.ai`) from Intel Shards (`.rdat` files) collected during breaches. Implants dramatically improve hack success rates.

### Forging an Implant

1. **Check available shards:** `deepai -stock all`
2. **Get a recommendation:** `deepai -suggest <service>` (e.g., `deepai -suggest smtp`)
3. **Start a forge job:** `deepai -start <service> injection_profile`
4. **Check job status:** `deepai -jobs`
5. **Collect results:** Jobs auto-collect to your `~/ai/` directory

Each implant targets a specific service (SSH, FTP, SMTP, HTTP, HTTPS, DNS) and has four stats:

| Stat | Description |
|------|-------------|
| **Power** | Breach strength — higher power = lower effective risk |
| **Noise** | Detection footprint — higher noise = more trace per action |
| **Stability** | Reliability during breach |
| **Opsec** | Stealth rating — higher opsec = lower detection chance |

### Forge Economy

- Forging costs RCH (injection_profile ~120 RCH, often free with shards)
- Queue limit: 2 concurrent jobs
- Daily limit: 35 jobs
- Higher-tier AI models produce better implants (when available)

## Tuning and Upgrading

Implants can be **tuned** up to 5 times to specialize their stats:

```
deepai -upgrade <file.ai> focus=<power|stability|noise|opsec> steps=1
```

Each tune improves the focused stat but may reduce others. Tuning also costs a small amount of **durability**.

### Durability

All AI tools have an integrity rating (durability_cur / durability_max). Integrity decreases on:

- Failed hacks
- Tuning operations

At 0 integrity, the tool becomes **inoperable**. Use `deepai -recover <file.ai>` to restore it (costs RCH).

## Augmenting

Augmentation adds special modifiers (affixes) to implants:

```
deepai -augment <file.ai> [mode=auto|add|replace] [slot=1..N]
```

## Firewall

The firewall system uses `.df` (FORGE-MAT) capture files collected during breaches to build defensive appliances.

### Compilation Flow

1. Collect `.df` files from breaches (they appear in your `~/firewall/` directory)
2. Compile: `deepai -firewall-compile <file1.df> <file2.df> output=rulepack appliance=relay_filter`
3. Check job status: `deepai -firewall-jobs`
4. Collect: `deepai -firewall-collect <job_id>`
5. Deploy via the FIREWALL tab in the DeepAI GUI

Firewall appliance types include ICE Nodes, Relay Filters, and Telemetry Sinks.

## Pathways

Pathways are DeepAI's neural skill tree — a permanent progression system with 17 nodes across 4 layers. Each node requires `.ndf` (training data) fragments collected during breaches. Higher-tier targets drop more valuable fragments.

### Layer Structure

| Layer | Nodes | Fragments Each | Requirements |
|-------|-------|---------------|--------------|
| **Input** | 6 nodes | 2 frags | None (always available) |
| **Hidden** | 5 nodes | 4 frags | Requires 2 trained Input nodes |
| **Deep** | 3 nodes | 7 frags | Requires 2 trained Hidden nodes |
| **Output** | 3 nodes | 10 frags | Requires 2 trained Deep nodes |

### Input Layer (Reinforceable x5)

Input pathways can be **reinforced** up to 5 times, stacking their effects:

| Node | Effect (per level) |
|------|-------------------|
| **IDS** | Passive trace rate x0.85 in breach |
| **OPSEC** | Heat gain x0.85 per hack |
| **NET** | Netscan +2 hosts, scan shows IDS profile |
| **EXPLOIT** | Tool match score +0.1 on all services |
| **CRYPTO** | +1 exfil slot per breach |
| **RECON** | Scan duration -20% |

### Hidden Layer

| Node | Effect | Prerequisites |
|------|--------|--------------|
| **PHANTOM** | Breach noise x0.7 on all commands | IDS + OPSEC |
| **CLEANER** | Wipe removes 25% heat + reduces trace 5% | OPSEC + IDS |
| **INFILTRATOR** | Hack duration reduced | EXPLOIT + NET |
| **EXTRACTOR** | Exfil noise halved, ls shows value tags | CRYPTO + EXPLOIT |
| **ANALYST** | Scan shows value estimate, cat shows RCH value | NET + RECON |

### Deep Layer

| Node | Effect | Prerequisites |
|------|--------|--------------|
| **SHADOW** | First hack per 24h = zero heat + zero trace | PHANTOM + CLEANER |
| **PREDATOR** | Backdoor x2 duration, detection risk halved | INFILTRATOR + EXTRACTOR |
| **ORACLE** | Netscan reveals all files, scan shows cycle days | ANALYST + any Hidden |

### Output Layer (Permanent — Choose ONE)

!!! warning "Output Choice is Permanent"
    You can only activate ONE output pathway. Choose carefully — this defines your operator identity. Recalibration costs 10,000 RCH + 48h cooldown.

| Node | Effect | Prerequisites | Playstyle |
|------|--------|--------------|-----------|
| **GHOST** | Zero trace when noise stays LOW, auto-wipe on disconnect | SHADOW + any Deep | Stealth operator |
| **SURGEON** | Exfil slots x2, data values +25%, log files exfilable, splice command | PREDATOR + any Deep | Data extraction specialist |
| **ARCHITECT** | Hacks count 3x for geo-escalation, trust gain x2, contacts post your name | ORACLE + any Deep | Political influence |

### Placing Fragments

Use the Pathways tab in the DeepAI GUI, or the CLI:

```
deepai -place <node_key> <file_id>
```

!!! tip "Pathway Strategy"
    Prioritize CRYPTO (more exfil slots = more files per breach = more RCH) and CLEANER (wipe removes heat = more hacking uptime). These two nodes have the highest immediate impact on progression speed.
