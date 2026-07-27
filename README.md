# Skynet Zero (Development Branch)

A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS) focused on modernization, POSIX compliance, and extreme execution speed.

Skynet Zero is an optimized version of the Asuswrt-Merlin firewall script designed to safely compile massive IP blocklists without destroying your attached USB flash drive through write-fatigue.

## Development Changelog

### v1.0.3-dev
- **Strict Hardware Enforcement**: The JFFS installation fallback option has been permanently removed. Skynet Zero now actively prevents users from installing its data directory to the router's internal flash memory, explicitly enforcing a USB-only requirement. This protects users from severe internal flash write-fatigue and prevents the Asus Security Daemon (ASD) from falsely flagging and deleting active IP blocklists.
- **Seamless JFFS Auto-Migration**: Legacy users currently stuck on a JFFS installation will now automatically be migrated to an external USB drive the moment they insert one, preserving all historical data and configurations without downtime. If no USB is present, the script injects a persistent, degraded-mode UI warning banner underneath the main menu advising immediate hardware compliance.

### v1.0.2-dev
- **Branch Switching CLI Flags**: Introduced `--development` and `--master` CLI flags to seamlessly hot-swap between stable and development environments directly from the command line, automatically fetching and overriding the local script execution sequence.
- **Native AMTM Support**: Removed legacy `# amtm NoMD5check` hashbang flag, completely finalizing native AMTM installation support and branch update compatibility.
- **Regex Logic Hardening**: Fixed a critical bug in the dynamic branch auto-updater where the runtime environment falsely detected branches based on stale cache variables rather than evaluating the executing source code header.

### v8.1.x (Development Preview)
- **Dynamic Branch Auto-Updater**: Refactored the core update engine to mathematically isolate codebase branches using a global `skynet_branch` payload. Skynet Zero now flawlessly respects your installed branch natively, permanently preventing cross-pollination and false MD5 upgrade notifications from AMTM.
- **Extended Fork Versioning Support**: Enhanced the underlying `Filter_Version` regex string parser to seamlessly capture and display extended semantic version fork suffixes (e.g., `-sz.1.0.1-dev`) inside the Skynet UI.
- **Dynamic Parallel Streaming Architecture**: The script now actively scans your router's hardware capabilities via `/proc/meminfo`. If your router possesses >500MB RAM or an active swap file, the `banmalware` pipeline spins up concurrent background streams to evaluate and crunch blocklists in parallel. Legacy low-RAM hardware gracefully falls back to a synchronous stream.
- **Zero-Storage Direct Streaming Pipeline**: The `banmalware` engine no longer downloads lists to the USB drive. Feed data is downloaded via `curl`, immediately crunched through an `awk` evaluation array, and streamed directly into an atomic `ipset restore` kernel payload in physical memory.
- **Advanced `awk` Optimization**: Replaced heavily bottlenecked `while read` loops inside `Check_Security` and `Unban_PrivateIP` with lightning-fast single-pass `awk` arrays to eliminate CPU exhaustion.
- **Dynamic Kernel Management**: The legacy UI toggle for "Swap Modes" and the buggy NVRAM kernel backups have been entirely removed. The script now securely evaluates your environment at runtime and dynamically relaxes kernel limits (`swappiness=0`, `overcommit_memory=0`) only when absolutely necessary, silently bypassing out-of-memory kernel panics without cluttering system configurations.
- **POSIX & Shellcheck Compliance**: Codebase stripped of Ash-incompatible array declarations and legacy abstractions, bringing the script closer to true POSIX standards for Asuswrt BusyBox execution.

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
