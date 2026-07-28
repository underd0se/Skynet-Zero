# Skynet Zero (Zero Swap)

A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS).

Skynet Zero is an optimized version of the Asuswrt-Merlin firewall script designed to eliminate the requirement for a USB swap file. By modifying how the Linux kernel handles virtual memory allocation, Skynet Zero safely compiles IP blocklists entirely in physical RAM. This prevents read/write degradation on attached USB flash drives while executing at native memory speeds.

## Technical Architecture

The original Skynet script requires a 2GB USB swap file. Because Asuswrt routers have limited RAM (e.g., 512MB), heavy operations like parsing massive IP sets or compiling malware blocklists frequently cause the Linux kernel to heavily utilize this swap file. While this prevents the router from crashing, it results in hundreds of megabytes of daily I/O writes to the USB flash drive, eventually destroying the drive through write-fatigue.

Skynet Zero solves this hardware degradation through a dynamic kernel optimization. When Skynet Zero mode is selected during installation, it alters the kernel's anonymous memory paging preference (`vm.swappiness=0`). 

By explicitly instructing the Linux kernel to prioritize dropping the page cache over swapping anonymous memory, Skynet Zero forces the router to execute all heavy array compilations strictly in physical RAM. The USB swap file remains mounted to satisfy the kernel's virtual memory math (preventing `can't fork` allocation lockups), but it is functionally dormant. 

## Memory Implications

Because Skynet Zero operates at native physical RAM speeds without relying on USB paging, users will notice higher overall physical memory utilization in the Asuswrt WebUI. This is completely normal and mathematically safe. The router seamlessly balances physical memory demands in the background, offering significantly faster execution times for firewall updates while achieving zero USB write degradation.

> **Zero-Swap Survival Note:** While keeping a dormant swap file mounted is recommended as a virtual `CommitLimit` fail-safe, telemetry proves that the `swappiness=0` architecture is so efficient at forcing the kernel to purge disk caches that Asuswrt routers can natively survive extreme firewall compilations even if the user has **0 bytes** of swap mounted.

## Stress Testing & Stability Validation

To validate the stability of the `swappiness=0` architecture, Skynet Zero was subjected to an extreme Edge Case Simulator on native Asuswrt hardware. The script flawlessly executed an exhaustive compilation (IP blocks, malware crunching, and dynamic stat generation) while the router was actively suppressed under four simultaneous catastrophic constraints:

| Constraint | Methodology | Result |
| :--- | :--- | :--- |
| **CPU Saturation** | 4 infinite subshells forcefully pinning the CPU to 100% (Load Avg: 7.71). | Passed |
| **Page Cache Exhaustion** | A concurrent 2GB continuous binary disk flush (`/dev/zero` to USB). | Passed |
| **Inode Starvation** | Rapid concurrent generation of 50,000 dummy files on the USB to saturate file tables. | Passed |
| **Process Collisions** | Simultaneous execution of `banmalware`, `aiprotect`, and WebUI log parsing. | Passed |

**System Outcome:** 
* **Zero USB Degradation:** Telemetry mathematically confirmed `0 Bytes` of the swap file were utilized during the entire crucible.
* **Zero Lockups:** The latent `overcommit=2` padding completely prevented any `can't fork` panics despite massive memory saturation.
* **Perfect Recovery:** All volatile RAM was instantly flushed upon test completion with 0 stuck processes.

## Requirements

A USB drive formatted for Asuswrt-Merlin is required to hold the installation scripts and logs. No swap file is required.

## Installation Procedure

In your SSH Client, run the following command:

```Shell
/usr/sbin/curl -s "https://raw.githubusercontent.com/underd0se/Skynet-Zero/master/firewall.sh" -o "/jffs/scripts/firewall" && chmod 755 /jffs/scripts/firewall && sh /jffs/scripts/firewall install
```
During the interactive installation wizard:
1. When prompted to **Select SWAP File Size**, choose **Option 3: 0GB (Zero Swap - Skynet Zero)** to install the firewall without generating a swap file.

   <img width="503" height="187" alt="zero swap" src="https://github.com/user-attachments/assets/684cf1a7-331a-4e7b-89ba-181f1d0a0add" />

3. Follow the remaining prompts to configure your logging and filtering preferences.

**Changes Made During Installation:**
1. Downloads the execution script to `/jffs/scripts/firewall`.
2. Creates the data directory (`skynetloc`) on your selected USB partition (or `/jffs` if explicitly selected).
3. Injects execution triggers into `/jffs/scripts/firewall-start`, `/jffs/scripts/services-stop`, and `/jffs/scripts/service-event`.
4. Injects configuration settings into `/jffs/configs/profile.add` and `/jffs/configs/dnsmasq.conf.add`.
5. Backs up your router's default `swappiness` and `overcommit_memory` settings to NVRAM.
6. Dynamically enforces `swappiness=0` (if Skynet Zero mode is selected) and `overcommit_memory=0`, and injects persistence commands into `/jffs/scripts/firewall-start`.

## Switching Swap Modes

Skynet Zero provides a seamless UI toggle to switch between traditional USB Swap mode and the optimized Zero Swap mode at any time, without needing to reinstall.

<img width="555" height="793" alt="switching" src="https://github.com/user-attachments/assets/98c7d913-c60c-4303-a65a-8247188fb66a" />

1. Open the Skynet interactive menu by typing `firewall` in your SSH client.
2. Navigate to **Settings** (Option 11).
3. Select **Switch Swap Mode** (Option 18).
4. **If switching TO Zero Swap:** The engine will prompt you to cleanly delete your existing USB swap file, revert your kernel logic, and then activate the Skynet Zero configuration.
5. **If switching TO Traditional Swap:** The engine will cleanly wipe the Skynet Zero kernel hooks and walk you through generating a new 2GB swap file on your USB drive.

## Uninstallation Procedure

To uninstall Skynet Zero, run the following command from your SSH client or select the Uninstall option from the interactive menu:

```Shell
sh /jffs/scripts/firewall uninstall
```

**Changes Made During Uninstallation:**
1. Purges all IPSet arrays, cron jobs, and custom iptables rules from active memory.
2. Deletes the entire SkyNet data directory from your USB drive.
3. Scrubs the `# Skynet` execution triggers from all `/jffs/scripts/` and `/jffs/configs/` files.
4. Restores the router's original `swappiness` and `overcommit_memory` kernel settings using the backups saved in NVRAM.
5. Scrubs the injected `# Skynet Zero` overrides from your boot scripts and performs legacy bypass cleanup.
6. Restarts the firewall and dnsmasq services to return the router to a pristine baseline state.
