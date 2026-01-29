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

## SStables in v0.5
**Role:**
In v0.5, SSTables define the immutable on-disk data model and the read-path contract for persisted data.<br>

They establish:
- Immutability of on-disk state
- Total ordering via the internal key comparator
- CRC-verified, corruption-aware reads
- Version-aware merging with in-memory state<br>

SSTables in v0.5 serve as a correctness anchor for future lifecycle operations (flush, compaction), not as an active write target.

**Limitations (Intentional)**:<br>
In v0.5, SSTables:
- Are not produced by memtable flush
- Do not participate in compaction
- Do not reduce memory usage
- Do not optimize read performance
- Do not participate in recovery replay<br>

They are read-only structures used to lock invariants, not to manage data movement.<br>

**Design Intent:**<br>
The presence of SSTables in v0.5 separates the data model from the data lifecycle.<br>
v0.5 answers:
- What does immutable on-disk data look like?
- How is it ordered?
- How is it read safely?
- How does it interact with newer versions?

**Version Scope Note**<br>
SSTables become an active persistence mechanism only in later versions when memtable flushing and compaction are introduced.

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
