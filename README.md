# Skynet Zero

A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS) designed for Asuswrt-Merlin.

Skynet Zero optimizes the firewall script to eliminate the requirement for a USB swap file. By modifying the kernel's virtual memory allocation (`vm.swappiness=0`), Skynet Zero compiles IP blocklists entirely in physical RAM, preventing write-degradation on USB flash drives while operating at native memory speeds.

## Features & Improvements

* **Zero Swap Architecture:** Forces heavy array compilations strictly into physical RAM without causing `can't fork` lockups. Your USB drive is saved from daily I/O write fatigue.
* **Concurrency & Thread Safety:** Utilizes a dynamic PID-based temporary file system (`$$`) to completely eliminate race conditions. Background Cron jobs and WebUI polling run simultaneously without colliding.
* **AMTM Native Integration:** Natively integrated into Asuswrt-Merlin Terminal Menu (amtm).

## Requirements

A USB drive formatted for Asuswrt-Merlin to hold installation scripts and logs. No active swap file is required (though keeping a dormant swap file mounted is mathematically safe).

## Installation

**Via AMTM (Recommended):**
In your SSH Client, launch **amtm** and select the firewall installation option. 

When prompted, amtm provides the option to install:
1. **Skynet**
2. **Skynet Zero**

Select **Option 2** to install Skynet Zero without generating a USB swap file. 

<img width="503" height="187" alt="zero swap" src="https://github.com/user-attachments/assets/684cf1a7-331a-4e7b-89ba-181f1d0a0add" />

Follow the remaining prompts to configure your preferences.

**Via Manual Install:**
Alternatively, run the following command to install directly from the repository:
```Shell
/usr/sbin/curl -s "https://raw.githubusercontent.com/underd0se/Skynet-Zero/master/firewall.sh" -o "/jffs/scripts/firewall" && chmod 755 /jffs/scripts/firewall && sh /jffs/scripts/firewall install
```

## Switching Swap Modes

You can seamlessly toggle between the optimized Zero Swap mode and the traditional USB Swap fallback at any time without reinstalling.

<img width="555" height="793" alt="switching" src="https://github.com/user-attachments/assets/98c7d913-c60c-4303-a65a-8247188fb66a" />

1. Open the Skynet menu by typing `firewall` in SSH.
2. Navigate to **Settings** (Option 11) -> **Switch Swap Mode** (Option 18).
3. The engine will guide you through cleanly transitioning your kernel hooks and USB configuration.

## Stress Testing & Stability Validation

Skynet Zero flawlessly executed exhaustive compilations (IP blocks, malware crunching, and dynamic stat generation) under extreme edge-case stress:

| Constraint | Methodology | Result |
| :--- | :--- | :--- |
| **CPU Saturation** | 4 infinite subshells forcefully pinning the CPU to 100% | Passed |
| **Page Cache Exhaustion** | A concurrent 2GB continuous binary disk flush | Passed |
| **Inode Starvation** | Rapid concurrent generation of 50,000 dummy files on the USB | Passed |
| **Process Collisions** | Simultaneous execution of `banmalware`, `aiprotect`, and WebUI log parsing | Passed |

**System Outcome:** Zero USB write degradation (0 Bytes swapped), zero lockups, and perfect RAM flushing upon completion.

## Uninstallation

Run `sh /jffs/scripts/firewall uninstall` to purge all IPSet arrays, scripts, logs, and safely restore the router's original `swappiness` kernel settings.
