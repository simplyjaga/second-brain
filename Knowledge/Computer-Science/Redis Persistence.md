# Redis Persistence

## What is it?

How Redis saves in-memory data to disk so it survives restarts. Two mechanisms — **RDB** (point-in-time snapshot) and **AOF** (append-only log of every write).

## Why is it needed?

Redis is in-memory; without persistence, a crash or restart loses everything. The two options trade off durability vs performance/snapshot size.

## How does it work?

### RDB (snapshot)

- Forks the process; child writes the whole dataset to a compact binary `dump.rdb`.
- Triggered on schedule (`save 900 1` = every 900s if ≥1 change) or manually (`BGSAVE`, `SAVE` — blocking).
- Fork uses copy-on-write — parent keeps serving; only modified pages are duplicated.

### AOF (append-only file)

- Logs every write command (`appendonly.aof`); on restart Redis replays it to rebuild state.
- `appendfsync` durability knob: `always` (every write — safest, slowest), `everysec` (default — lose ≤1s), `no` (OS decides).
- Grows forever → **AOF rewrite**: periodically forks and writes a minimal command set for current data (like RDB but in command form). Auto-triggers when file doubles.

### Together

- Can run both — AOF for durability, RDB for fast restarts/backups. On boot, AOF wins if enabled (more complete).
- Replication also relies on RDB-style snapshot + command stream to sync replicas.

## Important details

- Fork cost: fine on small datasets; on huge ones the COW memory spike and fork latency can hurt (a reason for `noeviction`-adjacent ops concerns).
- RDB can lose minutes of data on crash; AOF `everysec` loses at most 1s.
- `SAVE` is blocking — production uses `BGSAVE`.

## Log vs snapshot

RDB and AOF are concrete instances of the generic [[Log vs Snapshot]] trade-off — snapshot cost ∝ dataset size vs log cost ∝ mutation rate, which is why frequent RDB snapshots can't match AOF durability and why AOF needs rewrite/compaction.

## Related concepts

- [[Log vs Snapshot]] — the general pattern this note instantiates
- [[Processes Threads and Cores]] — fork/COW is process-level, not threads

## Collections

- [[Redis Internals]]

## Sources

- [[Redis Internals playlist]] — episode "Implementing AOF Persistence"

## Questions / Things to revisit

- RDB fork vs AOF rewrite internals — how exactly does the child serialize while parent mutates?
