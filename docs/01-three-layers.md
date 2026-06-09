# 1. The three layers, and what a table format provides

People blur three different things together, and that is the root of most confusion. Keep them separate.

1. **Storage** — where the bytes physically live: S3, ADLS, GCS. A dumb bucket of files with no notion of "tables."
2. **File format** — how a single file is laid out internally: Parquet, ORC, Avro. Parquet is columnar, storing each column's values together, so reading one column (e.g. summing a revenue column) touches far less data than a row store.
3. **Table format** — the bookkeeping layer that makes a folder of files behave like one coherent table. Delta, Iceberg, Hudi, and Paimon all live at this layer.

!!! info "Key idea"
    Delta and Paimon are both *table formats* sitting on top of Parquet files in object storage. So a Delta-vs-Paimon comparison is almost never about the storage or the file format — those are usually identical. It is about **the bookkeeping that decides what counts as the current state of the table**.

## What a table format gives you that raw files can't

Object storage has no transactions. If you simply call a folder of 100 Parquet files "a table," you immediately hit:

- A reader arriving mid-write sees half-written, inconsistent files.
- There is no safe way to update or delete rows — and no agreement on what a reader sees during a rewrite.
- Two writers running at once can silently clobber each other.
- There is no way to ask "what did this table look like an hour ago?"

A table format fixes all of this with one move: readers never look at the folder directly. They read a metadata/log layer that tells them exactly which files are currently valid. From that single indirection you get the four things every modern table format advertises: **ACID transactions, snapshot isolation, time travel** (read an older version), and **schema evolution**. Delta and Paimon both provide these; they differ in *how* the metadata is structured and, crucially, in *how updates are physically applied*.
