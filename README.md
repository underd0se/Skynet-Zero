# SkyNet-SF (Swap-Free)

*A hardware-friendly, RAM-only fork of [Adamm00's IPSet_ASUS (Skynet)](https://github.com/Adamm00/IPSet_ASUS).*

SkyNet-SF is a highly optimized version of the popular Asuswrt-Merlin firewall script designed specifically to eliminate the requirement for a massive USB swap file. By modifying how the Linux kernel handles virtual memory allocation, SkyNet-SF safely compiles massive IP blocklists entirely in physical RAM. This completely prevents constant read/write degradation on your attached USB flash drives while executing at native memory speeds.

## Technical Architecture (How It Works)

The original Skynet script required users to mount a 2GB USB swap file to mathematically appease the Linux kernel's strict virtual memory reservation system (`vm.overcommit_memory=2`). Without this massive phantom buffer, heavy background processes (like `dnsmasq` forks) would instantly crash with `Cannot allocate memory` kernel panics—even if the router had plenty of free physical RAM.

SkyNet-SF eliminates this dependency by injecting a dynamic kernel patch (`vm.overcommit_memory=0`) directly into the initialization sequence. This forces the Asuswrt kernel into Heuristic Mode, allowing it to dynamically assess the actual physical RAM available rather than relying on strict virtual math. Combined with flattened `awk` parsing pipelines, SkyNet-SF gracefully juggles memory allocations entirely within RAM.

## Stress Testing & Stability

To conclusively prove the stability of the `overcommit_memory=0` patch without a swap file, SkyNet-SF was subjected to an extreme "Doomsday" validation gauntlet on live Asuswrt hardware. 

The script successfully executed its heaviest array compilations while the router was actively suppressed under four simultaneous constraints:
1. **RAM Starvation**: 100MB of physical RAM artificially locked within the `tmpfs`.
2. **CPU Saturation**: All router cores forcefully pinned to 100% utilization.
3. **Page Cache / Inode Exhaustion**: A concurrent 2GB Samba disk flush combined with the rapid generation of 50,000 fragmented files (simulating the intense memory thrashing of an active Apple Time Machine backup).
4. **Pipeline Collisions**: Diversion's heavy 7-stage `awk/grep` blocklist pipeline executed at the exact same millisecond that Skynet rebuilt its IPSet arrays from zero.

**Result**: Zero `Cannot allocate memory` panics. The kernel efficiently distributed the physical RAM natively between all processes. The pipeline is invincible.

## Requirement

All that's required is a USB drive formatted for Asuswrt-Merlin to hold the installation scripts. **No swap file is required.**

## Installation

In your favorite SSH Client:

```Shell
/usr/sbin/curl -s "https://raw.githubusercontent.com/underd0se/SkyNet-SF/master/firewall.sh" -o "/jffs/scripts/firewall" && chmod 755 /jffs/scripts/firewall && sh /jffs/scripts/firewall install
```

## Usage

Skynet provides both a user interactive menu, and command line interface for those who prefer it.

To open the menu its as simple as;

```Shell
firewall
```

## Skynet Script Commands

### Example Unban Commands:

- `firewall unban ip 8.8.8.8`: Unban the specified IP.
- `firewall unban range 8.8.8.8/24`: Unban the specified CIDR block.
- `firewall unban domain google.com`: Unban the specified URL.
- `firewall unban comment "Apples"`: Unban entries with the comment "Apples".
- `firewall unban country`: Unban entries added by the "Ban Country" feature.
- `firewall unban asn AS123456`: Unban the specified ASN.
- `firewall unban malware`: Unban entries added by the "Ban Malware" feature.
- `firewall unban nomanual`: Unban everything but manual bans.
- `firewall unban all`: Unban all entries from both blacklists.

### Example Ban Commands:

- `firewall ban ip 8.8.8.8 "Apples"`: Ban the specified IP with the comment "Apples".
- `firewall ban range 8.8.8.8/24 "Apples"`: Ban the specified CIDR block with the comment "Apples".
- `firewall ban domain google.com`: Ban the specified URL.
- `firewall ban country "pk cn sa"`: Ban known IPs for the specified countries.
- `firewall ban asn AS123456`: Ban the specified ASN.

### Example Banmalware Commands:

- `firewall banmalware`: Ban IPs from the predefined filter list.
- `firewall banmalware google.com/filter.list`: Use the filter list from the specified URL.
- `firewall banmalware reset`: Reset Skynet back to the default filter URL.
- `firewall banmalware exclude "list1.ipset|list2.ipset"`: Exclude lists matching the names "list1.ipset" and "list2.ipset" from the current filter.
- `firewall banmalware exclude reset`: Reset the exclusion list.

### Example Whitelist Commands:

- `firewall whitelist ip 8.8.8.8 "Apples"`: Whitelist the specified IP with the comment "Apples".
- `firewall whitelist range 8.8.8.8/24 "Apples"`: Whitelist the specified range with the comment "Apples".
- `firewall whitelist domain google.com`: Whitelist the specified URL.
- `firewall whitelist asn AS123456`: Whitelist the specified ASN.
- `firewall whitelist vpn`: Refresh VPN whitelist.
- `firewall whitelist remove all`: Remove all non-default entries.
- `firewall whitelist remove entry 8.8.8.8`: Remove the specified IP/range.
- `firewall whitelist remove comment "Apples"`: Remove entries with the comment "Apples".
- `firewall whitelist refresh`: Regenerate shared whitelist files.

### Example Import Commands:

- `firewall import blacklist file.txt "Apples"`: Ban all IPs from the URL/local file with the comment "Apples".
- `firewall import whitelist file.txt "Apples"`: Whitelist all IPs from the URL/local file with the comment "Apples".

### Example Deport Commands:

- `firewall deport blacklist file.txt`: Unban all IPs from URL/local file.
- `firewall deport whitelist file.txt`: Unwhitelist all IPs from URL/local file.

### Example Update Commands:

- `firewall update`: Standard update check - if nothing detected, exit.
- `firewall update check`: Check for updates only - won't update if detected.
- `firewall update -f`: Force update even if no changes detected.

### Example Settings Commands:

- `firewall settings autoupdate enable|disable`: Enable/disable Skynet autoupdating.
- `firewall settings banmalware daily|weekly|disable`: Enable/disable automatic malware list updating.
- `firewall settings logmode enable|disable`: Enable/disable logging.
- `firewall settings loginvalid enable|disable`: Enable/disable invalid packet logging.
- `firewall settings logsize 10` : Configure Skynet log size in MB 
- `firewall settings filter all|inbound|outbound`: Select what traffic to filter.
- `firewall settings unbanprivate enable|disable`: Enable/disable Unban_PrivateIP function.
- `firewall settings banaiprotect enable|disable`: Enable/disable banning IPs flagged by AiProtect.
- `firewall settings securemode enable|disable`: Enable/disable insecure settings being applied in WebUI.
- `firewall settings extendedstats enable|disable`: Enable/disable dnsmasq log lookups for blocked IPS.
- `firewall settings fs google.com/filter.list|disable`: Configure/disable fast malware list switching.
- `firewall settings syslog|syslog1 /tmp/syslog.log|default`: Configure custom syslog/syslog-1 location.
- `firewall settings iot unban|ban 8.8.8.8,9.9.9.9`: Unban/ban IOT device(s) (or CIDR) from accessing WAN (allow NTP/remote access via OpenVPN/Wireguard only).
- `firewall settings iot view`: View currently banned IOT devices.
- `firewall settings iot ports 123,124,125`: Allow port(s) to access WAN.
- `firewall settings iot ports reset`: Reset allowed port list to default.
- `firewall settings iot proto udp|tcp|all`: Select IOT allowed port protocol.
- `firewall settings iotlogging enable|disable`: Enable/disable IOT logging for protected devices.
- `firewall settings lookupcountry enable|disable`: Enable/disable country lookup for stat data.
- `firewall settings cdnwhitelist enable|disable`: Enable/disable CDN whitelisting.
- `firewall settings webui enable|disable`: Enable/disable WebUI.

### Example Debug Commands:

- `firewall debug watch`: Show debug entries as they appear.
- `firewall debug info`: Print useful debug info.
- `firewall debug info extended`: Debug info + config.
- `firewall debug genstats`: Update WebUI stats.
- `firewall debug clean`: Cleanup syslog entries.
- `firewall debug swap install|uninstall`: Install/uninstall SWAP file.
- `firewall debug backup`: Backup Skynet files to Skynet's install directory.
- `firewall debug restore`: Restore backup files from Skynet's install directory.

### Example Stats Commands:

- `firewall stats`: Compile stats with default top 10 output.
- `firewall stats 20`: Compile stats with customizable/optional top 20 output.
- `firewall stats tcp`: Compile stats showing only TCP entries.
- `firewall stats tcp 20`: Compile stats showing only TCP entries with customizable/optional top 20 output.
- `firewall stats search port 23`: Search logs for entries on port 23.
- `firewall stats search port 23 20`: Search logs for entries on port 23 with customizable/optional top 20 output.
- `firewall stats search ip 8.8.8.8`: Search logs for entries on 8.8.8.8.
- `firewall stats search ip 8.8.8.8 20`: Search logs for entries on 8.8.8.8 with customizable/optional top 20 output.
- `firewall stats search domain google.com 20`: Search logs for entries IP's resolving to domain with customizable/optional top 20 output.
- `firewall stats search malware 8.8.8.8`: Search malware lists for specified IP.
- `firewall stats search manualbans`: Search for all manual bans.
- `firewall stats search device 192.168.1.134`: Search for all outbound entries from local device 192.168.1.134.
- `firewall stats search reports`: Search previous hourly report history.
- `firewall stats search invalid`: Search for invalid packets.
- `firewall stats search iot`: Search for IOT packets.
- `firewall stats search connections ip|port|proto|id xxxxxxxxxx`: Search active connections.
- `firewall stats remove ip 8.8.8.8`: Remove log entries containing IP 8.8.8.8.
- `firewall stats remove port 23`: Remove log entries containing port 23.
- `firewall stats reset`: Reset all collected logs.

## About

> Skynet gained self-awareness after it had spread into millions of computer servers all across the world; realizing the extent of its abilities, its creators tried to deactivate it. In the interest of self-preservation, Skynet concluded that all of humanity would attempt to destroy it and impede its capability in safeguarding the world. Its operations are almost exclusively performed by servers, mobile devices, drones, military satellites, war-machines, androids and cyborgs (usually a terminator), and other computer systems. As a programming directive, Skynet's manifestation is that of an overarching, global, artificial intelligence hierarchy (AI takeover), which seeks to exterminate the human race in order to fulfill the mandates of its original coding. (▀̿Ĺ̯▀̿ ̿)
