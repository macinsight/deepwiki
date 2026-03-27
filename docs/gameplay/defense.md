# Defense Systems

Your node can be targeted by other operators. DeepNet provides several defensive tools to detect, deter, and counter intrusions.

## Tripwires

Tripwires are detection sensors placed on nodes you control.

### Commands

```
tripwire list                              — list your active tripwires
tripwire set <ip> [trigger] [action]       — plant a tripwire
tripwire remove <id>                       — disarm a tripwire
```

### Triggers

| Trigger | Description |
|---------|-------------|
| `file_access` | (default) Fires when an intruder accesses files |
| `login_fail` | Fires on failed intrusion attempts before breach |
| `port_scan` | Fires when someone scans the node |

### Actions

| Action | Description |
|--------|-------------|
| `alert` | (default) Notifies you of the intrusion |
| `counter_exposure` | Increases the attacker's exposure |
| `ddos_pulse` | Disrupts the attacker's connection |

### Honeypot Combo

Add `--honeypot` to arm bait files on the same node:

```
tripwire set 178.62.4.35 file_access counter_exposure --honeypot
```

Honeypot files look like real data. If an attacker exfiltrates them, the dealer integrity check refuses the sale and silently notifies the owner. Combined with `counter_exposure`, this creates a trap that both damages the attacker and provides intel.

### Limits

- Maximum 5 active tripwires at a time
- Triggered tripwires become disarmed and must be re-set

## Firewall

The firewall system provides passive defense through deployed appliances. See the [DeepAI Firewall section](../deepai.md#firewall) for compilation details.

Deployable appliance types include:

- **ICE Node** — Intrusion Countermeasures Electronics
- **Relay Filter** — Filters suspicious traffic
- **Telemetry Sink** — Absorbs and logs probe attempts

## Evidence Locker

When the Agency seizes your tools during a raid, they're held in the evidence locker.

```
evidence-locker status     — check locker contents
evidence-locker recover <tool_id>  — recover early (costs RCH)
```

Seized tools auto-release after **72 hours** if not recovered manually.
