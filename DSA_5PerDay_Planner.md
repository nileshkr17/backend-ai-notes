# DSA 5-Per-Day Interview Planner
### Target: 15 LPA – 24 LPA | JPMorgan · Salesforce · Microsoft · Amazon · JusPay + more
### Format: 5 problems/day · Topic-batched · Company-tagged · LeetCode linked

---

## How to Use This Planner

- Do **5 problems per day** in order — don't skip ahead
- Each day = 1–2 topics batched together (spaced repetition)
- Mark each problem: `[ ]` pending → `[~]` attempted → `[x]` solved → `[!]` needs review
- **Week 1–2:** Arrays, Strings, Hashing — foundation for every company
- **Week 3–4:** Trees, Graphs — Amazon, Microsoft favourites
- **Week 5–6:** DP, Sliding Window — JusPay, Salesforce, JPMorgan
- **Week 7–8:** Heaps, Tries, Advanced — FAANG-level finishers

**Time per problem:** 25–30 min max. If stuck after 15 min — read approach, code from understanding.

---

## Company Tag Legend

| Tag | Company |
|---|---|
| `[AMZN]` | Amazon |
| `[MSFT]` | Microsoft |
| `[SFDC]` | Salesforce |
| `[JPM]` | JPMorgan Chase |
| `[JP]` | JusPay |
| `[FLIP]` | Flipkart |
| `[GOOG]` | Google |
| `[META]` | Meta |

---

## WEEK 1 — Arrays & Hashing (Foundation)

---

### Day 1 — Array Basics + Two Sum Family

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Two Sum | Easy | [LC 1](https://leetcode.com/problems/two-sum/) | `[AMZN]` `[MSFT]` `[JP]` |
| 2 | Best Time to Buy and Sell Stock | Easy | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | `[AMZN]` `[JPM]` `[FLIP]` |
| 3 | Contains Duplicate | Easy | [LC 217](https://leetcode.com/problems/contains-duplicate/) | `[AMZN]` `[SFDC]` |
| 4 | Product of Array Except Self | Medium | [LC 238](https://leetcode.com/problems/product-of-array-except-self/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 5 | Maximum Subarray (Kadane's) | Medium | [LC 53](https://leetcode.com/problems/maximum-subarray/) | `[AMZN]` `[JPM]` `[SFDC]` |

**Pattern focus:** HashMap for O(1) lookup, prefix products, Kadane's algorithm.

---

### Day 2 — Array Manipulation + Intervals

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Merge Intervals | Medium | [LC 56](https://leetcode.com/problems/merge-intervals/) | `[AMZN]` `[MSFT]` `[JPM]` `[SFDC]` |
| 2 | Insert Interval | Medium | [LC 57](https://leetcode.com/problems/insert-interval/) | `[GOOG]` `[MSFT]` |
| 3 | 3Sum | Medium | [LC 15](https://leetcode.com/problems/3sum/) | `[AMZN]` `[MSFT]` `[JP]` |
| 4 | Container With Most Water | Medium | [LC 11](https://leetcode.com/problems/container-with-most-water/) | `[AMZN]` `[GOOG]` |
| 5 | Rotate Array | Medium | [LC 189](https://leetcode.com/problems/rotate-array/) | `[MSFT]` `[JPM]` |

**Pattern focus:** Sort + two-pointer, interval merging logic.

---

### Day 3 — Hashing + String Matching

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Valid Anagram | Easy | [LC 242](https://leetcode.com/problems/valid-anagram/) | `[AMZN]` `[JP]` `[SFDC]` |
| 2 | Group Anagrams | Medium | [LC 49](https://leetcode.com/problems/group-anagrams/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Top K Frequent Elements | Medium | [LC 347](https://leetcode.com/problems/top-k-frequent-elements/) | `[AMZN]` `[JPM]` `[FLIP]` |
| 4 | Longest Consecutive Sequence | Medium | [LC 128](https://leetcode.com/problems/longest-consecutive-sequence/) | `[AMZN]` `[GOOG]` |
| 5 | Subarray Sum Equals K | Medium | [LC 560](https://leetcode.com/problems/subarray-sum-equals-k/) | `[AMZN]` `[SFDC]` `[JPM]` |

**Pattern focus:** Frequency maps, prefix sum + hashmap for subarray problems.

---

### Day 4 — Strings

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Valid Palindrome | Easy | [LC 125](https://leetcode.com/problems/valid-palindrome/) | `[MSFT]` `[JP]` `[JPM]` |
| 2 | Longest Palindromic Substring | Medium | [LC 5](https://leetcode.com/problems/longest-palindromic-substring/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Longest Substring Without Repeating Characters | Medium | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |
| 4 | String to Integer (atoi) | Medium | [LC 8](https://leetcode.com/problems/string-to-integer-atoi/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 5 | Minimum Window Substring | Hard | [LC 76](https://leetcode.com/problems/minimum-window-substring/) | `[AMZN]` `[MSFT]` `[GOOG]` |

**Pattern focus:** Two-pointer, sliding window, expand-around-center.

---

### Day 5 — Binary Search

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Binary Search | Easy | [LC 704](https://leetcode.com/problems/binary-search/) | `[AMZN]` `[MSFT]` `[JP]` |
| 2 | Search in Rotated Sorted Array | Medium | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) | `[AMZN]` `[MSFT]` `[JPM]` `[JP]` |
| 3 | Find Minimum in Rotated Sorted Array | Medium | [LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | `[AMZN]` `[MSFT]` |
| 4 | Koko Eating Bananas | Medium | [LC 875](https://leetcode.com/problems/koko-eating-bananas/) | `[AMZN]` `[GOOG]` |
| 5 | Median of Two Sorted Arrays | Hard | [LC 4](https://leetcode.com/problems/median-of-two-sorted-arrays/) | `[AMZN]` `[MSFT]` `[JPM]` `[GOOG]` |

**Pattern focus:** Binary search on answer (not just index), conditions for left/right shift.

---

## WEEK 2 — Linked Lists + Stack/Queue

---

### Day 6 — Linked List Basics

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Reverse Linked List | Easy | [LC 206](https://leetcode.com/problems/reverse-linked-list/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |
| 2 | Merge Two Sorted Lists | Easy | [LC 21](https://leetcode.com/problems/merge-two-sorted-lists/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Linked List Cycle | Easy | [LC 141](https://leetcode.com/problems/linked-list-cycle/) | `[AMZN]` `[MSFT]` `[JP]` |
| 4 | Linked List Cycle II (find start) | Medium | [LC 142](https://leetcode.com/problems/linked-list-cycle-ii/) | `[AMZN]` `[MSFT]` |
| 5 | Remove Nth Node From End | Medium | [LC 19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | `[AMZN]` `[MSFT]` `[JPM]` |

**Pattern focus:** Fast/slow pointers (Floyd's), two-pass vs one-pass.

---

### Day 7 — Linked List Advanced

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Reorder List | Medium | [LC 143](https://leetcode.com/problems/reorder-list/) | `[AMZN]` `[MSFT]` |
| 2 | LRU Cache | Medium | [LC 146](https://leetcode.com/problems/lru-cache/) | `[AMZN]` `[MSFT]` `[JPM]` `[SFDC]` `[JP]` |
| 3 | Merge K Sorted Lists | Hard | [LC 23](https://leetcode.com/problems/merge-k-sorted-lists/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 4 | Copy List with Random Pointer | Medium | [LC 138](https://leetcode.com/problems/copy-list-with-random-pointer/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 5 | Reverse Nodes in k-Group | Hard | [LC 25](https://leetcode.com/problems/reverse-nodes-in-k-group/) | `[AMZN]` `[MSFT]` `[GOOG]` |

> **LRU Cache is the most important problem in this entire planner.** JPMorgan, JusPay, Amazon all ask it. Know the HashMap + DoublyLinkedList implementation cold.

---

### Day 8 — Stack

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Valid Parentheses | Easy | [LC 20](https://leetcode.com/problems/valid-parentheses/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Min Stack | Medium | [LC 155](https://leetcode.com/problems/min-stack/) | `[AMZN]` `[JPM]` `[SFDC]` |
| 3 | Daily Temperatures | Medium | [LC 739](https://leetcode.com/problems/daily-temperatures/) | `[AMZN]` `[SFDC]` |
| 4 | Largest Rectangle in Histogram | Hard | [LC 84](https://leetcode.com/problems/largest-rectangle-in-histogram/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 5 | Evaluate Reverse Polish Notation | Medium | [LC 150](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | `[AMZN]` `[JPM]` |

**Pattern focus:** Monotonic stack for next greater/smaller element problems.

---

### Day 9 — Queues + Design

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Implement Queue using Stacks | Easy | [LC 232](https://leetcode.com/problems/implement-queue-using-stacks/) | `[AMZN]` `[MSFT]` `[JP]` |
| 2 | Sliding Window Maximum | Hard | [LC 239](https://leetcode.com/problems/sliding-window-maximum/) | `[AMZN]` `[JPM]` `[GOOG]` |
| 3 | Design Hit Counter | Medium | [LC 362](https://leetcode.com/problems/design-hit-counter/) | `[AMZN]` `[JPM]` `[SFDC]` |
| 4 | Task Scheduler | Medium | [LC 621](https://leetcode.com/problems/task-scheduler/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 5 | Decode String | Medium | [LC 394](https://leetcode.com/problems/decode-string/) | `[AMZN]` `[MSFT]` `[GOOG]` |

**Pattern focus:** Monotonic deque for sliding window max, priority queues.

---

### Day 10 — Two Pointers + Sliding Window

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Move Zeroes | Easy | [LC 283](https://leetcode.com/problems/move-zeroes/) | `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Trapping Rain Water | Hard | [LC 42](https://leetcode.com/problems/trapping-rain-water/) | `[AMZN]` `[MSFT]` `[JPM]` `[GOOG]` |
| 3 | Permutation in String | Medium | [LC 567](https://leetcode.com/problems/permutation-in-string/) | `[AMZN]` `[MSFT]` |
| 4 | Longest Repeating Character Replacement | Medium | [LC 424](https://leetcode.com/problems/longest-repeating-character-replacement/) | `[AMZN]` `[MSFT]` `[JP]` |
| 5 | Fruit Into Baskets | Medium | [LC 904](https://leetcode.com/problems/fruit-into-baskets/) | `[AMZN]` `[GOOG]` |

**Pattern focus:** Variable-size sliding window with constraint tracking.

---

## WEEK 3 — Trees

---

### Day 11 — Binary Tree Traversals

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Invert Binary Tree | Easy | [LC 226](https://leetcode.com/problems/invert-binary-tree/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 2 | Maximum Depth of Binary Tree | Easy | [LC 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | `[AMZN]` `[JP]` `[SFDC]` |
| 3 | Level Order Traversal | Medium | [LC 102](https://leetcode.com/problems/binary-tree-level-order-traversal/) | `[AMZN]` `[MSFT]` `[JPM]` `[JP]` |
| 4 | Binary Tree Right Side View | Medium | [LC 199](https://leetcode.com/problems/binary-tree-right-side-view/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 5 | Count Good Nodes in Binary Tree | Medium | [LC 1448](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) | `[MSFT]` `[SFDC]` |

**Pattern focus:** BFS (level order), DFS (pre/in/post order), return type design in recursion.

---

### Day 12 — Binary Tree Properties

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Subtree of Another Tree | Easy | [LC 572](https://leetcode.com/problems/subtree-of-another-tree/) | `[AMZN]` `[MSFT]` |
| 2 | Diameter of Binary Tree | Easy | [LC 543](https://leetcode.com/problems/diameter-of-binary-tree/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Balanced Binary Tree | Easy | [LC 110](https://leetcode.com/problems/balanced-binary-tree/) | `[AMZN]` `[JP]` `[JPM]` |
| 4 | Binary Tree Maximum Path Sum | Hard | [LC 124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 5 | Serialize and Deserialize Binary Tree | Hard | [LC 297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |

**Pattern focus:** Post-order DFS where you return multiple values up the recursion.

---

### Day 13 — Binary Search Tree

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Validate Binary Search Tree | Medium | [LC 98](https://leetcode.com/problems/validate-binary-search-tree/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Lowest Common Ancestor of BST | Medium | [LC 235](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 3 | Lowest Common Ancestor (Binary Tree) | Medium | [LC 236](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |
| 4 | Kth Smallest Element in BST | Medium | [LC 230](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 5 | Construct BST from Preorder | Medium | [LC 1008](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/) | `[AMZN]` `[MSFT]` |

**Pattern focus:** BST property = inorder gives sorted order. LCA is asked very frequently by JPMorgan.

---

### Day 14 — Heaps & Priority Queues

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Kth Largest Element in Array | Medium | [LC 215](https://leetcode.com/problems/kth-largest-element-in-an-array/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | K Closest Points to Origin | Medium | [LC 973](https://leetcode.com/problems/k-closest-points-to-origin/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Find Median from Data Stream | Hard | [LC 295](https://leetcode.com/problems/find-median-from-data-stream/) | `[AMZN]` `[MSFT]` `[JPM]` `[GOOG]` |
| 4 | Top K Frequent Words | Medium | [LC 692](https://leetcode.com/problems/top-k-frequent-words/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JPM]` |
| 5 | Reorganize String | Medium | [LC 767](https://leetcode.com/problems/reorganize-string/) | `[AMZN]` `[MSFT]` |

**Pattern focus:** Min-heap of size K for top-K problems. Two-heap (max+min) for median.

---

### Day 15 — Tries

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Implement Trie (Prefix Tree) | Medium | [LC 208](https://leetcode.com/problems/implement-trie-prefix-tree/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |
| 2 | Design Add and Search Words | Medium | [LC 211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | `[AMZN]` `[MSFT]` |
| 3 | Word Search II | Hard | [LC 212](https://leetcode.com/problems/word-search-ii/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 4 | Search Suggestions System | Medium | [LC 1268](https://leetcode.com/problems/search-suggestions-system/) | `[AMZN]` `[SFDC]` |
| 5 | Replace Words | Medium | [LC 648](https://leetcode.com/problems/replace-words/) | `[SFDC]` `[MSFT]` |

**Pattern focus:** TrieNode with `children[26]` array vs HashMap. Trie beats prefix-substring search.

---

## WEEK 4 — Graphs

---

### Day 16 — Graph BFS/DFS

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Number of Islands | Medium | [LC 200](https://leetcode.com/problems/number-of-islands/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` `[JPM]` |
| 2 | Clone Graph | Medium | [LC 133](https://leetcode.com/problems/clone-graph/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Max Area of Island | Medium | [LC 695](https://leetcode.com/problems/max-area-of-island/) | `[AMZN]` `[MSFT]` |
| 4 | Pacific Atlantic Water Flow | Medium | [LC 417](https://leetcode.com/problems/pacific-atlantic-water-flow/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 5 | Surrounded Regions | Medium | [LC 130](https://leetcode.com/problems/surrounded-regions/) | `[AMZN]` `[MSFT]` |

**Pattern focus:** BFS/DFS flood-fill. Multi-source BFS from boundary inward.

---

### Day 17 — Graph Shortest Path

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Rotting Oranges | Medium | [LC 994](https://leetcode.com/problems/rotting-oranges/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Walls and Gates | Medium | [LC 286](https://leetcode.com/problems/walls-and-gates/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 3 | Network Delay Time (Dijkstra) | Medium | [LC 743](https://leetcode.com/problems/network-delay-time/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 4 | Cheapest Flights Within K Stops (Bellman-Ford) | Medium | [LC 787](https://leetcode.com/problems/find-cheapest-flights-within-k-stops/) | `[AMZN]` `[JPM]` |
| 5 | Swim in Rising Water | Hard | [LC 778](https://leetcode.com/problems/swim-in-rising-water/) | `[AMZN]` `[GOOG]` |

**Pattern focus:** Multi-source BFS, Dijkstra with min-heap, Bellman-Ford for negative weights.

---

### Day 18 — Topological Sort + Cycle Detection

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Course Schedule | Medium | [LC 207](https://leetcode.com/problems/course-schedule/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Course Schedule II | Medium | [LC 210](https://leetcode.com/problems/course-schedule-ii/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 3 | Alien Dictionary | Hard | [LC 269](https://leetcode.com/problems/alien-dictionary/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |
| 4 | Find Eventual Safe States | Medium | [LC 802](https://leetcode.com/problems/find-eventual-safe-states/) | `[AMZN]` `[MSFT]` |
| 5 | Parallel Courses III (Critical Path) | Hard | [LC 2050](https://leetcode.com/problems/parallel-courses-iii/) | `[AMZN]` `[JPM]` |

**Pattern focus:** Kahn's BFS (in-degree reduction) for topo sort. DFS with 3-color for cycle detection.

---

### Day 19 — Union Find (Disjoint Set)

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Number of Connected Components | Medium | [LC 323](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | `[MSFT]` `[JPM]` `[SFDC]` |
| 2 | Graph Valid Tree | Medium | [LC 261](https://leetcode.com/problems/graph-valid-tree/) | `[MSFT]` `[GOOG]` |
| 3 | Redundant Connection | Medium | [LC 684](https://leetcode.com/problems/redundant-connection/) | `[AMZN]` `[MSFT]` |
| 4 | Accounts Merge | Medium | [LC 721](https://leetcode.com/problems/accounts-merge/) | `[AMZN]` `[MSFT]` `[SFDC]` `[GOOG]` |
| 5 | Smallest String With Swaps | Medium | [LC 1202](https://leetcode.com/problems/smallest-string-with-swaps/) | `[AMZN]` `[MSFT]` |

**Pattern focus:** Union-Find with path compression + union by rank = O(α(n)) ≈ O(1).

---

### Day 20 — Graph Review + Matrix Problems

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Word Ladder | Hard | [LC 127](https://leetcode.com/problems/word-ladder/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 2 | 01 Matrix | Medium | [LC 542](https://leetcode.com/problems/01-matrix/) | `[AMZN]` `[JP]` `[SFDC]` |
| 3 | Shortest Path in Binary Matrix | Medium | [LC 1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 4 | As Far from Land as Possible | Medium | [LC 1162](https://leetcode.com/problems/as-far-from-land-as-possible/) | `[AMZN]` `[GOOG]` |
| 5 | Minimum Cost to Connect All Points (MST) | Medium | [LC 1584](https://leetcode.com/problems/min-cost-to-connect-all-points/) | `[AMZN]` `[JPM]` |

---

## WEEK 5 — Dynamic Programming (Part 1)

---

### Day 21 — DP Foundations (1D)

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Climbing Stairs | Easy | [LC 70](https://leetcode.com/problems/climbing-stairs/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |
| 2 | House Robber | Medium | [LC 198](https://leetcode.com/problems/house-robber/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 3 | House Robber II | Medium | [LC 213](https://leetcode.com/problems/house-robber-ii/) | `[AMZN]` `[MSFT]` |
| 4 | Longest Increasing Subsequence | Medium | [LC 300](https://leetcode.com/problems/longest-increasing-subsequence/) | `[AMZN]` `[MSFT]` `[JPM]` `[SFDC]` |
| 5 | Partition Equal Subset Sum | Medium | [LC 416](https://leetcode.com/problems/partition-equal-subset-sum/) | `[AMZN]` `[MSFT]` `[JP]` |

**Pattern focus:** Identify recurrence relation, memoize top-down or fill bottom-up.

---

### Day 22 — DP on Strings

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Longest Common Subsequence | Medium | [LC 1143](https://leetcode.com/problems/longest-common-subsequence/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JPM]` |
| 2 | Edit Distance | Hard | [LC 72](https://leetcode.com/problems/edit-distance/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |
| 3 | Distinct Subsequences | Hard | [LC 115](https://leetcode.com/problems/distinct-subsequences/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 4 | Palindromic Substrings | Medium | [LC 647](https://leetcode.com/problems/palindromic-substrings/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 5 | Word Break | Medium | [LC 139](https://leetcode.com/problems/word-break/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` `[SFDC]` |

**Pattern focus:** 2D DP table for string comparison. `dp[i][j]` = answer for `s1[0..i], s2[0..j]`.

---

### Day 23 — DP on Stocks + State Machine

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Best Time to Buy and Sell Stock II | Medium | [LC 122](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/) | `[AMZN]` `[JPM]` `[FLIP]` |
| 2 | Best Time to Buy and Sell Stock with Cooldown | Medium | [LC 309](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) | `[AMZN]` `[JPM]` |
| 3 | Best Time to Buy and Sell Stock with Fee | Medium | [LC 714](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) | `[AMZN]` `[JPM]` `[SFDC]` |
| 4 | Best Time to Buy and Sell Stock III (at most 2) | Hard | [LC 123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) | `[AMZN]` `[JPM]` |
| 5 | Best Time to Buy and Sell Stock IV (at most k) | Hard | [LC 188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) | `[AMZN]` `[JPM]` `[GOOG]` |

> Stock problems are **very frequent at JPMorgan** — they interview for trading/fintech roles. Know the entire series.

---

### Day 24 — DP 2D (Grid)

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Unique Paths | Medium | [LC 62](https://leetcode.com/problems/unique-paths/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JPM]` |
| 2 | Unique Paths II (with obstacles) | Medium | [LC 63](https://leetcode.com/problems/unique-paths-ii/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 3 | Minimum Path Sum | Medium | [LC 64](https://leetcode.com/problems/minimum-path-sum/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 4 | Dungeon Game | Hard | [LC 174](https://leetcode.com/problems/dungeon-game/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 5 | Maximal Square | Medium | [LC 221](https://leetcode.com/problems/maximal-square/) | `[AMZN]` `[MSFT]` `[JPM]` `[SFDC]` |

---

### Day 25 — DP Knapsack + Unbounded

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Coin Change | Medium | [LC 322](https://leetcode.com/problems/coin-change/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` `[SFDC]` |
| 2 | Coin Change II (count ways) | Medium | [LC 518](https://leetcode.com/problems/coin-change-ii/) | `[AMZN]` `[JPM]` `[JP]` |
| 3 | Target Sum | Medium | [LC 494](https://leetcode.com/problems/target-sum/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 4 | Last Stone Weight II | Medium | [LC 1049](https://leetcode.com/problems/last-stone-weight-ii/) | `[AMZN]` `[MSFT]` |
| 5 | Interleaving String | Hard | [LC 97](https://leetcode.com/problems/interleaving-string/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |

**Pattern focus:** 0/1 knapsack = include or exclude. Unbounded = can reuse same item. Know `dp[amount]` bottom-up.

---

## WEEK 6 — Backtracking + Recursion

---

### Day 26 — Combinations & Subsets

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Subsets | Medium | [LC 78](https://leetcode.com/problems/subsets/) | `[AMZN]` `[MSFT]` `[JP]` |
| 2 | Subsets II (with duplicates) | Medium | [LC 90](https://leetcode.com/problems/subsets-ii/) | `[AMZN]` `[MSFT]` |
| 3 | Combination Sum | Medium | [LC 39](https://leetcode.com/problems/combination-sum/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JP]` |
| 4 | Combination Sum II | Medium | [LC 40](https://leetcode.com/problems/combination-sum-ii/) | `[AMZN]` `[MSFT]` |
| 5 | Permutations | Medium | [LC 46](https://leetcode.com/problems/permutations/) | `[AMZN]` `[MSFT]` `[JPM]` |

**Pattern focus:** Decision tree — at each node, include or skip. Sort to handle duplicates.

---

### Day 27 — Backtracking Applied

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Letter Combinations of Phone Number | Medium | [LC 17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JPM]` |
| 2 | Word Search | Medium | [LC 79](https://leetcode.com/problems/word-search/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 3 | N-Queens | Hard | [LC 51](https://leetcode.com/problems/n-queens/) | `[AMZN]` `[MSFT]` `[GOOG]` |
| 4 | Sudoku Solver | Hard | [LC 37](https://leetcode.com/problems/sudoku-solver/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |
| 5 | Palindrome Partitioning | Medium | [LC 131](https://leetcode.com/problems/palindrome-partitioning/) | `[AMZN]` `[MSFT]` `[SFDC]` |

---

### Day 28 — Greedy

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Jump Game | Medium | [LC 55](https://leetcode.com/problems/jump-game/) | `[AMZN]` `[MSFT]` `[JP]` `[SFDC]` |
| 2 | Jump Game II | Medium | [LC 45](https://leetcode.com/problems/jump-game-ii/) | `[AMZN]` `[MSFT]` |
| 3 | Gas Station | Medium | [LC 134](https://leetcode.com/problems/gas-station/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 4 | Hand of Straights | Medium | [LC 846](https://leetcode.com/problems/hand-of-straights/) | `[AMZN]` `[SFDC]` |
| 5 | Minimum Number of Arrows to Burst Balloons | Medium | [LC 452](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | `[AMZN]` `[GOOG]` `[JPM]` |

**Pattern focus:** Greedy works when local optimum leads to global. Prove it with invariant, not just intuition.

---

## WEEK 7 — Math + Bit Manipulation

---

### Day 29 — Bit Manipulation

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Single Number | Easy | [LC 136](https://leetcode.com/problems/single-number/) | `[AMZN]` `[MSFT]` `[JP]` |
| 2 | Number of 1 Bits | Easy | [LC 191](https://leetcode.com/problems/number-of-1-bits/) | `[AMZN]` `[JP]` `[JPM]` |
| 3 | Counting Bits | Easy | [LC 338](https://leetcode.com/problems/counting-bits/) | `[AMZN]` `[MSFT]` `[SFDC]` |
| 4 | Reverse Bits | Easy | [LC 190](https://leetcode.com/problems/reverse-bits/) | `[AMZN]` `[JP]` |
| 5 | Sum of Two Integers (no + operator) | Medium | [LC 371](https://leetcode.com/problems/sum-of-two-integers/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |

---

### Day 30 — Math

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Reverse Integer | Medium | [LC 7](https://leetcode.com/problems/reverse-integer/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |
| 2 | Pow(x, n) | Medium | [LC 50](https://leetcode.com/problems/powx-n/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JP]` |
| 3 | Multiply Strings | Medium | [LC 43](https://leetcode.com/problems/multiply-strings/) | `[AMZN]` `[MSFT]` `[JPM]` |
| 4 | Happy Number | Easy | [LC 202](https://leetcode.com/problems/happy-number/) | `[AMZN]` `[MSFT]` `[JP]` |
| 5 | Missing Number | Easy | [LC 268](https://leetcode.com/problems/missing-number/) | `[AMZN]` `[MSFT]` `[JP]` `[JPM]` |

---

## WEEK 8 — Design Problems (High-Value for 15–24 LPA)

> These "design" LC problems are the most frequently asked at JPMorgan, JusPay, and Salesforce. They test data structure design — not system design.

---

### Day 31 — Design I

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | LRU Cache | Medium | [LC 146](https://leetcode.com/problems/lru-cache/) | `[AMZN]` `[MSFT]` `[JPM]` `[JP]` `[SFDC]` |
| 2 | LFU Cache | Hard | [LC 460](https://leetcode.com/problems/lfu-cache/) | `[AMZN]` `[MSFT]` `[JPM]` `[GOOG]` |
| 3 | Design HashMap | Easy | [LC 706](https://leetcode.com/problems/design-hashmap/) | `[AMZN]` `[JP]` `[JPM]` |
| 4 | Design HashSet | Easy | [LC 705](https://leetcode.com/problems/design-hashset/) | `[AMZN]` `[JP]` |
| 5 | Time Based Key-Value Store | Medium | [LC 981](https://leetcode.com/problems/time-based-key-value-store/) | `[AMZN]` `[MSFT]` `[GOOG]` `[JPM]` |

---

### Day 32 — Design II

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Design Twitter | Medium | [LC 355](https://leetcode.com/problems/design-twitter/) | `[AMZN]` `[SFDC]` `[JPM]` |
| 2 | Design Search Autocomplete | Hard | [LC 642](https://leetcode.com/problems/design-search-autocomplete-system/) | `[AMZN]` `[GOOG]` `[JPM]` |
| 3 | Design Browser History | Medium | [LC 1472](https://leetcode.com/problems/design-browser-history/) | `[AMZN]` `[SFDC]` |
| 4 | Design Underground System | Medium | [LC 1396](https://leetcode.com/problems/design-underground-system/) | `[AMZN]` `[SFDC]` `[JPM]` |
| 5 | Design Logger Rate Limiter | Easy | [LC 359](https://leetcode.com/problems/logger-rate-limiter/) | `[AMZN]` `[JP]` `[JPM]` `[SFDC]` |

---

### Day 33 — Design III (JusPay/Fintech Favourites)

| # | Problem | Difficulty | LC | Companies |
|---|---|---|---|---|
| 1 | Design a Stack With Increment Operation | Medium | [LC 1381](https://leetcode.com/problems/design-a-stack-with-increment-operation/) | `[JP]` `[JPM]` `[AMZN]` |
| 2 | Snapshot Array | Medium | [LC 1146](https://leetcode.com/problems/snapshot-array/) | `[JP]` `[AMZN]` `[GOOG]` |
| 3 | Maximum Frequency Stack | Hard | [LC 895](https://leetcode.com/problems/maximum-frequency-stack/) | `[AMZN]` `[GOOG]` `[JPM]` |
| 4 | All O(1) Data Structure | Hard | [LC 432](https://leetcode.com/problems/all-oone-data-structure/) | `[AMZN]` `[GOOG]` `[JPM]` |
| 5 | Insert Delete GetRandom O(1) | Medium | [LC 380](https://leetcode.com/problems/insert-delete-getrandom-o1/) | `[AMZN]` `[MSFT]` `[SFDC]` `[JP]` |

---

## WEEK 8 FINAL — Company-Specific Revision Days

---

### Day 34 — JPMorgan Focus

> JPMorgan pattern: stocks, intervals, LCA, edit distance, design problems.

| # | Problem | Difficulty | LC |
|---|---|---|---|
| 1 | Best Time to Buy and Sell Stock IV | Hard | [LC 188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) |
| 2 | Merge Intervals | Medium | [LC 56](https://leetcode.com/problems/merge-intervals/) |
| 3 | LCA of Binary Tree | Medium | [LC 236](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) |
| 4 | Edit Distance | Hard | [LC 72](https://leetcode.com/problems/edit-distance/) |
| 5 | Design Underground System | Medium | [LC 1396](https://leetcode.com/problems/design-underground-system/) |

---

### Day 35 — JusPay Focus

> JusPay pattern: graph problems, design, bit manipulation, system-level thinking.

| # | Problem | Difficulty | LC |
|---|---|---|---|
| 1 | Course Schedule | Medium | [LC 207](https://leetcode.com/problems/course-schedule/) |
| 2 | Number of Islands | Medium | [LC 200](https://leetcode.com/problems/number-of-islands/) |
| 3 | LRU Cache | Medium | [LC 146](https://leetcode.com/problems/lru-cache/) |
| 4 | Sum of Two Integers | Medium | [LC 371](https://leetcode.com/problems/sum-of-two-integers/) |
| 5 | Word Break | Medium | [LC 139](https://leetcode.com/problems/word-break/) |

---

### Day 36 — Salesforce Focus

> Salesforce pattern: strings, arrays, trees, data structure design.

| # | Problem | Difficulty | LC |
|---|---|---|---|
| 1 | Group Anagrams | Medium | [LC 49](https://leetcode.com/problems/group-anagrams/) |
| 2 | Task Scheduler | Medium | [LC 621](https://leetcode.com/problems/task-scheduler/) |
| 3 | Accounts Merge | Medium | [LC 721](https://leetcode.com/problems/accounts-merge/) |
| 4 | Design Logger Rate Limiter | Easy | [LC 359](https://leetcode.com/problems/logger-rate-limiter/) |
| 5 | Maximal Square | Medium | [LC 221](https://leetcode.com/problems/maximal-square/) |

---

### Day 37 — Microsoft Focus

> Microsoft pattern: trees, graphs, strings, recursion depth.

| # | Problem | Difficulty | LC |
|---|---|---|---|
| 1 | Serialize/Deserialize Binary Tree | Hard | [LC 297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) |
| 2 | Word Ladder | Hard | [LC 127](https://leetcode.com/problems/word-ladder/) |
| 3 | Merge K Sorted Lists | Hard | [LC 23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 4 | Alien Dictionary | Hard | [LC 269](https://leetcode.com/problems/alien-dictionary/) |
| 5 | N-Queens | Hard | [LC 51](https://leetcode.com/problems/n-queens/) |

---

### Day 38 — Amazon Focus

> Amazon focuses on Leadership Principles in interviews — every DSA answer connects to a story. Most asked: BFS/DFS, LRU, two-sum family, Kafka-like design.

| # | Problem | Difficulty | LC |
|---|---|---|---|
| 1 | K Closest Points to Origin | Medium | [LC 973](https://leetcode.com/problems/k-closest-points-to-origin/) |
| 2 | Trapping Rain Water | Hard | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 3 | Sliding Window Maximum | Hard | [LC 239](https://leetcode.com/problems/sliding-window-maximum/) |
| 4 | Find Median from Data Stream | Hard | [LC 295](https://leetcode.com/problems/find-median-from-data-stream/) |
| 5 | Word Search II | Hard | [LC 212](https://leetcode.com/problems/word-search-ii/) |

---

## Summary Stats

| Week | Topics | Problems |
|---|---|---|
| Week 1 | Arrays, Hashing, Strings, Binary Search | 25 |
| Week 2 | Linked Lists, Stack, Queue, Sliding Window | 25 |
| Week 3 | Trees, Heaps, Tries | 25 |
| Week 4 | Graphs (BFS/DFS, Topo Sort, Union Find) | 25 |
| Week 5 | Dynamic Programming (1D, 2D, Strings, Stocks) | 25 |
| Week 6 | Backtracking, Greedy | 15 |
| Week 7 | Bit Manipulation, Math | 10 |
| Week 8 | Design Problems + Company-Specific Revision | 40 |
| **Total** | **8 weeks · 38 days** | **~190 problems** |

---

## Difficulty Distribution

| Level | Count | % |
|---|---|---|
| Easy | ~35 | 18% |
| Medium | ~120 | 63% |
| Hard | ~35 | 19% |

---

## Top 20 Must-Solve (If Time Is Short)

> These 20 cover 80% of what 15–24 LPA interviews actually ask.

| # | Problem | LC |
|---|---|---|
| 1 | LRU Cache | [LC 146](https://leetcode.com/problems/lru-cache/) |
| 2 | Two Sum | [LC 1](https://leetcode.com/problems/two-sum/) |
| 3 | Number of Islands | [LC 200](https://leetcode.com/problems/number-of-islands/) |
| 4 | Merge Intervals | [LC 56](https://leetcode.com/problems/merge-intervals/) |
| 5 | Word Break | [LC 139](https://leetcode.com/problems/word-break/) |
| 6 | Coin Change | [LC 322](https://leetcode.com/problems/coin-change/) |
| 7 | Find Median from Data Stream | [LC 295](https://leetcode.com/problems/find-median-from-data-stream/) |
| 8 | Course Schedule | [LC 207](https://leetcode.com/problems/course-schedule/) |
| 9 | Longest Common Subsequence | [LC 1143](https://leetcode.com/problems/longest-common-subsequence/) |
| 10 | Trapping Rain Water | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 11 | Serialize/Deserialize Binary Tree | [LC 297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) |
| 12 | Sliding Window Maximum | [LC 239](https://leetcode.com/problems/sliding-window-maximum/) |
| 13 | Longest Substring Without Repeating Chars | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| 14 | Binary Tree Maximum Path Sum | [LC 124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) |
| 15 | Search in Rotated Sorted Array | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| 16 | Alien Dictionary | [LC 269](https://leetcode.com/problems/alien-dictionary/) |
| 17 | Implement Trie | [LC 208](https://leetcode.com/problems/implement-trie-prefix-tree/) |
| 18 | Edit Distance | [LC 72](https://leetcode.com/problems/edit-distance/) |
| 19 | Accounts Merge | [LC 721](https://leetcode.com/problems/accounts-merge/) |
| 20 | Best Time to Buy Stock III | [LC 123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) |
