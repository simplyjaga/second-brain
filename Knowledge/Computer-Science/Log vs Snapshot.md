# Log vs Snapshot

## What is it?

The two fundamental strategies for making in-memory state durable:

- **Log** (write-ahead log, journal, append-only file): record every *mutation* as it happens. Recovery = replay the log.
- **Snapshot** (checkpoint, dump): periodically write the *entire state* to disk. Recovery = load the snapshot.

They are complementary, not competitors — most durable systems combine both (snapshot + log of changes since).

## Why is it needed?

Any system that keeps state in memory must survive crashes/restarts. The choice of strategy determines the trade-off between write-path cost, recovery speed, and durability granularity.

## How does it work?

### Cost models

- **Log**: write cost ∝ *mutation rate* — appending bytes per change. Durable at high frequency (every write or every second). Weakness: the file grows forever → needs **compaction** (rewrite to minimal form), and recovery is O(history).
- **Snapshot**: write cost ∝ *total dataset size* — serialize everything each time. Cheap at low frequency (hourly backups), catastrophic at high frequency. Weakness: loses everything since the last snapshot.

> [!question] Can't you just snapshot more often — e.g. every 1 second — and get log-level durability?

Same durability window, but the cost model kills it: every snapshot is O(dataset). A 1 GB dataset snapshotted per second = 1 GB/s of sustained disk writes; 50 GB → impossible. Meanwhile a log appends only the changes (KB/sec). Snapshot processes can also *overlap* if a dump outlives the interval — stacked forks + copy-on-write churn = memory balloon. **Making snapshots frequent is reinventing a log, badly.**

Systems that truly snapshot at high frequency (LSM-tree databases) cheat the same way: they flush only a small in-memory memtable — cost ∝ recent changes again, not total data.

## Important details

- Recovery cost is the mirror image: snapshots load fast (binary/compact), logs replay slowly (O(history)) — which is why logs need compaction and snapshots don't.
- Real-world instances:
  - **Redis**: RDB = snapshot, AOF = log (+ AOF rewrite = compaction). See [[Redis Persistence]].
  - **PostgreSQL**: WAL = log, base backup + checkpoint = snapshot.
  - **Kafka**: the log *is* the storage; compaction is log compaction.
  - **Event sourcing**: the event log is the source of truth; snapshots are optional accelerators.
  - **Filesystems/VMs**: journaling (log) vs image backup (snapshot).

## Related concepts

- [[Processes Threads and Cores]] — fork()/copy-on-write is what makes snapshots cheap to take on Unix
- [[Event Loop]] — durability work must not stall the serving thread; hence background threads/forks

## Projects / Journeys

## Sources

- [[Redis Internals playlist]] — episode "Implementing AOF Persistence"

## Questions / Things to revisit

- Where do hybrid schemes land — Redis AOF-with-RDB-preamble, Postgres WAL + base backups?
