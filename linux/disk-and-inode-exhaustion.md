# Disk and Inode Exhaustion

A filesystem can fail in three ways that all surface as `No space left on device`
(`ENOSPC`): out of blocks, out of inodes, or space held by files that no longer have
a name. `df -h` only shows the first, so check all three before deleting anything.

## Tell them apart

```bash
df -h {{mount}}    # blocks: Use% near 100
df -i {{mount}}    # inodes: IUse% near 100
```

| `df -h` | `df -i` | Likely cause |
| --- | --- | --- |
| full | normal | Large files, logs, or deleted-but-open files |
| normal | full | Millions of small files (session dirs, mail spool, cache shards) |
| full | full | Both; start with the inode consumers, they are the harder to see |
| normal | normal | Quota, a different mount than the one you checked, or a read-only remount |

If `df` shows free space and writes still fail, run `findmnt -no OPTIONS {{mount}}`.
A filesystem the kernel remounted `ro` after an I/O error reports free space and
rejects every write. `dmesg -T | grep -i -E 'ext4|xfs|remount'` shows why.

## Blocks full: find what holds the space

```bash
# Largest directories on this filesystem only (-x stays on one mount)
du -xh --max-depth=2 {{mount}} 2>/dev/null | sort -rh | head -20

# Largest single files
find {{mount}} -xdev -type f -size +500M -exec ls -lh {} + 2>/dev/null
```

When `du` adds up to much less than `df` reports, the difference is held by something
`du` cannot see. The usual cause is the next section.

## Deleted but still open

Deleting a file removes its name. The blocks are freed only when the last process
closes it. A service that keeps writing to a rotated or deleted log keeps the space
allocated, and `du` no longer counts it.

```bash
# Open files with a link count below 1, i.e. deleted
lsof +L1 {{mount}}
# COMMAND   PID  USER  FD  TYPE  SIZE/OFF   NLINK  NAME
# java     4121  app   7w  REG   38654705664  0    /var/log/app/app.log (deleted)
```

`lsof` can be slow on a busy host. Without it, list the same thing from `/proc`:

```bash
ls -l /proc/{{pid}}/fd 2>/dev/null | grep '(deleted)'
```

To free the space without restarting the process, truncate the file through its
descriptor. This is a write to the live log, so only do it when losing the content is
acceptable:

```bash
: > /proc/{{pid}}/fd/{{fd}}
```

If the process can be restarted or told to reopen its logs (`systemctl reload`,
`kill -HUP`), do that instead. The space returns when the descriptor closes.

## Inodes full: find the directory

Each file uses one inode regardless of size. `ext4` fixes the inode count when the
filesystem is created, so a small-file workload can exhaust inodes at 10% block use.
`xfs` allocates inodes dynamically and rarely hits this, except when `maxpct` caps it.

```bash
# Entry count per top-level directory
for d in {{mount}}/*/; do printf '%s\t%s\n' "$(find "$d" -xdev | wc -l)" "$d"; done | sort -rn | head
```

Repeat one level down in the directory with the highest count. `ls` on a directory
with millions of entries is slow and sorts in memory; use `ls -f` or `find -maxdepth 1`.

Deleting a large flat directory with `rm -rf` is slow because it unlinks entry by
entry. `find {{dir}} -type f -delete` is steadier on memory and can be stopped partway.

Fixing it for good means stopping the producer (a cache without eviction, a
session store on disk, a job that writes one file per event). On `ext4` the inode count
can only be raised by recreating the filesystem with `mkfs.ext4 -N` or `-i`.

## Log rotation that actually frees space

Rotation fails to free space when the writer keeps the old descriptor.

- `copytruncate` copies the file and truncates the original in place. The writer's
  descriptor stays valid, but lines written between the copy and the truncate are lost,
  and the copy briefly needs as much free space as the log.
- `postrotate` with a reload or `SIGHUP` makes the process reopen its file. It loses
  nothing, and it works only if the service handles the signal.
- Compress and cap retention in the same stanza, or the rotated files fill the disk
  instead:

```text
/var/log/{{service}}/*.log {
    daily
    rotate 7
    maxsize 500M
    compress
    delaycompress
    missingok
    notifempty
    postrotate
        systemctl reload {{service}} >/dev/null 2>&1 || true
    endscript
}
```

`maxsize` rotates a file that grows past the limit before the daily run, but
`logrotate` itself runs from a daily timer by default. For a log that can fill the disk
in an hour, run it hourly (`/etc/cron.hourly` or an extra timer) or have the
application rotate by size.

Check a config without running it: `logrotate -d /etc/logrotate.d/{{service}}`.

On systemd hosts, the journal has its own cap. `journalctl --disk-usage` shows its
size, and `SystemMaxUse=` in `journald.conf` limits it. To shrink it now:
`journalctl --vacuum-size=500M`.

## Reserved blocks

`ext4` reserves 5% of blocks for root by default. On a 2 TB data volume that is
100 GB that non-root writers cannot use, and `df` shows the volume full while root
can still write. Check with `tune2fs -l {{device}} | grep -i 'reserved block'`. For a
data-only volume, `tune2fs -m 1 {{device}}` lowers the reserve; leave it alone on the
root filesystem.

## Triage order

1. `df -h` and `df -i` on the failing mount, and `findmnt` for a read-only remount.
2. Blocks full and `du` adds up lower than `df`: `lsof +L1`, then reload or restart the holder.
3. Blocks full and `du` agrees: `du -xh --max-depth=2`, then act on the largest consumer.
4. Inodes full: count entries per directory, then stop the producer before deleting.
5. After freeing space, fix the cause (rotation config, retention, eviction), and add an
   alert on `node_filesystem_avail_bytes` and `node_filesystem_files_free` so the next
   one pages before 100%.
