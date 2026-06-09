# 11. Delta vs Paimon — at a glance

A summary of the contrasts drawn throughout this document.

| Dimension                  | Delta Lake                                          | Apache Paimon                                                                  |
|----------------------------|-----------------------------------------------------|--------------------------------------------------------------------------------|
| Core data structure        | Flat Parquet files + an append-only transaction log | An LSM tree per bucket, tracked by snapshots and manifests                     |
| Update strategy            | Copy-on-write — pay at write time                   | Merge-on-read by default; MOW and COW available on the same dial               |
| Streaming upserts          | Expensive (heavy write amplification)               | Purpose-built — small cheap writes, background compaction                       |
| Primary keys               | Not part of the model                               | First-class; data is sharded by key into buckets                                |
| Combining same-key records | Single behaviour (latest overwrites)                | Pluggable merge engines: `deduplicate`, `partial-update`, `aggregation`, `first-row` |
| Read cost                  | Low and constant (open live files)                  | A merge whose cost depends on how many overlapping runs cover the key           |
| Native change stream       | Change Data Feed                                    | Changelog producers: `none` / `input` / `lookup` / `full-compaction`            |
| Ordering of versions       | By commit order                                     | By per-bucket sequence number (or a chosen `sequence.field`)                    |
| Sweet spot                 | Batch-heavy, append-mostly analytics                | High-frequency streaming updates / CDC into the lake                            |

!!! success "The one-sentence summary"
    Paimon swaps Delta's copy-on-write for an LSM tree with merge-on-read, which makes streaming primary-key updates cheap; everything else — buckets, compaction, table modes, merge engines, and the changelog model — exists to manage the consequences of that one swap.

---

*Reference: Apache Paimon documentation (master / 1.x). Prepared as an internal learning aid; verify version-specific option names against the release you deploy.*
