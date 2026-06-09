# 11. Delta vs Iceberg vs Paimon — at a glance

A summary of the contrasts drawn throughout this document.

| Dimension                  | Delta Lake                                          | Apache Iceberg                                                       | Apache Paimon                                                                  |
|----------------------------|-----------------------------------------------------|----------------------------------------------------------------------|--------------------------------------------------------------------------------|
| Origin                     | Databricks (2019)                                   | Netflix (2018)                                                       | Alibaba / Flink community (2022)                                               |
| Metadata structure         | Append-only JSON log + checkpoints                  | Tree: metadata.json → snapshot → manifest list → manifest files      | Snapshots → manifests → LSM tree per bucket                                    |
| COW & MOR support          | Both; COW default, MOR via deletion vectors         | Both; configurable per-operation (`write.update.mode` etc.)          | Both; MOR default. MOW (deletion vectors) and COW available on the same dial    |
| How MOR is implemented     | Deletion vectors (bitmaps marking deleted rows)     | Position deletes + equality deletes (small delete files)             | LSM tree per bucket — built for continuous high-frequency churn                 |
| Streaming upserts          | Heavy write amplification in COW; DVs help but not engineered for it | Heavy write amplification in COW; delete files help but not engineered for it | Purpose-built — small cheap writes, background compaction                       |
| Primary keys               | Not part of the model                               | Not part of the model                                                | First-class; data sharded by key into buckets                                   |
| Partitioning               | Hive-style (visible)                                | Hidden partitioning + partition evolution                            | Hive-style + bucket-by-key                                                      |
| Combining same-key records | Single behaviour (latest overwrites)                | Single behaviour (latest overwrites via merge)                       | Pluggable merge engines: `deduplicate`, `partial-update`, `aggregation`, `first-row` |
| Read cost                  | Low and constant (open live files)                  | Low and constant (open live files, filter delete files)              | A merge whose cost depends on how many overlapping runs cover the key           |
| Native change stream       | Change Data Feed                                    | Incremental scans (snapshot diffs)                                   | Changelog producers: `none` / `input` / `lookup` / `full-compaction`            |
| Branches & tags            | No (snapshot versions only)                         | Yes (Iceberg 1.2+)                                                   | Yes (modelled on Iceberg's)                                                     |
| Schema evolution           | Add / drop / rename (by name + position)            | Full, by column ID — safe rename/reorder                             | Add / drop / rename                                                             |
| Partition evolution        | No                                                  | Yes (unique to Iceberg)                                              | No                                                                              |
| Ordering of versions       | By commit order                                     | By snapshot sequence                                                 | By per-bucket sequence number (or a chosen `sequence.field`)                    |
| Multi-engine support       | Spark-first; Trino / Flink growing                  | Broad: Spark, Flink, Trino, Snowflake, Athena, Hive, Dremio          | Flink-first; Spark / Trino / StarRocks growing                                  |
| Sweet spot                 | Spark/Databricks ecosystem, batch-heavy analytics   | Multi-engine batch analytics on very large tables                    | High-frequency streaming updates / CDC into the lake                            |

!!! success "The one-sentence summary"
    All three formats support both copy-on-write and merge-on-read, but **Delta and Iceberg** treat MOR as a delete-marker bolted onto a write-once file layout — fine for occasional updates, not engineered for continuous streaming churn. **Paimon** is the only one whose storage layout (an LSM tree of buckets, levels, and sorted runs) is built for MOR as the primary mode — which is why it's the right choice for high-frequency primary-key streaming updates and CDC.

## Picking one — a rough heuristic

- **You live on Databricks, workloads are mostly batch + occasional updates** → Delta is the path of least resistance.
- **Multi-engine shop (Trino + Spark + Flink + Snowflake), huge tables, complex partitioning needs** → Iceberg.
- **High-frequency CDC or streaming primary-key updates into the lake** → Paimon. (And note: Paimon 1.0+ can expose Iceberg-compatible metadata, so you can use Paimon for ingest and let Iceberg-only engines still read the table.)

---

*Reference: Apache Paimon, Apache Iceberg, and Delta Lake documentation. Prepared as an internal learning aid; verify version-specific option names against the release you deploy.*
