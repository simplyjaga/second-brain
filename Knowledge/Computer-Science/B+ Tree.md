# B+ Tree

## What is it?

A self-balancing search tree designed for storage that reads and writes in fixed-size **pages** (disk blocks). Unlike binary trees, each node is a whole page holding many keys — high **fan-out** — so the tree stays extremely shallow. The **B+** variant keeps all actual data (or pointers to it) in the **leaf pages only**; internal pages hold nothing but routing keys. It is the index structure underneath nearly every relational database — see [[Database Tables Are Just Files]].

## Why is it needed?

Two constraints force the design:

- **Disk reads happen one page at a time, and they're expensive.** A binary tree with a million keys is ~20 levels deep — up to 20 page reads per lookup. A page holding ~1000 keys gives fan-out of 1000: a million rows fit in **2–3 levels**, so any lookup costs 2–3 page reads.
- **Sorted order with cheap range scans.** Hashes give point lookups but can't do `<`, `BETWEEN`, or ordered iteration. A tree whose leaves are sorted (and linked) scans ranges by walking sideways.

## How does it work?

### Structure

- **Internal pages**: entries of the form `key range → child page number`. Pure routing — no row data.
- **Leaf pages**: entries of the form `key → payload` (in a database index, the payload is the row's address). All leaves sit at the **same depth**, always.
- Every page is **at least half full** (except the root) — bounded occupancy is what keeps the shape predictable.

### Search

Descend from the root: at each internal page, pick the key range covering the search key, follow to the child page, repeat. The descent **always ends at exactly one leaf** — the leaf is where the tree stops pointing at itself.

### Insert and the split — the entire balancing act

There are **no rotations** (unlike AVL or red-black trees). The whole balancing strategy is two operations: **split** on overflow, **merge/redistribute** on underflow. A split is three steps:

1. **Divide the entries** — first half stays on the existing page.
2. **Allocate a new page at the same level** — the second half goes there. Where the page physically lands is the storage layer's business, not the tree's (in a database file it gets *appended at the end* — the tree only cares about its page number).
3. **Push a separator up** — the parent page's key range splits into two ranges pointing at the two pages.

If the parent is also full, the split **propagates upward**, recursively. When the root itself splits, the tree grows one level taller — uniformly, for every leaf at once. That's why all leaves always stay at the same depth: there is no code path that can make one branch deeper than another.

**Splits buy room in advance.** After a split both pages are roughly half full, so most future inserts in that range just slide into existing room — one page rewrite, no split. Splits get rarer as the tree grows, which is why maintenance stays cheap at scale.

### Delete

The mirror image: remove the entry; if the page drops below half full, redistribute entries with a sibling or merge the two pages and pull the separator down from the parent. In practice many engines (PostgreSQL included) are lazy about merging — a sparse page costs little, and the next insert in that range often refills it.

### B-tree vs B+ tree

| | B-tree | B+ tree |
|---|---|---|
| Data stored | In **all** nodes (internal + leaf) | **Leaves only** — internal nodes are pure routing |
| Keys in internal nodes | Appear once, with data | Duplicated down to leaves as separators |
| Leaf pages linked? | No | Usually yes, sideways → cheap range scans |
| Point lookup | Can finish early at an internal node | Always descends to a leaf, but internal pages are smaller → higher fan-out |

Databases overwhelmingly use B+ trees: since internal pages carry no data, more routing keys fit per page, the fan-out is higher, and the tree is shallower.

## Important details

- Fan-out math: ~1000 entries per 8 KB page → level 1 root covers 1000 leaves, 2 levels ≈ a million rows, 3 levels ≈ a billion. Depth barely grows as data does.
- The invariants doing the work: **all leaves same depth** + **every page at least half full** + **ranges partition the key space with no gaps or overlaps**. Splits and merges exist only to maintain these three.
- Inserts/updates/deletes all run the same machinery — an update to an indexed column is internally delete-entry + insert-entry.

> [!question] Things that confused me
> - "Is the split just the B+ tree's version of balancing — and does the same apply to updates, deletes, and the heap?" — Yes for everything inside indexes: all mutations run the same split/merge machinery. No for the heap: a heap is not a tree at all, no balancing exists there — it's an unordered pile, which is exactly why scanning it is the slow path.
> - "Where does the new page from a split physically go?" — The tree doesn't care: it only records the page *number*. In a database file the new page is appended at the end (files can't open up space in the middle), so physical file order and logical tree order diverge — sorted order lives entirely in the pointers. See [[Database Tables Are Just Files]].

## Related concepts

- [[Database Tables Are Just Files]] — the primary application: every database index is a B+ tree whose pages live in plain files
- [[Log vs Snapshot]] — LSM trees are the write-optimized alternative index structure (buffer writes, flush sorted runs instead of splitting pages in place)
- [[Trie]] — a different lookup structure: key = path through characters, no key comparisons at nodes

## Projects / Journeys

## Sources

- My blog post: "How Does a Database Actually Store Your Rows in Blocks on Disk?" — includes a worked page-split example with toy numbers: https://simplyjaga.github.io/blog/2026/11/27/database-file-storage-blocks/

## Questions / Things to revisit

- InnoDB's clustered index: leaf payload = the full row, so the table *is* the tree. PostgreSQL instead: leaf payload = row address into a separate heap. Trade-offs?
- How leaf-page links make range scans work in practice (and whether engines actually maintain sibling pointers)
- B+ trees outside databases: filesystems, KV stores
