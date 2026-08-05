# RDS Maintenance Playbook

## Pre-maintenance checks

```sql
SELECT pg_is_in_recovery();
SELECT * FROM pg_stat_replication;
SELECT pg_database_size(current_database());
```

## Major version upgrade (blue-green)

1. Create blue-green deployment
2. Wait for replication lag = 0
3. Run extension compatibility check
4. Switchover (< 1 min downtime)
5. Monitor for 24h
6. Delete old instance

## Post-upgrade

```sql
ANALYZE VERBOSE;  -- update statistics
REINDEX DATABASE current_database;  -- rebuild indexes
```
