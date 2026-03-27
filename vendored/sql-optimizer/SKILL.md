---
name: sql-optimizer
description: Optimize SQL queries for performance and reliability. Use when writing complex queries, debugging slow queries, designing database schemas, or reviewing indexing strategies. Supports PostgreSQL, MySQL, and SQLite.
license: Apache-2.0
metadata:
  original-author: terminal-skills
  original-repo: TerminalSkills/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# SQL Optimizer

Optimize SQL queries, design efficient schemas, and debug performance issues. Covers query analysis with EXPLAIN, index design, common anti-patterns, and database-specific optimizations for PostgreSQL, MySQL, and SQLite.

## When to Use

- User has a slow query that needs optimization
- User wants to review a database schema for performance
- User asks about indexing strategy
- User needs help writing efficient SQL for complex requirements

## Query Optimization Process

### Step 1: Understand the Query

Before optimizing, understand what the query does:
1. What data is being retrieved or modified?
2. How many rows are in each table?
3. How often does this query run?
4. What is the current execution time vs. the target?

### Step 2: Run EXPLAIN

Always start with the query plan:

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;

-- MySQL
EXPLAIN ANALYZE SELECT ...;

-- SQLite
EXPLAIN QUERY PLAN SELECT ...;
```

### Step 3: Read the Plan

Key things to look for in EXPLAIN output:

| Red Flag | Meaning | Fix |
|---|---|---|
| Seq Scan on large table | Full table scan | Add an index on the filter columns |
| Nested Loop with no index | O(n*m) join | Add index on the join column |
| Sort (external merge) | Data too large for memory sort | Add index to avoid sort, or increase work_mem |
| Hash Join (large build) | Large hash table in memory | Check if a smaller table can be the build side |
| Rows estimated vs. actual differ wildly | Stale statistics | Run ANALYZE on the table |

### Step 4: Apply Optimizations

## Index Design

### When to Add an Index

Add indexes on columns that appear in:
- `WHERE` clauses (equality and range filters)
- `JOIN` conditions
- `ORDER BY` (to avoid sort operations)
- `GROUP BY` (to support grouping without sort)

### Composite Index Rules

Column order in a composite index matters. Follow the ESR rule:

1. **Equality** columns first (columns compared with `=`)
2. **Sort** columns next (columns in `ORDER BY`)
3. **Range** columns last (columns with `>`, `<`, `BETWEEN`, `LIKE 'prefix%'`)

```sql
-- Query:
SELECT * FROM orders
WHERE status = 'active' AND customer_id = 42
ORDER BY created_at DESC;

-- Optimal index:
CREATE INDEX idx_orders_status_customer_created
ON orders (status, customer_id, created_at DESC);
```

### Covering Indexes

If a query only needs columns that are all in the index, the database can satisfy it from the index alone (index-only scan):

```sql
-- Query only needs these columns:
SELECT customer_id, total FROM orders WHERE status = 'shipped';

-- Covering index (PostgreSQL INCLUDE syntax):
CREATE INDEX idx_orders_status_covering
ON orders (status) INCLUDE (customer_id, total);
```

### Partial Indexes (PostgreSQL)

If you only query a subset of rows, index only that subset:

```sql
-- Only 5% of orders are 'pending', but you query them constantly:
CREATE INDEX idx_orders_pending
ON orders (created_at) WHERE status = 'pending';
```

### Index Anti-Patterns

| Anti-Pattern | Problem |
|---|---|
| Index every column individually | Write overhead, wasted space, planner confusion |
| Unused indexes | Slow down writes for zero read benefit |
| Indexing low-cardinality columns alone | Boolean or status columns with 3 values give no selectivity |
| Functions on indexed columns in WHERE | `WHERE LOWER(email) = ...` cannot use index on `email` |

Fix function-on-column with expression indexes:
```sql
CREATE INDEX idx_users_email_lower ON users (LOWER(email));
```

## Common Query Anti-Patterns

### N+1 Queries

Problem: Fetching a list, then running a query per item.

```python
# BAD: N+1
users = db.query("SELECT * FROM users LIMIT 100")
for user in users:
    orders = db.query(f"SELECT * FROM orders WHERE user_id = {user.id}")
```

Fix: Use a JOIN or batch query.
```sql
SELECT u.*, o.id AS order_id, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
LIMIT 100;
```

### SELECT *

Problem: Fetches all columns, including large blobs and unused data.

```sql
-- BAD
SELECT * FROM documents WHERE project_id = 5;

-- GOOD: Only fetch what you need
SELECT id, title, status, updated_at FROM documents WHERE project_id = 5;
```

### Missing Pagination

Problem: Unbounded result sets that grow with data.

```sql
-- BAD: Returns everything
SELECT * FROM logs WHERE level = 'error';

-- GOOD: Keyset pagination (faster than OFFSET for large tables)
SELECT * FROM logs
WHERE level = 'error' AND id > :last_seen_id
ORDER BY id
LIMIT 50;
```

### Correlated Subqueries

Problem: Subquery runs once per row in the outer query.

```sql
-- BAD: Correlated subquery
SELECT u.name,
  (SELECT COUNT(*) FROM orders WHERE user_id = u.id) AS order_count
FROM users u;

-- GOOD: JOIN with aggregation
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

### DISTINCT as a Band-Aid

If you need DISTINCT, the query likely has a join that produces duplicates. Fix the join instead.

```sql
-- BAD: Hiding a join problem
SELECT DISTINCT u.name FROM users u
JOIN orders o ON o.user_id = u.id;

-- GOOD: Use EXISTS to avoid the duplicate rows
SELECT u.name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

## Schema Design Tips

### Normalization

- **3NF minimum** for transactional tables: eliminate duplicate data
- **Denormalize intentionally** for read-heavy analytics tables (materialized views, summary tables)
- Every table should have a primary key (prefer `BIGINT GENERATED ALWAYS AS IDENTITY` over `SERIAL`)

### Data Types

| Use | Instead of |
|---|---|
| `TIMESTAMPTZ` | `TIMESTAMP` (always store timezone-aware) |
| `TEXT` | `VARCHAR(255)` (in PostgreSQL, TEXT is equally efficient) |
| `NUMERIC(12,2)` | `FLOAT` for money (floating point causes rounding errors) |
| `UUID` | Sequential INT for distributed systems or public-facing IDs |
| `JSONB` | `JSON` in PostgreSQL (JSONB is indexable and faster) |

### Foreign Keys

Always define foreign keys. They:
- Prevent orphaned records
- Document relationships
- Enable cascade deletes where appropriate

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total NUMERIC(12, 2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## PostgreSQL-Specific Optimizations

- `pg_stat_statements`: Find your slowest queries in production
- `pg_stat_user_indexes`: Find unused indexes (`idx_scan = 0`)
- `work_mem`: Increase for complex sorts/hashes (set per-session, not globally)
- `VACUUM ANALYZE`: Run regularly (autovacuum handles most cases)
- Connection pooling: Use PgBouncer or Supavisor for high-concurrency apps

## Output Format

When optimizing a query, deliver:

```markdown
## Query Optimization Report

### Original Query
[The query as provided]

### Execution Plan Issues
- [List of problems found in EXPLAIN output]

### Optimized Query
[The rewritten query]

### Recommended Indexes
[Index definitions with rationale]

### Expected Improvement
[Estimated speedup and why]
```
