---
name: db-migration-safety
description: Use when writing or reviewing database migration files (ALTER TABLE, CREATE INDEX, data backfills), especially for large tables in production. Triggers on migration PRs, schema changes, or Alembic/Knex/Prisma migration files.
---

# Database Migration Safety

## Overview

A migration that works on a 1000-row dev database can lock a 50M-row production table for minutes. This skill enforces safe migration patterns that avoid downtime, data loss, and lock contention.

## When to Use

- Writing DDL migrations (ALTER TABLE, CREATE INDEX, DROP COLUMN)
- Backfilling data in existing columns
- Renaming or modifying columns that are actively read/written
- Reviewing migration PRs from other engineers

## When NOT to Use

- Adding a new empty table (low risk, just do it)
- INSERT-only seed data in non-production environments
- Read-only analytics database changes

## Core Rules

### 1. Never Mix DDL and DML in One Migration

Schema changes and data backfills must be separate migrations. DDL locks the table; if the backfill fails halfway, rollback becomes dangerous.

### 2. Add Columns as Nullable First

```
-- Step 1 (migration A): add column as nullable
ALTER TABLE orders ADD COLUMN discount_cents INT NULL;

-- Step 2 (deploy): write to both columns
-- Step 3 (migration B): backfill existing rows
UPDATE orders SET discount_cents = 0 WHERE discount_cents IS NULL;

-- Step 4 (migration C): add NOT NULL constraint
ALTER TABLE orders ALTER COLUMN discount_cents SET NOT NULL;
```

### 3. Use Concurrent Index Creation

| Database | Safe Syntax |
|----------|-------------|
| PostgreSQL | `CREATE INDEX CONCURRENTLY idx_name ON table(col)` |
| MySQL 8.0+ | `ALTER TABLE table ADD INDEX idx_name (col), ALGORITHM=INPLACE, LOCK=NONE` |

Standard `CREATE INDEX` acquires an exclusive lock. On large tables, this blocks all reads and writes.

### 4. Batch Large Data Updates

Never `UPDATE table SET col = val` on millions of rows in one transaction.

```sql
-- Batch in chunks of 10,000
UPDATE orders SET discount_cents = 0
WHERE id IN (
  SELECT id FROM orders
  WHERE discount_cents IS NULL
  LIMIT 10000
);
```

### 5. Never Drop a Column in the Same Deploy as Stopping Reads

The safe sequence:
1. Deploy code that no longer reads the column
2. Wait for all pods to pick up the new code
3. Then drop the column in a separate migration

## Anti-Patterns

| Wrong | Right |
|-------|-------|
| `ALTER TABLE ... ADD COLUMN ... NOT NULL` | Add nullable first, backfill, then add constraint |
| `CREATE INDEX` (blocking) | `CREATE INDEX CONCURRENTLY` |
| Drop column + stop reading in same PR | Two separate deploys |
| Single UPDATE on 10M rows | Batched updates with LIMIT |
| Migration that can't be rolled forward | Write reversible migrations OR forward-fix migrations |

## Migration PR Checklist

- [ ] No blocking DDL on tables > 1M rows
- [ ] DDL and DML are in separate migration files
- [ ] Large backfills are batched
- [ ] Column drops are staged (code first, then schema)
- [ ] Index creation uses CONCURRENTLY (Postgres) or ALGORITHM=INPLACE (MySQL)
- [ ] Migration is reversible OR has a documented forward-fix plan
