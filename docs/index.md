# Apache Paimon

**A practical primer — how the format works internally, and how it differs from Delta Lake.**

A from-the-ground-up explanation of Paimon's internal design: the LSM-tree storage model, merge-on-read, compaction and the read/write table modes, sequence-based ordering, the four merge engines, and the changelog/CDC model — with the Delta Lake contrast drawn out at every step.

Prepared as a shareable team reference, based on the Apache Paimon documentation.

## Contents

1. [The three layers, and what a table format provides](01-three-layers.md)
2. [Delta Lake from scratch (the baseline)](02-delta-baseline.md)
3. [The fundamental trade-off: copy-on-write vs merge-on-read](03-cow-vs-mor.md)
4. [Inside Paimon: buckets, the LSM tree, sorted runs & levels](04-lsm-tree.md)
5. [Why Level 0 files overlap](05-l0-overlap.md)
6. [The read path: merge-on-read in action](06-read-path.md)
7. [Compaction & the table modes (MOR / MOW / COW)](07-compaction-modes.md)
8. [Sequence numbers, ordering & RowKind](08-sequence-rowkind.md)
9. [Merge engines: how same-key records combine](09-merge-engines.md)
10. [Changelog producers: emitting a correct CDC stream](10-changelog-producers.md)
11. [Delta vs Paimon — at a glance](11-delta-vs-paimon.md)

!!! tip "How to read this"
    Each section builds on the previous one. The single load-bearing idea is in Sections 3–4 (copy-on-write vs merge-on-read, and the LSM tree). Once that clicks, everything later — compaction, merge engines, the changelog model — follows directly from it.
