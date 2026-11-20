---
name: mac-health
description: Diagnose overall Mac hardware and system health (storage, memory pressure, battery, thermal, load, crashes) and decide whether anything should change. Use when the user asks for a Mac health check, system diagnosis, "check again", "improve the health", battery or disk health, or why the Mac feels slow or hot.
---

# Mac health

Read-only diagnosis. Do not delete files, kill processes, change power settings, or install anything.

## Collect

Run these together:

```bash
system_profiler SPSoftwareDataType SPHardwareDataType
uptime
memory_pressure
df -h / /System/Volumes/Data
diskutil info disk0
pmset -g batt
pmset -g therm
system_profiler SPPowerDataType
ps -axo %cpu,rss,comm -r | head -12
ls -lt /Library/Logs/DiagnosticReports | head -15
```

If uptime is under a few hours or load looks high, name the top CPU processes. Do not treat post-boot Spotlight or File Provider spikes as hardware failure.

Do not run `brctl status`. It hangs. For sleep holds, use `pmset -g assertions` and stop after the named assertions.

## Report

Lead with one verdict: healthy, watch, or act. Then short sections:

- **Machine** — model, chip, memory, macOS, uptime, SIP
- **Storage** — SMART, Data volume used and free
- **Memory** — pressure, compressor, whether swap is growing
- **Battery** — percent, charging, condition, cycle count, max capacity
- **Stability and load** — load averages, thermal warnings, kernel panics, newest diagnostic reports

On a repeat check in the same conversation, compare load, battery, swap, and free disk to the previous reading. Call out only what changed.

Omit serial numbers. Skip recommendations when the verdict is healthy.

## Improve

When the user asks to improve health, remeasure first. Change nothing unless one of these is true.

- SMART is not Verified, or free space on the Data volume is under 20 GB.
- Swap-outs are climbing across two checks, or system-wide memory free is under 10%.
- Battery condition is not Normal, or maximum capacity is under 80%.
- A kernel panic exists, or the same forced restart repeats after the last check.
- Load stays above the CPU core count after the top processes are named, and those processes are not the apps the user is actively using.

A load of about 2 to 4 on a 10-core Mac, with WindowServer or Cursor on top, is the session. Do not quit Cursor, Grok, or other user apps to release idle sleep. Name the sleep assertion and leave it.

Do not delete caches, kill processes, or change `pmset` when the verdict is healthy.
