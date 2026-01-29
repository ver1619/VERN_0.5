## Project Scope - v0.5

Vern_KV v0.5 is a core correctness and internal data model milestone.<br>
This version focuses exclusively on establishing durable writes, deterministic recovery, and a well-defined internal storage model.<br>


**v0.5 implements and guarantees:**<br>
- Write-ahead logging with fsync-based durability
- Batched fsyncs (fsync per N writes {configurable}) over fsync per write
- Crash-safe recovery with deterministic replay
- Monotonic sequence-number–based total write ordering 
- Append-only, immutable on-disk data structures
- Explicit metadata authority via a manifest log
- WAL segmentation to stop unbounded growth of single WAL file
- Record framing and checksums
- Safe WAL truncation of segments driven by metadata
- Tombstone-based delete semantics
- InternalKey encoding and global comparator contract
- Insert-only, rebuildable memtable
- Immutable SSTable data model (read-only)
- Iterator-based read path with version-aware merging
- Minimal engine API (Open, Put, Delete, Get)
- Extensive unit, integration, and crash testing

---

### Version Boundary
v0.5 prioritizes **correctness over capability**.
By fully defining write semantics, recovery behavior, and internal invariants first, later versions can safely evolve lifecycle management, access semantics, and concurrency without redesigning the storage core.