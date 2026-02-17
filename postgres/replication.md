# PostgreSQL Replication

## Streaming replication check
```sql
-- Primary
SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS replay_lag_bytes
FROM pg_stat_replication;

-- Replica
SELECT pg_is_in_recovery(),
       pg_last_wal_receive_lsn(),
       pg_last_wal_replay_lsn(),
       pg_last_xact_replay_timestamp();
```

## Logical replication
- Publisher/Subscriber model
- Per-table granularity
- Useful for major version upgrades
- Does NOT replicate DDL

## Updated: performance tuning
- Note added 2026-02-01

## Updated: best practices
- Note added 2026-02-13

## Updated: security hardening
- Note added 2026-02-17
