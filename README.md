# Skynet Zero (Development Branch)

A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS) focused on modernization, POSIX compliance, and extreme execution speed.

Skynet Zero is an optimized version of the Asuswrt-Merlin firewall script designed to safely compile massive IP blocklists without destroying your attached USB flash drive through write-fatigue.

## Development Changelog

### v1.1.0-dev
- **2026-07-27**: Implemented full Zero-Storage Direct Streaming Pipeline (fixed stats to extract from kernel RAM, bypassed /tmp writes for custom list imports).

### v1.0.4-dev
- **2026-07-27**: Merged `feature/awk-optimization` branch (single-pass awk arrays yielding ~70% speedup).

### v1.0.3-dev
- **2026-07-27**: Added safe purge of legacy unmount injections for Zero Swap users, protecting third-party swaps.
- **2026-07-27**: Updated ASCII UI header to feature minimalist `ZER0` branding.
- **2026-07-21**: Enforced strict USB-only installation to protect internal JFFS flash.
- **2026-07-21**: Added seamless JFFS auto-migration to USB with degraded-mode UI warnings.

### v1.0.2-dev
- **2026-07-18**: Added `--development` and `--master` CLI flags for dynamic branch swapping.
- **2026-07-18**: Finalized native AMTM installation and update support.
- **2026-07-18**: Fixed a branch updater bug caused by stale cache variable evaluation.

### v8.1.x (Development Preview)
- **2026-07-16**: Introduced Dynamic Branch Auto-Updater to natively isolate fork branches.
- **2026-07-16**: Enhanced `Filter_Version` regex parser for extended fork suffix support.
- **2026-07-16**: Implemented memory-aware Dynamic Parallel Streaming Architecture for `banmalware`.
- **2026-07-16**: Built Zero-Storage Direct Streaming Pipeline natively in RAM via `curl` and `awk`.
- **2026-07-16**: Overhauled kernel management by dynamically scaling `swappiness=0` during heavy loads.
- **2026-07-16**: Hardened codebase for strict POSIX & Shellcheck compliance.

## Memory Implications

Because Skynet Zero operates natively in physical RAM, you will notice higher overall physical memory utilization in the Asuswrt WebUI. This is mathematically safe. The router seamlessly balances memory demands, offering significantly faster execution times while completely eliminating USB write degradation.

## Installation Procedure

You can install the bleeding-edge `development` branch directly to your Asuswrt router via your SSH Client:

```Shell
/usr/sbin/curl -s "https://raw.githubusercontent.com/underd0se/Skynet-Zero/development/firewall.sh" -o "/jffs/scripts/firewall" && chmod 755 /jffs/scripts/firewall && sh /jffs/scripts/firewall install
```

During the interactive installation wizard, when prompted to **Select SWAP File Size**, choose **None (Skynet Zero)** to install the firewall natively without generating a USB swap file.

## Uninstallation Procedure

To cleanly remove Skynet Zero, run the following command from your SSH client:

```Shell
sh /jffs/scripts/firewall uninstall
```

This sequence purges all IPSet arrays, cron jobs, and custom iptables rules from active memory, deletes the Skynet data directory, and gracefully scrubs all injected triggers from your Asuswrt boot and config files.
