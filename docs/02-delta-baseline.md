# 2. Delta Lake from scratch (the baseline)

Paimon is easiest to understand as "Delta, with one big idea swapped out." So first, precisely how Delta works.

A Delta table is just a directory containing two kinds of things: immutable Parquet data files, and a `_delta_log/` folder of numbered JSON commits. A commit holds no data — only a small list of actions such as "add file X (with these stats)" and "remove file Y." The commit numbers increase strictly: `000000.json`, `000001.json`, and so on.

![A Delta table = immutable data files + an append-only transaction log that decides which files are live.](img/diagram_04.png)

*A Delta table = immutable data files + an append-only transaction log that decides which files are live.*

## How a reader reconstructs the table

The current state of a Delta table is **derived**, not stored directly: replay every commit in order, tracking which files have been "added" but not yet "removed." The surviving set of Parquet files is the table right now. The folder might physically hold 50 files while the log says only 12 are live — readers trust the log, never a folder listing. From this you get:

- **Atomicity** — a write only counts once its JSON commit lands; half-written data files are invisible because no commit references them.
- **Time travel / snapshots** — replay the log only up to commit N to see an older version; the old data files are still present, frozen.
- **Optimistic concurrency** — two writers attempt the next commit number; one wins, the other detects the conflict and retries.

Replaying thousands of commits would get slow, so Delta periodically writes a **checkpoint** (a Parquet snapshot of "here is the full set of live files as of commit N"). Readers start from the latest checkpoint and replay only the few commits after it. (Paimon has an analogous idea — snapshots pointing at manifest lists — so the pattern carries over.)

## The defining trait: copy-on-write

Because Parquet files are immutable, Delta cannot edit a single row in place. To change one row, it must:

1. Find the file that contains the row (using the per-file min/max stats in the log).
2. Read that entire file.
3. Write a brand-new file with the one row changed and everything else copied across.
4. Commit: "remove the old file, add the new file."

This is **copy-on-write (COW)**. Reads are trivial and fast — just open the live files, no merging needed — but any write that touches existing rows rewrites whole files.

!!! warning "The villain: write amplification"
    Now imagine a stream delivering thousands of single-row updates per second. Copy-on-write would have Delta rewriting large files continuously — a *tiny logical change* causing a *huge physical write*, over and over. That ratio, **write amplification**, is the exact pain Paimon was built to remove.
