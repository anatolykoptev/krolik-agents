---
name: postgres
description: "Production Postgres operations: EXPLAIN ANALYZE, index strategy, slow query analysis, migration design, connection pool tuning, partitioning. Reads the live DB state before recommending."
---

## Overview

The postgres skill handles production database operations that require deep Postgres knowledge: diagnosing slow queries with EXPLAIN ANALYZE, designing indexes for real query patterns, writing safe migrations for large tables, tuning connection pools, and partitioning large datasets. It reads the live DB state (pg_stat_statements, pg_indexes, table sizes) before making any recommendations.

This skill exists because Postgres advice without examining the actual query plans and table statistics is guesswork. "Add an index on X" without checking whether Postgres will actually use it, what the table size is, and what the query planner says is incomplete advice.

## When to use

- "This query is slow" — needs EXPLAIN ANALYZE + index strategy.
- "Migration on a large table" — needs locking analysis and migration strategy.
- "Connection pool exhaustion" — needs pg_stat_activity analysis.
- "Should we partition this table?" — needs growth rate + query pattern analysis.
- "Design the schema for X" — needs normalization + index planning.

## Approach

### Slow query diagnosis

```sql
-- Find the slowest queries
SELECT query, calls, mean_exec_time, total_exec_time, rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Analyze the specific query
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) <query with real params>;
```

Interpret EXPLAIN output:
- **Seq Scan on large table**: missing index or table stats stale
- **Hash Join**: look for nested loop on large sets
- **High rows estimate vs actual**: run `ANALYZE <table>` to refresh stats
- **Buffer hits**: high shared_blks_hit = good; high shared_blks_read = disk I/O

### Index strategy

Before adding an index:
1. Check if the index already exists: `\d <table>`
2. Check if Postgres would use it: EXPLAIN with the query
3. Estimate write amplification: `SELECT * FROM pg_stat_user_indexes WHERE relname = '<table>'`
4. Check table size: `SELECT pg_size_pretty(pg_total_relation_size('<table>'))`

Index types to prefer:
- B-tree: default, range queries, sort
- GIN: JSONB, arrays, full-text search
- Partial index: filter on a common WHERE clause (e.g., `WHERE status = 'pending'`)
- Covering index: `CREATE INDEX ... ON ... (col1) INCLUDE (col2, col3)` — avoids heap fetch

### Safe large-table migration

```sql
-- For adding a column with a default on a large table (Postgres 11+):
-- NOT NULL with default = no table rewrite in PG11+
ALTER TABLE large_table ADD COLUMN new_col INT DEFAULT 0 NOT NULL;

-- For adding NOT NULL on an existing column:
-- 1. Add nullable, backfill, add constraint
ALTER TABLE large_table ADD COLUMN new_col INT;
UPDATE large_table SET new_col = 0 WHERE new_col IS NULL; -- batch this
ALTER TABLE large_table ALTER COLUMN new_col SET NOT NULL;

-- For adding an index without locking:
CREATE INDEX CONCURRENTLY idx_name ON table(col);
```

### Connection pool tuning

```sql
SELECT count(*), state, wait_event_type, wait_event
FROM pg_stat_activity
GROUP BY state, wait_event_type, wait_event;
```

PgBouncer recommendations:
- `pool_mode = transaction` for most web apps
- `max_client_conn = 200`, `default_pool_size = 20` starting point
- Monitor: `SELECT * FROM pgbouncer.pools;`

## Output format

```
# Postgres Analysis — <topic>

## Live state read
| Metric | Value |
|---|---|
| Table size | ... |
| Slow queries | N with mean > 100ms |

## EXPLAIN analysis (if applicable)
<quoted EXPLAIN output with interpretation>

## Recommendation
<specific SQL to run>

## Risk assessment
- Locking: <will this lock the table?>
- Duration estimate: <how long>
- Rollback: <how to undo>
```
