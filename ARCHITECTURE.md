# VernKV Architecture

This document describes the internal architecture of VernKV v0.5.

## High-Level Overview

VernKV is structured as a layered storage engine.

### Write Path (PUT / DELETE)

```
Client
  │
  │ Put(key, value) / Delete(key)
  ▼
Assign sequence number
  │
  ▼
Create WAL Batch
  │
  ▼
Append batch to WAL segment
  │
  ▼
fsync(WAL)   ← durability boundary
  │
  ▼
Insert record into Memtable
  │
  ▼
Return success to client (Acknowledged)
```

**RULE** : <br>
A write is acknowledged only after WAL fsync. <br>
Memtable updates are never durable by themselves.

---

### WAL Durability Boundary

```
Append record to WAL buffer
        │
        ▼
Write bytes to OS page cache
        │
        ▼
      fsync
        │
        ▼
OS guarantees persistence
        │
        ▼
Write is now durable
```

**Crash cases**
- Crash before fsync → record must NOT appear after recovery
- Crash after fsync → record MUST appear after recovery

---

### WAL Record Framing

```
Put / Delete request
        │
        ▼
Assign sequence number
        │
        ▼
Construct internal key
(user key + seq + type)
        │
        ▼
Encode value (or tombstone)
        │
        ▼
Build WAL payload
(internal key + value)
        │
        ▼
Compute record length
        │
        ▼
Compute checksum (CRC)
        │
        ▼
Assemble WAL record frame
[length | payload | checksum]
        │
        ▼
Write is now framed
```

After framing, recovery can always detect whether the record is complete, valid, or corrupted.

---

### Crash Recovery Flow

```
Process start
    │
    ▼
Open Manifest
    │
    ▼
Replay Manifest
    │
    ▼
Build VersionSet
    │
    ▼
Determine WAL cutoff sequence
    │
    ▼
Discover WAL segments (old → new)
    │
    ▼
Replay WAL records above cutoff
    │
    ├─ if corruption → STOP safely
    ▼
Rebuild Memtable
    │
    ▼
Set next sequence number
    │
    ▼
Recovered engine state
```

---

### WAL Truncation

```
Some WAL segments become obsolete
        │
        ▼
Confirm data is durably reflected
(in MemTable / immutable on-disk state)
        │
        ▼
Determine truncation boundary
(oldest WAL segment still required)
        │
        ▼
Append truncation record to Manifest
        │
        ▼
fsync Manifest
        │
        ▼
──────── TRUNCATION SAFETY BOUNDARY ────────
        │
        ▼
Delete obsolete WAL segments
        │
        ▼
Truncation complete
```
None of the WAL segments are deleted until the Manifest durably records that it is safe to do so.

---

### Read Path Flow(Internal Iterators)

```
Get(key)
  │
  ▼
Create Memtable Iterator
  │
  ▼
Create SSTable Iterators
  │
  ▼
Merge Iterator
  │
  ▼
Compare InternalKeys
  │
  ▼
Select newest version
  │
  ▼
If tombstone → NotFound
Else → return value
```

---

### Complete Write-Path

```
Client issues Put/Delete
        │
        ▼
Assign sequence number
        │
        ▼
Encode internal key + value
        │
        ▼
Frame WAL record
(length + payload + checksum)
        │
        ▼
Append record to WAL buffer (userspace)
        │
        ▼
Write WAL buffer to OS page cache
        │
        ▼
    fsync(WAL)
        │
        ▼
──────── DURABILITY BOUNDARY ────────
        │
        ▼
OS guarantees persistence
        │
        ▼
Apply record to MemTable
  (InternalKey order)
        │
        ▼
Acknowledge write to client
```

A write is not visible and not acknowledged until it crosses the durability boundary.

--- 

