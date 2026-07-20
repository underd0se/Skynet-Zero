# SkyNet-SF (Swap-Free)

A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS).

SkyNet-SF is an optimized version of the Asuswrt-Merlin firewall script designed to eliminate the requirement for a USB swap file. By modifying how the Linux kernel handles virtual memory allocation, SkyNet-SF safely compiles IP blocklists entirely in physical RAM. This prevents read/write degradation on attached USB flash drives while executing at native memory speeds.

## Technical Architecture

The original Skynet script requires users to mount a 2GB USB swap file to satisfy the Linux kernel's strict virtual memory reservation system (`vm.overcommit_memory=2`). Without this buffer, background processes crash with `Cannot allocate memory` kernel panics.

SkyNet-SF eliminates this dependency by injecting a dynamic kernel patch (`vm.overcommit_memory=0`) directly into the initialization sequence. This forces the Asuswrt kernel into Heuristic Mode, allowing it to dynamically assess the physical RAM available rather than relying on strict virtual math. 

## Disclaimer: Memory Implications and Asus Security Daemon (asd)

Running a zero-storage architecture on embedded devices with limited physical RAM (e.g., 512MB) carries specific side effects. Without a swap file to page idle memory, physical RAM is consistently maintained near maximum capacity.

**AiProtection / asd Daemon Conflict**
The Asus Security Daemon (`asd`), which powers TrendMicro's AiProtection, requires significant contiguous memory allocation to parse its encrypted malware signatures. When SkyNet-SF and `asd` compete for the same non-swappable physical RAM, `asd` will encounter allocation failures. 

This results in the `asd` daemon entering a soft crash-loop, causing sustained CPU spikes (averaging 30% to 100% utilization on a single core) as it aggressively retries its memory allocation. 

**Recommendation:** If you utilize Asus AiProtection or rely on the `asd` daemon for security assessments, it is highly recommended to use the original Skynet script with a standard 2GB swap file. SkyNet-SF is designed for users who keep AiProtection disabled or prefer to manage their memory constraints manually.

## Stress Testing & Stability Validation

To validate the stability of the `overcommit_memory=0` patch without a swap file, SkyNet-SF was subjected to validation testing on Asuswrt hardware. The script executed its array compilations while the router was actively suppressed under four simultaneous constraints.

| Constraint | Methodology | Result |
| :--- | :--- | :--- |
| RAM Starvation | 100MB of physical RAM artificially locked within the `tmpfs`. | Passed |
| CPU Saturation | All router cores forcefully pinned to 100% utilization. | Passed |
| Page Cache Exhaustion | A concurrent 2GB Samba disk flush combined with the rapid generation of 50,000 fragmented files. | Passed |
| Pipeline Collisions | Diversion's 7-stage awk/grep blocklist pipeline executed concurrently during IPSet rebuilds. | Passed |

**System Outcome:** Zero `Cannot allocate memory` panics. The kernel successfully distributed the physical RAM natively between all processes.

## Requirements

A USB drive formatted for Asuswrt-Merlin is required to hold the installation scripts and logs. No swap file is required.

## Installation Procedure

In your SSH Client, run the following command:

```Shell
/usr/sbin/curl -s "https://raw.githubusercontent.com/underd0se/SkyNet-SF/master/firewall.sh" -o "/jffs/scripts/firewall" && chmod 755 /jffs/scripts/firewall && sh /jffs/scripts/firewall install
```

**Changes Made During Installation:**
1. Downloads the execution script to `/jffs/scripts/firewall`.
2. Creates the data directory (`skynetloc`) on your selected USB partition.
3. Injects execution triggers into `/jffs/scripts/firewall-start`, `/jffs/scripts/services-stop`, and `/jffs/scripts/service-event`.
4. Injects configuration settings into `/jffs/configs/profile.add` and `/jffs/configs/dnsmasq.conf.add`.
5. Writes a dummy `# swapon bypassed for SkyNet-SF` comment into `/jffs/scripts/post-mount` to satisfy legacy bypass checks.
6. Backs up your router's default `overcommit_memory` setting to NVRAM and safely enforces `0` dynamically during operation.

## Uninstallation Procedure

To uninstall SkyNet-SF, run the following command from your SSH client or select the Uninstall option from the interactive menu:

```Shell
sh /jffs/scripts/firewall uninstall
```

**Changes Made During Uninstallation:**
1. Purges all IPSet arrays, cron jobs, and custom iptables rules from active memory.
2. Deletes the entire SkyNet data directory from your USB drive.
3. Scrubs the `# Skynet` execution triggers from all `/jffs/scripts/` and `/jffs/configs/` files.
4. Scrubs the dummy `# swapon bypassed for SkyNet-SF` comment from `/jffs/scripts/post-mount`.
5. Restores the router's original `overcommit_memory` kernel setting using the backup saved in NVRAM.
6. Restarts the firewall and dnsmasq services to return the router to a pristine baseline state.

## Usage and Commands

To open the interactive menu:

```Shell
firewall
```

### Example Commands

**Unban:**
* `firewall unban ip 8.8.8.8`: Unban the specified IP.
* `firewall unban range 8.8.8.8/24`: Unban the specified CIDR block.
* `firewall unban domain google.com`: Unban the specified URL.

**Ban:**
* `firewall ban ip 8.8.8.8 "Comment"`: Ban the specified IP with a comment.
* `firewall ban range 8.8.8.8/24 "Comment"`: Ban the specified CIDR block with a comment.
* `firewall ban domain google.com`: Ban the specified URL.

**Settings:**
* `firewall settings autoupdate enable|disable`: Enable/disable Skynet autoupdating.
* `firewall settings filter all|inbound|outbound`: Select what traffic to filter.
* `firewall settings webui enable|disable`: Enable/disable WebUI.

**Update:**
* `firewall update`: Standard update check.
* `firewall update -f`: Force update even if no changes detected.

**Stats:**
* `firewall stats`: Compile stats with default top 10 output.
* `firewall stats search ip 8.8.8.8`: Search logs for entries on 8.8.8.8.
