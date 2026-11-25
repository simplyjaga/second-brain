# Radix Trie

## What is it?

A radix tree (compressed trie / Patricia trie) is a [[Trie]] where any chain of single-child nodes is merged into one node. Each node stores a **string label** (`prefix`) instead of one character. Children are keyed by the first character of their label, so each node has at most one child per starting character.

## Why is it needed?

A plain trie uses one node per character — long shared prefixes waste nodes and depth. Merging chains shrinks both. This is the structure Redis uses (`rax`) for things like `KEYS` scanning and stream IDs.

## How does it work?

Example — inserting `jaga`, `jack`, `jama`, `jackie`, `jacy` in that order:

- `jaga` → leaf `jaga`
- `jack` → shares `ja`: **split** `jaga` into `ja` + `ga`, add `ck`
- `jama` → adds `ma` under `ja`
- `jackie` → `ck` fully matches, descends, adds leaf `ie`
- `jacy` → `ck` vs `cy` diverge at `c`: **split** `ck` into `c` + `k`, add `y`

```
root
 └─ "ja"
     ├─ "ga"   (end: jaga)
     ├─ "ma"   (end: jama)
     └─ "c"
         ├─ "k"   (end: jack)
         │   └─ "ie"  (end: jackie)
         └─ "y"   (end: jacy)
```

## Node and helper

```python
class RadixNode:
    def __init__(self, prefix=""):
        self.prefix = prefix      # edge label leading into this node
        self.children = {}        # first char of child's prefix -> child node
        self.is_end = False       # True if a stored word ends here


def common_prefix_len(a, b):
    """Length of the longest common prefix of a and b."""
    n = min(len(a), len(b))
    i = 0
    while i < n and a[i] == b[i]:
        i += 1
    return i


class RadixTree:
    def __init__(self):
        self.root = RadixNode()
```

## Insert

At each node, look at the child keyed by the next character. Three cases:

1. No such child: add a leaf with the whole remainder.
2. The child's label fully matches the start of the remainder: walk into the child.
3. The label partly matches: split it at the divergence point, creating a `mid` node for the shared part.

```python
def insert(self, word):
    node = self.root
    i = 0

    while i < len(word):
        c = word[i]
        rest = word[i:]

        # Case 1: no edge starts with this character -> new leaf
        if c not in node.children:
            leaf = RadixNode(rest)
            leaf.is_end = True
            node.children[c] = leaf
            return

        child = node.children[c]
        j = common_prefix_len(child.prefix, rest)

        # Case 2: whole edge matches -> descend
        if j == len(child.prefix):
            node = child
            i += j
            continue

        # Case 3: partial match -> split the edge at position j
        mid = RadixNode(child.prefix[:j])       # shared part
        child.prefix = child.prefix[j:]         # old child keeps the rest
        mid.children[child.prefix[0]] = child
        node.children[c] = mid

        node = mid
        i += j

    node.is_end = True
```

## Search

Walk down the tree, requiring each edge label to match the word exactly at the current position. The word is found only if the walk ends on a node marked `is_end`.

```python
def search(self, word):
    node = self.root
    i = 0
    while i < len(word):
        child = node.children.get(word[i])
        if child is None:
            return False
        if not word.startswith(child.prefix, i):
            return False
        i += len(child.prefix)
        node = child
    return node.is_end
```

## Starts with (prefix check)

The prefix can end in the middle of an edge, so compare against the partial label instead of requiring a full match.

```python
def starts_with(self, prefix):
    node = self.root
    i = 0
    while i < len(prefix):
        child = node.children.get(prefix[i])
        if child is None:
            return False
        remaining = prefix[i:]
        j = common_prefix_len(child.prefix, remaining)
        if j == len(remaining):      # prefix ends inside this edge: match
            return True
        if j < len(child.prefix):    # diverged inside this edge: no match
            return False
        i += j
        node = child
    return True
```

## Usage

```python
t = RadixTree()
for w in ["jaga", "jack", "jama", "jackie", "jacy"]:
    t.insert(w)

print(t.search("jack"))        # True
print(t.search("jac"))         # False (prefix, not a stored word)
print(t.starts_with("jac"))    # True  (ends inside the "c" edge)
print(t.starts_with("jag"))    # True
print(t.starts_with("jau"))    # False
```

## Delete

Not covered in the session — high level: remove the `is_end` flag, delete a leaf with no children, and merge a non-end node that has one child back into its parent (the inverse of splitting).

## Practice

- Visualizer: `Attachments/radix_visualizer_v2.html` — file lives in this vault; open it in a browser (not Obsidian) to step through insertions and watch edge splits happen.

## Important details

- Insert splits an edge **only** when a new word diverges partway through a label.
- `search` checks `is_end`, so a prefix of a stored word is not a match; `starts_with` succeeds if the prefix ends inside an edge.
- Deletion can merge nodes — splits and merges keep the "no single-child chains" invariant.

## Related concepts

- [[Trie]]

## Projects / Journeys

- [[Redis Internals]] — Redis uses a radix tree (`rax`)

## Sources

- Claude session — radix tree implementation walkthrough

## Questions / Things to revisit

- How does Redis's rax differ — node encoding, memory packing?
