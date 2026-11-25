# Trie

## What is it?

A tree where each node is one **character**, and a path from the root spells out a string. Words sharing a prefix share the same path — so "car" and "card" share `c → a → r`. Nodes can be flagged as "end of word" so a prefix path can itself be a stored word.

## Why is it needed?

Fast prefix lookups: autocomplete, spell-check, IP routing, dictionary search. Finding all words with prefix `p` costs O(len(p)) to reach the prefix node, then just walks that subtree — independent of how many words are stored.

## How does it work?

### Structure

Each node holds: children (a map/char-indexed array), and an `is_end` flag. The root is empty.

### Insert

Walk character by character, creating missing nodes; flag the last node.

```python
class TrieNode:
    def __init__(self):
        self.children = {}   # char -> TrieNode  (hash-map approach)
        self.is_end = False

def insert(root, word):
    node = root
    for ch in word:
        node = node.children.setdefault(ch, TrieNode())
    node.is_end = True
```

### Search / prefix search

Walk the chars; fail early if a char is missing. For `search`, also require `is_end` at the final node. For `starts_with`, reaching the node is enough.

### Delete

Trickier than insert — can't just unset `is_end` if the word is a prefix of others, and must clean up nodes only if no other word needs them.

```python
def delete(node, word, i=0):
    if i == len(word):
        node.is_end = False
        return len(node.children) == 0   # safe to remove this node?
    ch = word[i]
    if ch not in node.children:
        return False                     # word not present
    if delete(node.children[ch], word, i + 1):
        del node.children[ch]            # child became useless
    return len(node.children) == 0 and not node.is_end
```

Rules at each node on the way back up:
- Delete the child pointer **only if** the child has no other children and isn't end-of-another-word.
- Unset `is_end` on the final node always (the word is gone even if its path stays).

### Update

No special case needed — **delete the old word, then insert the new one.** Since delete correctly prunes and insert rebuilds shared prefixes, update = `delete(old); insert(new)`.

## Hash-map vs pointer-array children

Two ways to store children:

| | Hash map (`dict`) | Fixed array / pointers |
|---|---|---|
| Node size | Small, grows with actual children | 26 (or 256) slots always |
| Lookup | O(1) average, hash overhead | O(1) direct index |
| Memory | Frugal for sparse alphabets | Wasteful when alphabet is large but few children used |
| Code | `node.children[ch]` | `node.children[ord(ch) - ord('a')]` |

With the **pointer approach** you fix the pointer per case — same logic, just index by `ord(ch) - base` instead of a dict key. Choose array for small fixed alphabets (a–z), hash map when alphabets are large/unknown.

## Important details

- Time: O(L) for insert/search/delete where L = word length.
- Space cost is the trade-off vs a plain hash set — you pay it for prefix queries.
- `is_end` matters: "car" stored + "card" stored → the `r` node is both mid-path and end-of-word.

## Related concepts

- [[Radix Trie]] — compressed trie; merges single-child chains

## Projects / Journeys

- [[Redis Internals]]

## Sources

- [Implement Trie (Prefix Tree) — NeetCode](https://www.youtube.com/watch?v=oobqoCJlHA0) — insert/search with Python dict approach
- [What is the Trie data structure — Gaurav Sen](https://www.youtube.com/watch?v=YG6iX28hmd0) — delete flow, pointer approach

## Questions / Things to revisit

- Where else radix tries show up (Redis rax, IP routing tables).
