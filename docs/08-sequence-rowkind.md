# 8. Sequence numbers, ordering & RowKind

**When versions of a key collide, "newest" is decided by a sequence number — and where that counter lives matters.**

Each row carries a **sequence number**: a monotonically increasing counter maintained **per bucket** by that bucket's writer — not per primary key, and not one global counter for the whole table.

!!! info "Why per-bucket is exactly right"
    You only ever compare sequence numbers between two rows that share a primary key, and all versions of a key always land in the same bucket. So every comparison you'd ever need happens inside one bucket, where its single counter already gives a correct order. A global counter would force every parallel writer to coordinate on one number — destroying the parallelism buckets exist to provide.

## Controlling the order

- **Default = input order** — the last record received is the last to merge (and therefore wins).
- **`sequence.field`** — for streams that can arrive out of order, point Paimon at a column (e.g. `update_time`) so the record with the largest value in that field is treated as latest, regardless of arrival order. Supported types include integer/bigint and timestamps.

## RowKind — every row knows what it is

Alongside its sequence number, every row carries a **RowKind** tag:

| RowKind | Meaning                                                       |
|---------|---------------------------------------------------------------|
| `+I`    | Insert — a new row.                                           |
| `-U`    | Update before-image — the row's value before a change.        |
| `+U`    | Update after-image — the row's value after a change.          |
| `-D`    | Delete — supersedes earlier versions; the key is removed.     |

A delete is therefore just a `-D` row with a high sequence number. A **superseded version** is any older row for a key that a newer write has replaced — reads ignore it, and compaction physically deletes it. The before/after pair (`-U` / `+U`) becomes essential in the next section, where downstream consumers need both to react correctly.
