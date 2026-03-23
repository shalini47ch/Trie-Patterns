# Trie-Patterns

# 🌲 Trie Problems — Complete Guide with Intuition & Templates

> All solutions use the standard Node-based Trie template (Python).

---

## 📦 Base Template

```python
class Node:
    def __init__(self):
        self.links = [None for i in range(26)]
        self.flag = False  # end of word

    def containsKey(self, ch):
        return self.links[ord(ch) - ord("a")] != None

    def get(self, ch):
        return self.links[ord(ch) - ord("a")]

    def put(self, ch, node):
        self.links[ord(ch) - ord("a")] = node

    def setEnd(self):
        self.flag = True

    def isEnd(self):
        return self.flag


class Trie:
    def __init__(self):
        self.root = Node()

    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            if not node.containsKey(ch):
                node.put(ch, Node())
            node = node.get(ch)
        node.setEnd()

    def search(self, word: str) -> bool:
        node = self.root
        for ch in word:
            if not node.containsKey(ch):
                return False
            node = node.get(ch)
        return node.isEnd()

    def startsWith(self, prefix: str) -> bool:
        node = self.root
        for ch in prefix:
            if not node.containsKey(ch):
                return False
            node = node.get(ch)
        return True
```

---

## 📁 Categories

- [1. Introductory Questions](#1-introductory-questions)
- [2. Trie with Bit Manipulation](#2-trie-with-bit-manipulation)
- [3. Trie Involving Strings](#3-trie-involving-strings)
- [4. Trie Involving Recursion](#4-trie-involving-recursion)
- [5. Trie Involving File System](#5-trie-involving-file-system)

---

## 1. Introductory Questions

### 🟡 Implement Trie (Prefix Tree) — Medium

**Intuition:** A Trie is a tree where each root→node path spells out a string. Each node holds 26 children (one per letter) and a flag marking "a word ends here."

**Key insight:** Three operations: insert (walk + create nodes), search (walk + check `isEnd`), `startsWith` (walk only, no end check).

> See base template above.

---

### 🔴 Trie Delete — Hard

**Intuition:** You can't just remove a node if other words share the prefix. Recurse down, then on the way back up remove nodes that are (1) not a word-end and (2) have no children.

**Key insight:** Use recursive backtracking. A node is deletable only if it has no other children and isn't an end of another word.

```python
def delete(self, word: str) -> bool:
    def _delete(node, word, depth):
        if not node:
            return False
        if depth == len(word):
            if not node.isEnd():
                return False
            node.flag = False  # unmark end
            return all(c is None for c in node.links)
        ch = word[depth]
        if not node.containsKey(ch):
            return False
        should_delete = _delete(node.get(ch), word, depth + 1)
        if should_delete:
            node.links[ord(ch) - 97] = None
            return not node.isEnd() and all(c is None for c in node.links)
        return False
    _delete(self.root, word, 0)
```

---

### 🟡 Design Add and Search Words Data Structure — Medium (LC 211)

**Intuition:** Same as basic Trie, but search supports `.` wildcard which matches any single character. When you hit a `.`, try ALL 26 children recursively.

**Key insight:** Wildcard `.` triggers a DFS over all 26 children. Everything else is a standard Trie walk.

```python
def search(self, word: str) -> bool:
    def dfs(node, i):
        if i == len(word):
            return node.isEnd()
        ch = word[i]
        if ch == '.':
            for child in node.links:
                if child and dfs(child, i + 1):
                    return True
            return False
        else:
            if not node.containsKey(ch):
                return False
            return dfs(node.get(ch), i + 1)
    return dfs(self.root, 0)
```

---

### 🟡 Map Sum Pairs — Medium (LC 677)

**Intuition:** Store `(key → val)` pairs in a Trie. For `sum(prefix)`, traverse to the prefix node then DFS the subtree summing all `isEnd` values. Replace the boolean flag with an integer value.

**Key insight:** Replace `flag` with an integer `val`. `sum(prefix)` = DFS total of all vals in subtree rooted at prefix.

```python
class MapSum:
    def __init__(self):
        self.root = {}

    def insert(self, key, val):
        node = self.root
        for ch in key:
            node = node.setdefault(ch, {})
        node['#'] = val  # store val at end

    def sum(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node:
                return 0
            node = node[ch]

        def dfs(n):
            total = n.get('#', 0)
            for k, v in n.items():
                if k != '#':
                    total += dfs(v)
            return total
        return dfs(node)
```

---

## 2. Trie with Bit Manipulation

> **Core idea:** Build a binary Trie (depth = 31 or 32) inserting each number bit by bit from MSB to LSB. To maximize XOR, at each bit greedily go to the **opposite** child if it exists.

---

### 🔴 Maximum XOR of Two Numbers in an Array — Hard (LC 421)

**Intuition:** Insert all numbers into a binary Trie. For each number, greedily query: at each bit, try to go to the bit that gives XOR = 1 (the opposite bit). This greedy works because higher bits contribute more to the XOR value.

**Key insight:** XOR is maximized greedily bit-by-bit from MSB. If we want XOR = 1 at a bit, we need the opposite bit in the Trie.

```python
class TrieNode:
    def __init__(self):
        self.children = {}

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, num):
        node = self.root
        for i in range(31, -1, -1):  # MSB first
            bit = (num >> i) & 1
            if bit not in node.children:
                node.children[bit] = TrieNode()
            node = node.children[bit]

    def max_xor(self, num):
        node = self.root
        xor = 0
        for i in range(31, -1, -1):
            bit = (num >> i) & 1
            want = 1 - bit  # opposite bit → XOR = 1
            if want in node.children:
                xor |= (1 << i)
                node = node.children[want]
            else:
                node = node.children[bit]
        return xor

def findMaximumXOR(nums):
    trie = Trie()
    for n in nums:
        trie.insert(n)
    return max(trie.max_xor(n) for n in nums)
```

---

### 🔴 Minimum XOR Value Pair — Hard

**Intuition:** Opposite of max XOR — at each bit level, try to go to the **same** bit (XOR = 0). Only fall back to opposite if same doesn't exist.

**Key insight:** Greedy same direction: prefer same bit child to keep XOR minimal.

```python
def min_xor(self, num):
    node = self.root
    xor = 0
    for i in range(31, -1, -1):
        bit = (num >> i) & 1
        want = bit  # same bit → XOR = 0
        if want in node.children:
            node = node.children[want]
        else:
            xor |= (1 << i)  # forced XOR = 1 here
            node = node.children[1 - bit]
    return xor
```

---

### 🔴 Maximum XOR With an Element From Array — Hard (LC 1707)

**Intuition:** Offline queries. Sort queries by limit, sort array too. Process queries in order: add array elements ≤ limit to Trie, then query max XOR. Elements added later don't violate the constraint.

**Key insight:** Offline sort trick — sort both queries and array. Incrementally add elements to Trie as the limit grows.

```python
def maximizeXor(nums, queries):
    nums.sort()
    indexed_q = sorted(enumerate(queries), key=lambda x: x[1][1])
    trie = Trie()
    ans = [-1] * len(queries)
    j = 0
    for qi, (x, limit) in indexed_q:
        while j < len(nums) and nums[j] <= limit:
            trie.insert(nums[j])
            j += 1
        if j > 0:
            ans[qi] = trie.max_xor(x)
    return ans
```

---

### 🔴 Count Pairs With XOR in a Range — Hard (LC 1803)

**Intuition:** For each number, query "how many numbers already in Trie give XOR in [low, high]?" = `count(XOR < high+1) - count(XOR < low)`. Store subtree counts in each node.

**Key insight:** Store subtree count in each node. Range XOR count = `prefix_count(high+1) - prefix_count(low)`.

```python
class TrieNode:
    def __init__(self):
        self.children = [None, None]
        self.cnt = 0  # count of elements in subtree

def count_less(root, num, limit):
    node = root
    result = 0
    for i in range(14, -1, -1):
        nb = (num >> i) & 1
        lb = (limit >> i) & 1
        if lb == 1:
            if node.children[nb]:
                result += node.children[nb].cnt
            if not node.children[1 - nb]:
                break
            node = node.children[1 - nb]
        else:
            if not node.children[nb]:
                break
            node = node.children[nb]
    return result
```

---

### 🔴 Maximum Strong Pair XOR II — Hard (LC 2935)

**Intuition:** Strong pair: `|x - y| ≤ min(x, y)` → `x/2 ≤ y ≤ 2x`. Use a sliding window on the sorted array + binary Trie. As `y` increases, evict from Trie elements where `x < y/2`.

**Key insight:** Sliding window + binary Trie. Keep only valid partners in Trie. Evict when `nums[left] * 2 < y`.

```python
def maximumStrongPairXor(nums):
    nums.sort()
    trie = Trie()  # with insert/delete support
    ans = 0
    left = 0
    for right, y in enumerate(nums):
        trie.insert(y)
        while nums[left] * 2 < y:
            trie.delete(nums[left])
            left += 1
        ans = max(ans, trie.max_xor(y))
    return ans
```

---

## 3. Trie Involving Strings

---

### 🟡 Longest Common Prefix — Medium (LC 14)

**Intuition:** Insert all strings into Trie. The LCP is the longest path from root where each node has exactly 1 child and no node is an end-of-word. Stop at first branch or end-of-word.

**Key insight:** Walk down while exactly 1 child AND not `isEnd`. The depth gives LCP length.

```python
def longestCommonPrefix(strs):
    trie = Trie()
    for s in strs:
        trie.insert(s)
    prefix = []
    node = trie.root
    for ch in strs[0]:
        children = [c for c in node.links if c]
        if len(children) != 1 or node.isEnd():
            break
        prefix.append(ch)
        node = node.get(ch)
    return ''.join(prefix)
```

---

### 🟡 Find the Length of the Longest Common Prefix — Medium (LC 3043)

**Intuition:** Insert all arr1 numbers (as strings) into Trie. For each arr2 number, walk the Trie as deep as possible — the depth is the LCP length with the best match in arr1.

**Key insight:** Insert arr1, query each arr2 element: walk until mismatch, depth = LCP length.

```python
def longestCommonPrefix(arr1, arr2):
    trie = Trie()
    for n in arr1:
        trie.insert(str(n))
    ans = 0
    for n in arr2:
        node = trie.root
        depth = 0
        for ch in str(n):
            if not node.containsKey(ch):
                break
            node = node.get(ch)
            depth += 1
        ans = max(ans, depth)
    return ans
```

---

### 🟡 Search Suggestions System — Medium (LC 1268)

**Intuition:** Sort products. For each prefix of `searchWord`, find all words with that prefix and return the first 3. Store up to 3 suggestions at each Trie node during insertion.

**Key insight:** At each node store up to 3 lexicographically smallest suggestions. Since products are sorted, the first 3 inserted are the correct ones.

```python
def suggestedProducts(products, searchWord):
    products.sort()
    trie = {}
    for p in products:
        node = trie
        for ch in p:
            node = node.setdefault(ch, {})
            node.setdefault('$sug', [])
            if len(node['$sug']) < 3:
                node['$sug'].append(p)
    result, node = [], trie
    for ch in searchWord:
        if node and ch in node:
            node = node[ch]
            result.append(node.get('$sug', []))
        else:
            node = None
            result.append([])
    return result
```

---

### 🔴 Sum of Prefix Scores of Strings — Hard (LC 2416)

**Intuition:** Each node stores a `passCount` — how many words passed through it during insert. The score of a word = sum of `passCount` values along its path from root to end.

**Key insight:** Store `passCount` (not just a flag) at each node. Score = sum of `passCount` on the path.

```python
class Node:
    def __init__(self):
        self.links = [None] * 26
        self.cnt = 0   # words passing through here
        self.flag = False

def insert(root, word):
    node = root
    for ch in word:
        if not node.links[ord(ch) - 97]:
            node.links[ord(ch) - 97] = Node()
        node = node.links[ord(ch) - 97]
        node.cnt += 1
    node.flag = True

def score(root, word):
    node, total = root, 0
    for ch in word:
        node = node.links[ord(ch) - 97]
        total += node.cnt
    return total
```

---

### 🔴 Prefix and Suffix Search — Hard (LC 745)

**Intuition:** For a word like `"apple"`, insert all `(suffix + '#' + word)` pairs: `"apple#apple"`, `"pple#apple"`, etc. Then `f(pref, suff)` = lookup `suff + '#' + pref` in the Trie.

**Key insight:** Wrap with suffix: insert `suff + "#" + word` for every suffix. Query: lookup `suff + "#" + pref`, return stored index.

```python
class WordFilter:
    def __init__(self, words):
        self.trie = {}
        for idx, word in enumerate(words):
            for i in range(len(word) + 1):
                key = word[i:] + '#' + word
                node = self.trie
                for ch in key:
                    node = node.setdefault(ch, {})
                    node['*'] = idx  # store latest (highest) index

    def f(self, pref, suff):
        node = self.trie
        for ch in suff + '#' + pref:
            if ch not in node:
                return -1
            node = node[ch]
        return node.get('*', -1)
```

---

### 🔴 Stream of Characters — Hard (LC 1032)

**Intuition:** Build a Trie of **reversed** words. Maintain a list of "active pointers" — Trie nodes currently being tracked. Each new character advances all active pointers. If any pointer reaches an `isEnd` node, a word matched.

**Key insight:** Reversed Trie + active pointer list. New char → advance all pointers + add root. Check if any reached `isEnd`.

```python
class StreamChecker:
    def __init__(self, words):
        self.root = Node()
        for w in words:
            self._insert(w[::-1])  # insert reversed
        self.actives = []

    def _insert(self, word):
        node = self.root
        for ch in word:
            if not node.containsKey(ch):
                node.put(ch, Node())
            node = node.get(ch)
        node.setEnd()

    def query(self, letter):
        new_actives = []
        result = False
        for node in self.actives:
            if node.containsKey(letter):
                nxt = node.get(letter)
                new_actives.append(nxt)
                if nxt.isEnd():
                    result = True
        if self.root.containsKey(letter):
            nxt = self.root.get(letter)
            new_actives.append(nxt)
            if nxt.isEnd():
                result = True
        self.actives = new_actives
        return result
```

---

### 🟡 Extra Characters in a String — Medium (LC 2707)

**Intuition:** DP + Trie. `dp[i]` = min extra chars in `s[0..i-1]`. At each position `i`, either skip the char (`dp[i+1] = dp[i] + 1`) or find a dictionary word starting at `i` using Trie.

**Key insight:** `dp[i] = min(dp[i+1] + 1, dp[i+len] for each word in Trie starting at i)`.

```python
def minExtraChar(s, dictionary):
    trie = Trie()
    for w in dictionary:
        trie.insert(w)
    n = len(s)
    dp = [0] * (n + 1)
    for i in range(n - 1, -1, -1):
        dp[i] = dp[i + 1] + 1  # skip s[i]
        node = trie.root
        for j in range(i, n):
            if not node.containsKey(s[j]):
                break
            node = node.get(s[j])
            if node.isEnd():
                dp[i] = min(dp[i], dp[j + 1])
    return dp[0]
```

---

### 🟡 Implement Magic Dictionary — Medium (LC 676)

**Intuition:** Walk character by character with a `replaced` boolean. At each node, try ALL 26 characters — if it doesn't match the query char, set `replaced = True`. Allow exactly 1 mismatch total.

**Key insight:** DFS with a `used_replacement` boolean. Allow exactly one character swap during search.

```python
def search(self, searchWord):
    def dfs(node, i, replaced):
        if i == len(searchWord):
            return replaced and node.isEnd()
        ch = searchWord[i]
        for c in 'abcdefghijklmnopqrstuvwxyz':
            if node.containsKey(c):
                new_replaced = replaced or (c != ch)
                if not (new_replaced and replaced):  # at most one swap
                    if dfs(node.get(c), i + 1, new_replaced):
                        return True
        return False
    return dfs(self.root, 0, False)
```

---

### 🟡 Short Encoding of Words — Medium (LC 820)

**Intuition:** A word doesn't need its own slot if it's a suffix of another word. Build a Trie of **reversed** words. Words that are NOT suffixes of others correspond to leaf nodes. Sum their `(len + 1)` for the `'#'` separator.

**Key insight:** Reversed Trie → leaf nodes = words that are not suffixes of any other word. Answer = sum of `(len + 1)` for each leaf.

---

### 🟡 Longest Word in Dictionary — Medium (LC 720)

**Intuition:** Insert all words. A word qualifies if every prefix of it is also in the dictionary (every node on its path is `isEnd`). BFS/DFS from root following only `isEnd` nodes, tracking the deepest word found.

**Key insight:** BFS from root following only `isEnd` nodes. Deepest reachable node = longest valid word.

---

### 🔴 Construct String With Minimum Cost — Hard

**Intuition:** DP + Trie on source string. `dp[i]` = min cost to construct `target[0..i-1]`. For each position `i`, walk the Trie matching `target[i..]` and update `dp[i + len]` with the word's cost.

**Key insight:** Same pattern as Word Break DP but with costs. Walk Trie from each position, update `dp[j+1] = min(dp[j+1], dp[i] + cost)`.

---

### 🔴 Encrypt and Decrypt Strings — Hard (LC 2227)

**Intuition:** Encrypt is straightforward character substitution. For decrypt, precompute all possible decrypted words using DFS on the substitution map, then count how many exist in the dictionary. Build a Trie/hashmap of the dictionary for fast lookup.

**Key insight:** Precompute decrypt by DFS over all character substitution paths; count valid words in dictionary.

---

### 🟡 Number of Matching Subsequences — Medium (LC 792)

**Intuition:** Instead of checking each word against `s` separately (O(n × |s|)), group words by their current expected character. Use a bucket/pointer approach: advance pointers only when the matching character appears in `s`.

**Key insight:** Bucket words by next expected character. For each char in `s`, advance all matching buckets. Completed words (pointer past end) → count.

---

### 🟡 Camelcase Matching — Medium (LC 1023)

**Intuition:** A query matches a pattern if you can insert lowercase letters into the pattern to get the query. Walk both strings: uppercase letters in query must match exactly; lowercase can be skipped in pattern.

**Key insight:** Two-pointer: uppercase must match, lowercase in query must appear in pattern in order (extras allowed only if lowercase).

---

## 4. Trie Involving Recursion

---

### 🟡 Word Break — Medium (LC 139)

**Intuition:** DP + Trie. `dp[i] = True` if `s[0..i-1]` can be segmented. For each `True dp[i]`, use Trie to find all words starting at `i`, mark `dp[i + len(word)] = True`.

**Key insight:** `dp[i] = True` → walk Trie from `s[i..]`, set `dp[j+1] = True` for each word found ending at `j`.

```python
def wordBreak(s, wordDict):
    trie = Trie()
    for w in wordDict:
        trie.insert(w)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(n):
        if not dp[i]:
            continue
        node = trie.root
        for j in range(i, n):
            if not node.containsKey(s[j]):
                break
            node = node.get(s[j])
            if node.isEnd():
                dp[j + 1] = True
    return dp[n]
```

---

### 🔴 Word Break II — Hard (LC 140)

**Intuition:** Same as Word Break but collect all paths. DFS + memoization. At each index, try all words in Trie starting here, recurse on the remainder, combine results.

**Key insight:** `memo[i]` = all valid sentences starting at index `i`. Combine `word + ' ' + each result from memo[i + len]`.

```python
def wordBreak(s, wordDict):
    trie = Trie()
    for w in wordDict:
        trie.insert(w)
    from functools import lru_cache

    @lru_cache(None)
    def dfs(start):
        if start == len(s):
            return ['']
        result = []
        node = trie.root
        for end in range(start, len(s)):
            ch = s[end]
            if not node.containsKey(ch):
                break
            node = node.get(ch)
            if node.isEnd():
                word = s[start:end + 1]
                for rest in dfs(end + 1):
                    result.append(word + (' ' + rest if rest else ''))
        return result
    return dfs(0)
```

---

### 🔴 Word Search II — Hard (LC 212)

**Intuition:** Build Trie of all words. DFS on the board — at each cell, walk the Trie simultaneously. When we reach an `isEnd` node, we've found a word. Prune when Trie node has no children — this makes it far faster than searching for each word separately.

**Key insight:** Simultaneous board DFS + Trie walk. `isEnd` → found word. Prune when Trie node is empty. Remove found words from Trie to avoid duplicates.

```python
def findWords(board, words):
    trie = Trie()
    for w in words:
        trie.insert(w)
    R, C = len(board), len(board[0])
    result = []

    def dfs(r, c, node, path):
        ch = board[r][c]
        if not node.containsKey(ch):
            return
        node = node.get(ch)
        path += ch
        if node.isEnd():
            result.append(path)
            node.flag = False  # avoid duplicates
        board[r][c] = '#'      # mark visited
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and board[nr][nc] != '#':
                dfs(nr, nc, node, path)
        board[r][c] = ch       # restore

    for r in range(R):
        for c in range(C):
            dfs(r, c, trie.root, '')
    return result
```

---

### 🔴 Palindrome Pairs — Hard (LC 336)

**Intuition:** For each word `w`, insert reversed words into Trie. Check: (1) full word reversed exists; (2) some prefix of `w` reversed exists AND remaining suffix is a palindrome; (3) some suffix reversed exists AND remaining prefix is a palindrome.

**Key insight:** Insert reversed words. For each word, check palindrome condition at every split point. Store word index at Trie end-nodes.

---

### 🟡 Shortest Uncommon Substring in an Array — Medium (LC 3076)

**Intuition:** For each string `arr[i]`, find the shortest substring that doesn't appear in any other `arr[j]`. Build Trie of all substrings of the other strings, then find the shortest substring of `arr[i]` not present.

**Key insight:** Build Trie of all substrings excluding `arr[i]`. Query shortest substring of `arr[i]` not found in Trie.

---

## 5. Trie Involving File System

> File paths are naturally Trie-shaped: `/` separates components, each component is a Trie node.

---

### 🔴 Remove Sub-Folders from the Filesystem — Hard (LC 1233)

**Intuition:** Sort paths lexicographically. A sub-folder always comes after its parent. Keep a path only if it doesn't start with `last_kept + '/'`.

**Key insight:** Sort + greedy. Keep a path only if it doesn't start with `last_kept + '/'`. O(n log n).

```python
# Greedy approach
def removeSubfolders(folder):
    folder.sort()
    result = []
    for path in folder:
        if not result or not path.startswith(result[-1] + '/'):
            result.append(path)
    return result

# Trie approach
def removeSubfoldersTrie(folder):
    trie = {}
    for path in sorted(folder):
        parts = path.split('/')[1:]
        node = trie
        is_sub = False
        for part in parts:
            if '#end' in node:
                is_sub = True
                break
            node = node.setdefault(part, {})
        if not is_sub:
            node.clear()
            node['#end'] = True
```

---

### 🔴 Delete Duplicate Folders in System — Hard (LC 1948)

**Intuition:** Build a Trie from all paths. Serialize each subtree into a canonical string (e.g. `"(a(b)(c))"`). Use a hashmap: `serialization → count`. If count > 1, all those nodes are duplicates — mark and skip them during reconstruction.

**Key insight:** Post-order serialize each subtree. Nodes with the same serialization are duplicates. Mark and prune all duplicates.

```python
def deleteDuplicateFolder(paths):
    # 1. Build Trie
    trie = {}
    for path in paths:
        node = trie
        for part in path:
            node = node.setdefault(part, {})

    # 2. Serialize each subtree, count occurrences
    from collections import defaultdict
    count = defaultdict(int)

    def serialize(node):
        if not node:
            return ''
        parts = []
        for key in sorted(node.keys()):
            parts.append(f'({key}{serialize(node[key])})')
        serial = ''.join(parts)
        if serial:
            count[serial] += 1
        return serial

    serialize(trie)

    # 3. Collect non-duplicate paths
    result = []

    def collect(node, path):
        for key in sorted(node.keys()):
            child = node[key]
            serial = serialize(child)
            if serial and count[serial] > 1:
                continue  # skip duplicate subtree
            result.append(path + [key])
            collect(child, path + [key])

    collect(trie, [])
    return result
```

---

## 🧠 Pattern Summary

| Pattern | Key Idea |
|---|---|
| **Basic Trie** | 26-link node + flag; insert/search/startsWith |
| **Wildcard search** | `.` → try all 26 children (DFS) |
| **Pass-count node** | Increment counter on insert; query sums path |
| **Binary Trie (XOR)** | Depth=32, greedy opposite bit = max XOR |
| **Offline + Binary Trie** | Sort queries + array; incrementally add to Trie |
| **Reversed Trie** | Insert reversed words → suffix matching becomes prefix matching |
| **Suffix-wrap Trie** | Insert `suff + '#' + word` → handle prefix+suffix queries |
| **Active pointer list** | Maintain set of Trie nodes; advance all on each character |
| **DP + Trie** | Walk Trie at each DP position instead of set lookup |
| **Board DFS + Trie** | Simultaneous board + Trie traversal; prune on empty subtree |
| **Subtree serialization** | Post-order canonical string → detect duplicate subtrees |
