---
name: mac-health
description: Diagnose overall Mac hardware and system health (storage, memory pressure, battery, thermal, load, crashes). Use when the user asks for a Mac health check, system diagnosis, "check again", battery or disk health, or why the Mac feels slow or hot.
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

## Report

Lead with one verdict: healthy, watch, or act. Then short sections:

- **Machine** — model, chip, memory, macOS, uptime, SIP
- **Storage** — SMART, Data volume used and free
- **Memory** — pressure, compressor, whether swap is growing
- **Battery** — percent, charging, condition, cycle count, max capacity
- **Stability and load** — load averages, thermal warnings, kernel panics, newest diagnostic reports

On a repeat check in the same conversation, compare load, battery, swap, and free disk to the previous reading. Call out only what changed.

Omit serial numbers. Skip recommendations when the verdict is healthy.
