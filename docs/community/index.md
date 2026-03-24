## About DEEPNET

**DEEPNET** is a browser-based hacking simulation that runs entirely in your web browser—no installation required. Players interact with the game through a command-line interface, simulating a hands-on hacking experience driven by text-based commands.

---

## Core Gameplay Loop

At its core, DEEPNET follows a structured but flexible loop:

**Scan → Hack → Breach → Exfiltrate → Sell**

Players begin by scanning subnets to identify potential targets, each with varying services and security levels. Choosing the correct exploit—such as SSH, FTP, HTTP, or MySQL—is crucial: a well-matched tool minimizes risk, while poor choices increase the chance of detection.

Once access is gained, players enter the **Breach Engine**, where they explore a procedurally generated filesystem. Here, they must identify valuable data while managing their trace level. Every action generates noise, and reaching 100% trace results in an immediate disconnect and loss of all acquired data.

Successful operations allow players to extract data, sell it for profit, upgrade their systems, and develop more advanced tools—feeding back into the gameplay loop.

---

## A Reactive World

DEEPNET features a dynamic, interconnected world that responds to player actions:

- **Heat System:** Frequent or reckless hacking increases visibility, making targets harder and reducing rewards. Over time, heat can decay if activity slows.
- **Target Lifecycle:** Systems can become unstable, go offline, or be permanently removed, with new targets emerging to replace them.
- **NPC Interactions:** The game includes 21 AI-driven NPCs who track player behavior and adjust their responses accordingly.
- **Factions:** Players must align with either *COVENANT* or *MERIDIAN*, a permanent choice that influences difficulty, trust relationships, and exposure.
- **Geopolitical States:** The game world cycles through global states (e.g., stability, crisis, détente), which players can influence—particularly by targeting government systems.
- **Shadow Profile:** Over time, the network builds a persistent profile of the player, eventually assigning a unique codename.

---

## Tools and Crafting

Tools in DEEPNET are defined by multiple attributes, including **Power, Noise, Stability, and Operational Security (OpSec)**. They interact within an 18-service compatibility system, requiring players to strategically match tools to targets.

Players can develop tools using in-game systems such as a compiler and AI-assisted crafting, but all tools degrade over time and must be maintained or replaced.

---

## Multiplayer Features

DEEPNET includes several multiplayer elements:

- **PvP Hacking:** Players can target each other to steal in-game currency.
- **Crews:** Cooperative groups for shared objectives.
- **Contracts and Bounties:** Player-driven missions and timed challenges.
- **New Player Protection:** A “Newblood Shield” provides temporary safety for newcomers.

---

## Comparison to Similar Games

DEEPNET shares similarities with classic hacking simulations but distinguishes itself through its systemic design:

- Compared to *Hacknet*, which is narrative-driven and linear, DEEPNET emphasizes ongoing, emergent gameplay.
- Compared to *Uplink*, DEEPNET expands on the formula with multiplayer systems, faction mechanics, dynamic world states, and deeper post-exploitation gameplay.

---

## Current State

DEEPNET is a solo-developed project that is fully playable and actively evolving. While it offers significant depth, it remains a work in progress and may include rough edges. The game is currently free to play.

---

## Play the Game

👉 <https://deepnet.us/>
