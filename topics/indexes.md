# Indexes

<!-- TOC -->

* [Indexes](#indexes)
    * [Tuning](#tuning)
    * [Concept](#concept)
        * [Indexes accelerate reads but add write costs:](#indexes-accelerate-reads-but-add-write-costs)

<!-- TOC -->

## Tuning

1. TODO

## Concept

A PostgreSQL index is a separate **on-disk** data structure that provides a faster path to locate rows than scanning the
whole table.
Conceptually, an index stores:

* Search keys (derived from one or more columns or expressions), plus
* A pointer to the table row (the row’s TID: block number + offset within the block).

[The planner](https://www.interdb.jp/pg/pgsql03/01.html#314-planner-and-executor) may choose an Index Scan, Index-only
Scan or Bitmap Index Scan to find candidate row locations, then fetch
the table rows.

**Indexes accelerate reads but add write costs!**

* _INSERT_: add index entries.
* _UPDATE_: may add new entries (and sometimes remove old ones logically).
* _DELETE_: leaves “dead” entries until cleanup.
  This is why autovacuum/vacuum and reindexing considerations matter operationally.

## Index types

### B-tree (default, most common).

Best for: equality and ordering

* `=`, `<`, `<=`, `>`, `>=`
* `BETWEEN`
* `ORDER BY`
* Prefix pattern matching with LIKE 'abc%'

Typical fields: integers, timestamps, UUIDs, numeric, short text.

* This is default index for PRIMARY KEY and UNIQUE.
* Supports multi-column indexes with _left-to-right matching rules_.

Shortly about **left-to-right matching rules**:

If we have the table:\
`CREATE TABLE my_table(col1 INT, col2 INT, col3 INT)`

and we have the next index:\
`CREATE INDEX my_table_idx on my_table(col1, col2, col3);`

Then next queries can use index:
```sql
SELECT FROM my_table WHERE col1 < 5;
SELECT FROM my_table WHERE col1 < 5 AND col2 > 10;
SELECT FROM my_table WHERE col1 < 5 AND col2 > 10 AND col3 <= 25;
```

But these queries can **not**:
```sql
SELECT FROM my_table WHERE col2 < 10;
SELECT FROM my_table WHERE col3 <= 25;
SELECT FROM my_table WHERE col2 < 10 AND col3 <= 25;
```
