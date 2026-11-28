# Database Tables Are Just Files

## What is it?

How a database engine actually stores rows on disk: fixed-size **pages** (also called blocks) packed into plain files. Tables, rows, and trees don't exist as far as the OS is concerned — what exists is files, plain stretches of bytes. Every operation — scan, index lookup, insert, update, delete — ends the same way: a page number becomes a **filename plus a byte offset**, the OS moves one page of bytes, and the engine interprets what those bytes mean.

## Why is it needed?

Triggering question: *where does the grid of rows actually live?* The OS knows files and byte ranges, nothing about rows or tables. The database engine is the only thing that interprets the bytes — so understanding storage means understanding how the engine decides which file and which offset to touch for any operation.

## How does it work?

A toy model makes the whole thing visible: page = 100 bytes, each row ~45 bytes so **2 rows per page**, **2 pages per file**, pages zero-indexed. (Real engines: pages of 4–16 KB, a multiple of the filesystem block size.)

### Pages and files — the file I/O idea

- A table's files are named after it: `students.0`, `students.1`, `students.2`. One arithmetic step turns a page number into file + offset:

```python
PAGE_SIZE = 100
PAGES_PER_FILE = 2

def locate(table, page):
    file = f"{table}.{page // PAGES_PER_FILE}"
    offset = (page % PAGES_PER_FILE) * PAGE_SIZE
    return file, offset
```

- **Reading a page** is just `seek(offset)` + `read(PAGE_SIZE)` — plain file I/O, no special disk API for databases.
- **Writing a page**: `write` only fills the process's buffer → `flush` hands bytes to the OS (which may still hold them in memory) → `fsync` waits until they're actually on disk. A write only counts as done after fsync.
- Every page mutation follows the same shape: **find page → edit bytes in memory → write the whole page back**. The row itself is never written separately.

### Row addresses and the heap

- A row's address is **(page, position) within a table** — e.g. `(4, 0)` = page 4, position 0. The table name picks the files (another table can have its own page 4, position 0 in *its* files); the address picks the spot inside them.
- Data pages form a **heap**: an unordered pile of rows in arrival order. Not sorted by id or anything else. Inserts go to the last page with room.
- **Deletes leave holes.** Indexes record row addresses, so rows can't slide within a page — a deleted row's position stays empty. Real engines track free space to reuse holes later.

### Indexes — the indexing idea

- An index is a **sorted two-column table**: `key → row address`. It's stored in its own files (`students_id_idx.*`) as a [[B+ Tree]] — same pages, same `locate`, same reads. No new kind of object on disk.
- The heap and the index hold the *same rows* in *different orderings*: the heap is a pile in arrival order, the index is sorted by one key. Add another index → another set of files, another ordering, all pointing into the same heap.
- **Internal pages** hold `key → child page number` (a page of the *same* tree). **Leaf pages** hold `key → (page, position)` — a row address in the *heap*.
- The tree descent always ends at a leaf, and the leaf is where the tree stops pointing at itself. The leaf entry has no file/table name — it doesn't need one: the **catalog** (itself a table in pages) already says which table this index belongs to, and that bridges the two namespaces. Only then does `locate` switch to the table's files.
- **How entries get there:** index created on an empty table → fills as rows arrive; created after rows exist → engine scans the heap top to bottom and writes entries in sorted order. Either way, from then on every insert/update/delete writes the heap page *and* the matching index leaf in the same breath.

### Where tree maintenance meets files

- When an insert lands in the middle of an index's sorted order, the full leaf **splits** and the new page is **appended at the end of the files** — a file can't open up space in the middle without rewriting everything after it. Sorted order lives in the tree's pointers; files are just a place to put pages. (Page 4 can be last in the files but second in the root's ordering.) The split itself — divide entries, push a range up — is the balancing machinery of the [[B+ Tree]].
- **Index pages repack, heap pages can't.** Nothing outside the tree points at an entry's position inside an index page — entries are found by key — so a rewritten index page packs its entries together and the rest is plain free space. Heap rows can't move like that because index entries record their exact positions; hence holes in the heap but not in the index.

## Important details

- Index maintenance cost is why you don't index everything: every extra index is another leaf write (and occasional split) per mutation.
- The OS contract never changes: a filename, an offset, and one page of bytes. All the machinery — pages, trees, catalogs — exists only to decide *which* filename and offset.

> [!question] Things that confused me
> - "If all data is in leaf pages, how does the engine know when to switch from index files to table files — is leaf-ness the check?" — Almost: page headers do say internal vs leaf, but the descent *always* ends at a leaf by design, so it's really two phases. Phase 1: tree descent, every pointer means "another page of this same tree", all reads stay in the index's files. Phase 2: row fetch, which only begins after the descent — the leaf's payload is by definition a heap address, and the catalog supplies the table name. Leaf-ness doesn't *cause* the file switch; it *ends the tree walk*, and what the leaf contains is an address in a different namespace.
> - "After a split moves entries to the new page, is the old space on the original page marked deleted, and reused by future middle inserts?" — There's no marked-deleted hole at all: writing a page rewrites the whole page, so the surviving entries get packed together and the rest is just free space. And yes — future inserts in that page's key range reuse the room directly.

## Related concepts

- [[B+ Tree]] — the data structure every index here is built from; splits and balancing live there
- [[Log vs Snapshot]] — LSM-tree storage engines are the write-optimized alternative to page-in-place storage
- [[Trie]] — a different lookup structure (key = path through a tree of characters, not comparisons in sorted pages)

## Projects / Journeys

## Sources

- My blog post: "How Does a Database Actually Store Your Rows in Blocks on Disk?" — the 10-row toy storage engine walkthrough this note distills: https://simplyjaga.github.io/blog/2026/11/27/database-file-storage-blocks/

## Questions / Things to revisit

- Buffer pool: pages are cached in memory, so not every page read hits disk — how does that layer sit on top of this?
- InnoDB's clustered index: the table itself *is* a B+ tree with full rows in the leaves — no heap at all. How does that change the picture?
