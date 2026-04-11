# Linux Performance Debugging

## USE Method (Utilization, Saturation, Errors)

```bash
# CPU
mpstat -P ALL 1
sar -u 1 5

# Memory
free -h
vmstat 1 5

# Disk
iostat -xz 1

# Network
sar -n DEV 1
ss -s
```

## Quick triage
```bash
uptime           # load average
dmesg -T | tail  # kernel errors
vmstat 1 5       # overall
mpstat -P ALL 1  # per-CPU
free -h          # memory
iostat -xz 1     # disk
sar -n DEV 1     # network
```

## Updated: production patterns
- Note added 2026-02-11

## update for latest version
- Updated 2026-04-11
