# Database catalogue

Load when the change touches queries, schema, indexes, migrations, ORM usage, or any
storage with quotas.

## Don't do it

- Do not write what you can derive. Counters, timestamps of last activity, "is expired"
  flags, and anything computable from other columns plus the clock should not be columns.
- Do not read what you will not use: `SELECT` the columns you need; skip the join whose
  result is discarded.
- Do not store the same fact twice unless a measured read pattern requires denormalizing,
  and then write down how the copies reconcile.
- Do not `SELECT COUNT(*)` on large tables per request; maintain a counter in a batch job
  or accept an estimate.

## Do it once

- Cache immutable reference data (item banks, catalogues, config) in KV / memory with a
  version key; read the DB only on version change.
- Upsert instead of read-then-write when the semantics allow it.
- Use `RETURNING` to avoid a follow-up read after an insert/update.

## Do it later, in bulk

- Batch inserts in one statement (`INSERT ... VALUES (...), (...)`); one round trip, one
  transaction.
- Aggregate on a schedule: write raw events cheaply, roll up by cron into summary tables
  users read.
- Coalesce updates to the same row within a request into one `UPDATE`.

## Better algorithm

- Every `WHERE`, `JOIN`, and `ORDER BY` on a large table has an index that covers it, and
  no index exists that no query uses. Check with `EXPLAIN QUERY PLAN` (SQLite/D1) or
  `EXPLAIN ANALYZE` (Postgres). "SCAN TABLE" on anything non-trivial is a bug.
- Covering indexes so the query never touches the table.
- Cursor pagination on an indexed, unique ordering.
- Replace correlated subqueries with joins or window functions.
- Composite indexes ordered by equality columns first, then range, then sort.

## Smaller

- Narrow types: integers for ids and enums, epoch integers for timestamps if the engine
  lacks a compact date type, booleans as integers, no `TEXT` for structured data that is
  queried.
- Avoid storing JSON blobs you later filter on; promote the filtered field to a column.
- Short-lived data gets a TTL and a cleanup job; tables that only grow eventually cost on
  every scan.

## Parallel

- Independent reads in a request run in a single batch call where the driver supports it
  (`db.batch([...])` on D1); one round trip carries several statements.

## Tighter

- Prepared statements reused across calls; parameter binding, never string interpolation.
- Transactions scoped tightly; hold no lock across network I/O.
- Avoid ORM features that generate `SELECT *` or lazy-load relations in loops; inspect the
  generated SQL at least once per new query.

## Migrations

- Additive only during rollout: add column with default, deploy code that writes both,
  backfill in batches, then switch reads, then drop later. The previous version must run
  against the new schema.
- Backfills in chunks sized to the write quota, with resume markers.
- Never run a migration that rewrites a large table inside a request path.

## Measure

- Count statements per handler with the driver's metrics or a wrapping counter; assert in
  tests.
- Row reads / writes per operation (D1 reports these); multiply by expected daily volume
  against the quota.
- Query plan for every new or changed query, saved in the PR.
