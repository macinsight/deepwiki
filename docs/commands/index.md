# Terminal Commands Guide

This guide provides a comprehensive list of all available commands within the DeepNet terminal, categorized by their function. Use these tools to navigate the filesystem, manage system resources, perform network operations, and engage in hacking activities.

## Basic Usage
To get more details on any specific command, you can use:
- `man <command>`: View the manual page for a command.
- `help <topic>`: Show help for a specific topic.

---

## Access & Navigation
Commands for connecting to the network and moving through the filesystem.

| Command | Description |
| :--- | :--- |
| `rlogin <ip>` | Connect to a DeepNet access point. |
| `cd [dir]` | Change the current directory. |
| `ls [-laRh1] [path...]` | List directory contents. |
| `pwd` | Show the current directory path. |
| `tree [-a] [-L N] [path]` | Show a visual directory tree. |
| `clear` | Clear the terminal screen. |
| `exit` | Disconnect or logout from the current session. |

## Filesystem Operations
Tools for managing files and directories.

| Command | Description |
| :--- | :--- |
| `cat <file>` | Display file contents. |
| `touch <file>` | Create an empty file. |
| `mkdir [-p] <dir>` | Create a new directory (use `-p` for parents). |
| `cp <src> <dest>` | Copy files or directories. |
| `mv <src> <dest>` | Move or rename files. |
| `rm [-r] <path>` | Remove files (use `-r` for directories). |
| `chmod [+x\|-x] <file>` | Toggle the executable bit on a file. |
| `grep <pattern> <file>` | Search for a specific pattern within a file. |
| `find <path> -name <pat>` | Find files by name. |
| `stat <path>` | Show detailed status information for a file. |
| `du [-h] [path]` | Estimate the size of a path. |
| `head/tail [-n N] <file>` | Print the first or last N lines of a file. |
| `sort <file>` | Sort the lines of a file alphabetically. |
| `uniq <file>` | Remove adjacent duplicate lines. |
| `wc <file>` | Count lines, words, and bytes in a file. |

## System Information
Monitor the status and performance of your local machine.

| Command | Description |
| :--- | :--- |
| `date` | Show the current system date and time. |
| `df` | Show available disk space. |
| `free` | Display memory usage. |
| `ps` | Show currently running processes. |
| `uptime` | Show how long the system has been running. |
| `uname [-a]` | Display system information. |
| `id` | Show current user ID and group information. |
| `hw` | Check hardware status. |
| `history` | View your command history. |

## Network Tools
Commands for interacting with the DeepNet and remote hosts.

| Command | Description |
| :--- | :--- |
| `ip` | Show network interfaces and IP addresses. |
| `ping <host\|ip>` | Test reachability of a remote host. |
| `traceroute <host\|ip>` | Trace the hop path to a target. |
| `nslookup <host\|ip>` | Resolve hostnames or IP addresses. |
| `nc [-z] <host> <port>` | Probe a port and read the service banner. |
| `netstat` | Show active network connections. |
| `curl <url>` | Fetch content from an internal endpoint. |
| `wget [-O <file>] <url>` | Download internal content to a file. |
| `vpn -connect <route>` | Shift traffic through a named relay domain. |
| `who` | Show users currently logged into the system. |

## Operations & Hacking
Specialized tools for intelligence gathering and offensive operations.

| Command | Description |
| :--- | :--- |
| `deepscan [subnet]` | Scan a network for active hosts. |
| `hack [ip\|id]` | Open the exploit interface to hack a target. |
| `intel [scope]` | Surface sparse network hints (intel.sh). |
| `logscan [target-ip]` | Inspect security exposure on a target. |
| `deepinspect <tool>` | Preview exploit match scores against a target. |
| `forensics <target>` | Reconstruct evidence from purged logs. |
| `postmortem` | Analyze the results of a previous hack run. |
| `deepai` | Manage the DeepAI queue and upgrade workflow. |
| `deepscript` | Open the script editor or lint script files. |
| `deepcomp` | Compile source code into BIN artifacts. |
| `codex` | View collected intelligence and lore fragments. |
| `sitrep` | View world state, faction dynamics, and bounties. |
| `targets` | List current hacking targets. |

## Economy & Assets
Manage your RCH currency, hardware, and data sales.

| Command | Description |
| :--- | :--- |
| `wallet` | Check your current RCH balance. |
| `assets` | Manage owned hardware and shipments. |
| `broker` | Open the hardware broker application. |
| `deepshop` | Open the hardware shop. |
| `dealer [file]` | Open the dealer menu or sell a specific file. |
| `deepmine` | Manage the RCH cryptocurrency miner. |
| `bounties` | Access the operator bounty board. |
| `contracts` | Access the data contracts marketplace. |
| `jobs` | View and accept short-term objectives for RCH. |

## Social & Communications
Interact with other operators and the world.

| Command | Description |
| :--- | :--- |
| `board` | Open the DeepNet community board. |
| `news` | Read the latest updates from NET CHRONICLE. |
| `pine` | Open the internal mail client. |
| `crew` | Manage or join operator crews. |
| `hardline` | Access the darknet activity feed. |
| `honeypot` | Set up bait files to detect intruders. |
| `tripwire` | Plant counter-detection tripwires on your host. |

## General & Onboarding
Helpful commands for new operators.

| Command | Description |
| :--- | :--- |
| `tutorial [topic]` | Access the ZERO onboarding guide. |
| `next` | Get a suggestion for your next logical action. |
| `glossary [term]` | Explain a specific game term (e.g., `glossary heat`). |
| `factory-reset` | **WARNING:** Wipe your identity and local state. |

---
*Tip: A typical workflow involves `deepscan` to find targets, `hack` to gain access, and `sell` to monetize your findings.
