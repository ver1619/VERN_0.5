## Tests v0.5

**1. Low-Level Encoding & Safety Primitives**<br>

`TestInternalKeyEncodeDecode`<br>
Verifies lossless encoding/decoding of internal keys (user key + sequence + value type).<br>

`TestEncodeDecodeRecord`<br>
Ensures WAL record framing encodes and decodes correctly.<br>

`TestCRCFailureStopsDecode`<br>
Confirms CRC mismatches are detected and decoding halts to prevent corruption propagation.<br>

`TestPartialRecordFails`<br>
Ensures truncated or incomplete records are rejected.<br>

`TestExtractHelpers`<br>
Validates helper functions that extract user key, sequence number, and type from internal keys.<br>

**2. Ordering, Comparison & Iteration**<br>

`TestComparatorOrdering`<br>
Ensures comparator orders by:
  - User key (ascending)
  - Sequence number (descending)<br>

`TestSSTableIterator`<br>
Validates forward iteration over SSTable entries in correct sorted order.<br>

**3. Memtable Semantics**<br>

`TestMemtableInsertAndSize`<br>
Checks insert correctness and memory accounting.<br>

`TestMemtableOrderingByUserKey`<br>
Ensures memtable ordering by user key.<br>

`TestMemtableOrderingBySequenceDesc`<br>
Ensures newer versions shadow older ones for the same key.<br>

`TestMemtableStoresTombstone`<br>
Validates delete markers are stored and respected.<br>

**4. Read Path Composition**<br>

`TestMergeMemtableAndSSTable`<br>
Ensures correct merge semantics:
  - Memtable shadows SSTables
  - Newer entries shadow older ones
  - Tombstones suppress values<br>

**5. Manifest & Versioning**<br>

`TestManifestAppendAndDecode`<br>
Validates manifest record append and replay correctness.<br>

`TestManifestCorruption`<br>
Ensures corrupted manifest entries are detected and rejected safely.<br>

`TestVersionSetReplay`<br>
Ensures VersionSet reconstructs SSTable state from the manifest accurately.<br>

**6. WAL Segment Semantics**<br>

`TestSegmentAppendAndSync`<br>
Validates WAL segment append and fsync semantics.<br>

`TestSegmentReopenAppend`<br>
Ensures segments can be reopened and appended safely.<br>

`TestAppendAfterCloseFails`<br>
Confirms closed WAL segments reject writes.<br>

**7. WAL Lifecycle & Rotation**<br>

`TestWALAppendAndRotate`<br>
Ensures WAL rotates segments correctly once size thresholds are hit.<br>

`TestWALReopen`<br>
Validates WAL state reconstruction on restart.<br>

`TestWALCloseClosesAllSegments`<br>
Ensures clean shutdown closes all WAL resources.<br>

`TestWALTruncation`<br>
Ensures obsolete WAL segments are safely removed after flush.<br>

`TestTruncationIdempotence`<br>
Confirms WAL truncation can be retried safely.<br>

**8. Crash & Recovery Semantics (durability gaurantees)**<br>

`TestCrashBeforeWALFsync`<br>
Simulates crash before WAL fsync; ensures un-fsynced writes are not recovered.<br>

`TestRecoveryIsDeterministic`<br>
Ensures recovery produces identical state across runs.<br>

`TestReplayRepeatability`<br>
Confirms WAL replay can be repeated safely.<br>

`TestFullRecovery`<br>
Validates recovery across:
  - WAL
  - Memtables
  - SSTables
  - Manifest<br>

**9. Database-Level Integration (public API)**<br>

`TestDBPutGetDelete`<br>
Validates basic DB semantics via public API.<br>

`TestDBRecovery`<br>
Ensures DB state is restored correctly after restart.<br>

`TestOpenPutGet`<br>
Confirms DB open → write → read lifecycle works end-to-end.<br>

**10. Determinism & System Guarantees**<br>

`TestRecoveryIsDeterministic`<br>
Ensures identical on-disk state always yields identical in-memory state.<br>

---

## Test Results:

```go
=== RUN TestDBPutGetDelete
--- PASS: TestDBPutGetDelete (0.04s)
=== RUN TestDBRecovery
--- PASS: TestDBRecovery (0.01s)
=== RUN TestFullRecovery
--- PASS: TestFullRecovery (0.01s)
=== RUN TestVersionSetReplay
--- PASS: TestVersionSetReplay (0.02s)
=== RUN TestInternalKeyEncodeDecode
--- PASS: TestInternalKeyEncodeDecode (0.00s)
=== RUN TestComparatorOrdering
--- PASS: TestComparatorOrdering (0.00s)
=== RUN TestExtractHelpers
--- PASS: TestExtractHelpers (0.00s)
=== RUN TestMergeMemtableAndSSTable
--- PASS: TestMergeMemtableAndSSTable (0.00s)
=== RUN TestManifestAppendAndDecode
--- PASS: TestManifestAppendAndDecode (0.01s)
=== RUN TestManifestCorruption
--- PASS: TestManifestCorruption (0.00s)
=== RUN TestMemtableInsertAndSize
--- PASS: TestMemtableInsertAndSize (0.00s)
=== RUN TestMemtableOrderingByUserKey
--- PASS: TestMemtableOrderingByUserKey (0.00s)
=== RUN TestMemtableOrderingBySequenceDesc
--- PASS: TestMemtableOrderingBySequenceDesc (0.00s)
=== RUN TestMemtableStoresTombstone
--- PASS: TestMemtableStoresTombstone (0.00s)
=== RUN TestSSTableIterator
--- PASS: TestSSTableIterator (0.00s)
=== RUN TestEncodeDecodeRecord
--- PASS: TestEncodeDecodeRecord (0.00s)
=== RUN TestCRCFailureStopsDecode
--- PASS: TestCRCFailureStopsDecode (0.00s)
=== RUN TestPartialRecordFails
--- PASS: TestPartialRecordFails (0.00s)
=== RUN TestSegmentAppendAndSync
--- PASS: TestSegmentAppendAndSync (0.01s)
=== RUN TestSegmentReopenAppend
--- PASS: TestSegmentReopenAppend (0.02s)
=== RUN TestAppendAfterCloseFails
--- PASS: TestAppendAfterCloseFails (0.01s)
=== RUN TestWALTruncation
--- PASS: TestWALTruncation (0.02s)
=== RUN TestWALAppendAndRotate
--- PASS: TestWALAppendAndRotate (0.06s)
=== RUN TestWALReopen
--- PASS: TestWALReopen (0.01s)
=== RUN TestWALCloseClosesAllSegments
--- PASS: TestWALCloseClosesAllSegments (0.01s)
=== RUN TestRecoveryIsDeterministic
--- PASS: TestRecoveryIsDeterministic (0.02s)
=== RUN TestTruncationIdempotence
--- PASS: TestTruncationIdempotence (0.01s)
=== RUN TestCrashBeforeWALFsync
--- PASS: TestCrashBeforeWALFsync (0.54s)
=== RUN TestReplayRepeatability
--- PASS: TestReplayRepeatability (0.03s)
=== RUN TestOpenPutGet
--- PASS: TestOpenPutGet (0.01s)
PASS
ok vern_kv0.5/tests/crash 0.547s
ok vern_kv0.5/tests/determinism 0.042s
ok vern_kv0.5/tests/integration 0.029s
```
