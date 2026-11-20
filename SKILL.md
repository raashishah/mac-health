---
name: mac-health
description: Diagnose Mac hardware health and run the matching fix when a gate fails. Use when the user asks for a Mac health check, system diagnosis, "check again", "improve the health", "how to improve", battery or disk health, or why the Mac feels slow or hot.
---

# Mac health

Read-only until a gate below fails. Do not delete files, kill processes, change power settings, or install anything while every gate passes.

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

- **Machine.** Model, chip, memory, macOS, uptime, SIP.
- **Storage.** SMART, Data volume used and free.
- **Memory.** Pressure, compressor, whether swap is growing.
- **Battery.** Percent, charging, condition, cycle count, max capacity.
- **Stability and load.** Load averages, thermal warnings, kernel panics, newest diagnostic reports.

On a repeat check in the same conversation, compare load, battery, swap, and free disk to the previous reading. Call out only what changed.

Omit serial numbers.

When the user asks how to improve, or to improve health, print this table. Use the measured value. Set Run to yes only when that gate failed and you ran the action.

| Gate | Measured | Action if it fails | Run |
| --- | --- | --- | --- |
| SMART |  | Stop. Do not repair or erase the disk. Tell the user to copy files off the Mac. |  |
| Free space |  | Delete cache directories under ~/Library/Caches except ms-playwright. |  |
| Memory |  | Quit the largest process that is not WindowServer, kernel_task, loginwindow, or this chat's app. |  |
| Battery |  | No command. Tell the user the battery needs service. |  |
| Panic or repeated forced restart |  | Read the newest panic and name the process or extension. Do not reboot. |  |
| Load |  | Take a second sample 15 seconds later. Quit a non-system runaway. Leave Spotlight, File Provider, and photoanalysisd running. |  |

## Actions

Run an action only when its gate fails.

1. SMART is not Verified. Quote `diskutil info disk0`. Do not run `diskutil repairVolume`. Do not erase the disk. Tell the user to copy files off the Mac.
2. Free space on the Data volume is under 20 GB. Run `du -hd 1 "$HOME/Library/Caches" | sort -hr | head -20`. Delete only directories inside `$HOME/Library/Caches`, and keep `ms-playwright`.
3. Swap-outs rose since the previous check, or system-wide memory free is under 10%. Run `ps -axo rss,pid,comm -m | head -8`. Quit the largest process that is not WindowServer, kernel_task, loginwindow, or the app running this chat. Use `kill` on that pid. Do not start with `kill -9`.
4. Battery condition is not Normal, or maximum capacity is under 80%. Report the cycle count and maximum capacity. Tell the user the battery needs service.
5. A kernel panic file exists, or the same forced restart happened again after the last check. Read the newest panic in `/Library/Logs/DiagnosticReports`. Name the process or extension. Do not reboot.
6. Load stays above the CPU core count on two samples taken at least 15 seconds apart, and the top process is not an app the user is using. If that process is outside `/System` and `/usr`, quit it with `kill`. If it is `spotlightknowledged`, `fileproviderd`, or `photoanalysisd`, do not kill it. Report that process as the limiter.

A load of about 2 to 4 on a 10-core Mac, with WindowServer or Cursor on top, fails no gate. An idle-sleep assertion from Cursor is not a gate. Do not quit Cursor to clear it.

Do not delete caches, kill processes, or change `pmset` when every gate passes.
