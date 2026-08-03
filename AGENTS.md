# Skynet-Zero Agent Conventions

This document outlines the strict guidelines, workflows, and philosophies that all AI agents MUST follow when working on the `Skynet-Zero` repository. This is a highly optimized, RAM-only fork of Asuswrt-Merlin's Skynet.

## 1. Git Workflow & Repository Safety
- **Push to Origin**: The repository has been locally detached from Adamm00's upstream repository. The `origin` remote now points directly to the user's repository (`underd0se/Skynet-Zero`). All pushes must go to `origin`.
- **Isolated Branches**: Always work on a separate feature or fix branch (e.g., `feature/awk-optimization` or `fix/dnsmasq-loop`). Do not commit directly to `master` or `development`.
- **Ask Before Action**: NEVER commit, push, PR, or merge code without explicitly presenting the changes and asking the user for permission first.

## 2. Documentation & Versioning
- **Strict README Separation**: 
  - The `master` branch `README.md` must remain exactly as it is unless explicitly asked to modify it. It represents the stable release.
  - The `development` branch `README.md` must ALWAYS be updated with the latest changelog entries for every new feature or fix. Do not accidentally merge the development changelog into the master README.
- **Version Bumping**: When committing functional changes, always update the semantic version (semver) and the date inside the `firewall.sh` ASCII banner. (Example: use `v1.1.3-dev` for the development branch, and `v1.1.2` for the master branch so agents do not confuse them).

## 3. Testing & Performance Validation
- **Test Before Committing**: All code changes must be tested natively on the Asuswrt router *before* being committed to the local branch.
- **Comprehensive Coverage**: Test scripts must cover the happy path, alternative execution paths, and extreme edge cases. Use mock processes or simulations for edge cases (rather than testing destructive actions live) to avoid bricking the router.
- **Resource Profiling**: When running tests, you MUST actively monitor system resources via background tasks (`top` or `/proc` polling). You must track and report:
  - CPU saturation and spikes
  - RAM availability / Memory drops
  - Disk/Swap I/O usage
  - Total execution timing (speed)
- **Data-Driven Proof**: After testing, you must present the user with mathematical proof (numbers, stats, speedups) comparing the old logic vs the new logic.
- **Wait for the User**: If testing requires a hardware reboot or physical interaction, do not assume the state. Ask the user and wait for their explicit signal before proceeding.
