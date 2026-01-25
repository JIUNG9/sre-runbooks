# PostgreSQL Query Optimization

## EXPLAIN ANALYZE
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders
WHERE created_at > now() - interval '7 days'
AND status = 'pending';
```

## Key patterns
1. Covering index: `CREATE INDEX idx ON orders (status, created_at) INCLUDE (total)`
2. Partial index: `CREATE INDEX idx ON orders (created_at) WHERE status = 'pending'`
3. BRIN index for time-series: `CREATE INDEX idx ON events USING brin (created_at)`

## Statistics
```sql
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;
ANALYZE orders;
```

## Updated: monitoring tips
- Note added 2026-01-22

## Updated: production patterns
- Note added 2026-01-25
