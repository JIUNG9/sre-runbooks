# VACUUM Tuning

## Autovacuum settings
```sql
ALTER TABLE high_churn SET (
  autovacuum_vacuum_scale_factor = 0.01,
  autovacuum_analyze_scale_factor = 0.005,
  autovacuum_vacuum_cost_delay = 2
);
```

## Monitoring
```sql
SELECT relname, n_dead_tup, n_live_tup,
       round(n_dead_tup::numeric / greatest(n_live_tup, 1) * 100, 1) as dead_pct,
       last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 10;
```

## Bloat check
```sql
SELECT tablename,
       pg_size_pretty(pg_total_relation_size(tablename::text)) as total_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::text) DESC;
```

## Updated: monitoring tips
- Note added 2026-02-03

## Updated: performance tuning
- Note added 2026-02-10

## Updated: security hardening
- Note added 2026-03-01

## security hardening note
- Updated 2026-04-07
