# *VernKV v0.5*
[![Go Reference](https://pkg.go.dev/badge/github.com/ver1619/VernKV.svg?style=flat-square)](https://pkg.go.dev/github.com/ver1619/VernKV)
![Go Version](https://img.shields.io/badge/Go-1.22%2B-1f2937?logo=go&logoColor=00ADD8&style=flat-square)
![License](https://img.shields.io/github/license/ver1619/VERN_v0.1?label=license&color=yellow)

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/43ce00ec-3c74-4f7e-a5ce-0f5b4019f1f7"
    alt="VernKV logo"
    width="300"
  >
</p>

***VernKV*** is a correctness-first storage engine that incrementally builds towards a real **LSM-style** key–value store.

**Version v0.1 - Foundations**<br>
Version v0.1 established the structural foundations of VernKV. It defined the core data model, in-memory state (MemTable), and the basic write/read flow that follows an LSM-style design. This version focused on clarity of structure and deterministic behavior within a single process, with attempting to provide durability, crash recovery and persistence guarantees.<br>
🔗 -> [VernKV v0.1](https://github.com/ver1619/VERN_v0.1)<br>

**Version v0.5 - Write Correctness & Internal Data Model**<br>
Version v0.5 builds on these foundations and ensures that every acknowledged write is durable, ordered, atomically applied, internally consistent and safely recoverable across crashes.

## v0.5 Guarantees:
- **Crash Recovery & Durability**<br>
  Data is fully recoverable after crashes,  Once a write is acknowledged, it has been durably persisted to disk. Crashes at any point—including abrupt termination will not cause acknowledged writes to be lost.
- **Total Write ordering**<br>
  All write operations are totally ordered using a monotonic sequence-number–based internal key. This order is preserved across crashes and recovery.

- **Single Correct State After Recovery**<br>
After a crash, the database always reconstructs exactly one correct state, fully implied by the persisted on-disk data. There are no ambiguous or partially applied states.

- **Immutability of Persisted Data**<br>
Once written, on-disk data structures are immutable. State evolution occurs only through the creation of new files and explicit metadata updates. 

- **Well-Defined Failure Boundaries**<br>
Crashes may interrupt execution at any point, but never violate durability, ordering, or recovery invariants. Failure behavior is explicitly bounded and deterministic.

- **Internal Consistency Across Layers**<br>
In-memory state, on-disk state, and metadata are mutually consistent at all times. No layer can observe a write that another layer cannot justify from durable state.<br>
---
> **Invariant:** All externally observable behavior in v0.5 is derivable solely
> from persisted WAL and Manifest state.<br>

## Explicit Non-Goals:
v0.5 intentionally does not provide:
- Snapshot isolation or read/access semantics
- Compaction
- SSTable flushing from memtable
- Range scans or prefix scans 
- Bloom filters
- CLI or user-facing tools
- Performance optimizations or benchmarks<br>

These features will be explored in later versions..

## Tests (v0.5)

Vern_KV v0.5 relies on an extensive test suite to validate core correctness and recovery guarantees.
Testing is treated as the primary correctness interface.

### Unit Tests<br>
Unit tests validate individual components in isolation, including:
- Write-Ahead Log (segmentation, framing, checksums)
- Manifest and metadata replay
- InternalKey encoding and comparator ordering
- Memtable insertion and ordering
- SSTable read-only format and iterators
- Internal and merge iterators

---

### Integration & Crash Tests<br>
Integration tests validate full engine behavior across process boundaries, including:
- End-to-end write correctness semantics
- Deterministic crash recovery
- WAL fsync durability boundaries
- Prefix-safe WAL truncation
- Version-aware read correctness
- Tombstone behavior across layers<br>

Crash tests use real process termination (SIGKILL - abrupt termination) to validate recovery correctness under abrupt failure.<br>

All tests can be run with:

```bash
go test ./...
```

> **NOTE**: v0.5 does not include memtable flush or compaction; tests focus exclusively on write correctness, recovery determinism, and internal consistency.

<br>**MORE....**

🔗 [High-Level Overview](ARCHITECTURE.md)<br>
🔗 [Project Scope](ProjectScope.md)<br>
🔗 [Refer](RefCorner.md)<br>


