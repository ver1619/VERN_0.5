## Manifest File

The Manifest is a persistent metadata log of engine decisions that tells the database how to rebuild its state after a crash.

**What it stores?**<br>
The Manifest stores durable metadata about engine state, including:
- WAL segment creation and sealing
- WAL truncation boundaries
- Other structural decisions needed for recovery
- It never stores user data.

**How it works?**<br>
- Structural changes are appended to the Manifest
- Each entry is fsynced before taking effect
- On startup, the Manifest is replayed to reconstruct engine state
- WAL replay then proceeds according to the Manifest-defined boundaries<br>

**The Manifest remembers how the engine was structured, so recovery is deterministic.**


---

### Internal Data Model

All operations are represented as internal keys composed of:
- User key
- Sequence number
- Value type (PUT / DELETE)

-> `(user key ASC, seq.no DESC, Value type)`


---

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
SSTables becomes an active persistence mechanism only in later versions when memtable flushing and compaction are introduced.