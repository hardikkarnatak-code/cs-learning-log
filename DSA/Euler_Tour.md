# Euler Tour Technique

## Why Do We Need Euler Tour?

Suppose we have a tree with `n` nodes and `q` queries.

The queries can be of different types, for example:

```text
1 → Update the value of a particular node
2 → Find the sum of values in a subtree
```

We might also have queries such as:

```text
- Find maximum in a subtree
- Find minimum in a subtree
- Find some other aggregate value
```

A naive approach could require traversing a whole subtree for every query.

In the worst case, this can take approximately:

```text
O(q * n)
```

which can become too slow when `n` and `q` are large.

---

## The Main Idea

Data structures such as:

* Segment Tree
* Fenwick Tree (BIT)

are designed to work efficiently on **arrays/ranges**.

For example, a Segment Tree can perform:

```text
Point Update  → O(log n)
Range Query   → O(log n)
```

But our original data structure is a **tree**, not an array.

So the question is:

> Can we convert the tree into an array in such a way that a subtree becomes a continuous range?

Yes.

This is where the **Euler Tour technique** is useful.

---

# Euler Tour

Euler Tour is a technique based on **DFS (Depth First Search)** that can flatten a tree into an array.

The important observation is:

> All nodes belonging to the subtree of a node appear consecutively in the DFS ordering.

Therefore:

```text
Tree
 ↓
DFS / Euler Tour
 ↓
Flattened Array
 ↓
Subtree becomes a continuous range
```

Once this conversion is done, we can use Segment Trees or Fenwick Trees to efficiently process subtree queries.

---

# Entry and Exit Time

During DFS, we maintain a timer.

Whenever we first visit a node:

```cpp
in[node] = timer;
flat_tree[timer] = node;
timer++;
```

After completely processing the subtree of that node:

```cpp
out[node] = timer;
```

Therefore, the subtree of `node` occupies the range:

```text
[in[node], out[node] - 1]
```

### Why?

Suppose the flattened array is:

```text
1 2 4 5 3
```

For node `2`:

```text
subtree(2) = {2, 4, 5}
```

and these nodes appear continuously:

```text
      ↓─────↓
1 | 2 | 4 | 5 | 3
    └───────┘
```

So we can represent the subtree of `2` as an array range.

---

# What Is `flat_tree`?

`flat_tree` stores the nodes in their DFS order.

For example:

```text
flat_tree = [1, 2, 4, 5, 3]
```

If:

```text
in[2] = 1
out[2] = 4
```

then:

```text
[in[2], out[2] - 1]
= [1, 3]
```

corresponds to:

```text
flat_tree[1 ... 3]
= [2, 4, 5]
```

which is exactly the subtree of node `2`.

---

# Basic Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

ll timer = 0;

void dfs(
    ll node,
    ll parent,
    vector<vector<ll>> &adj,
    vector<ll> &in,
    vector<ll> &out,
    vector<ll> &flat_tree
)
{
    // Time when we enter the node
    in[node] = timer;

    // Store node in flattened array
    flat_tree[timer] = node;

    timer++;

    for (auto child : adj[node])
    {
        if (child == parent)
            continue;

        dfs(child, node, adj, in, out, flat_tree);
    }

    // Time after completely processing the subtree
    out[node] = timer;
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    ll n;
    cin >> n;

    vector<vector<ll>> adj(n + 1);

    for (int i = 0; i < n - 1; i++)
    {
        ll u, v;
        cin >> u >> v;

        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    vector<ll> in(n + 1);
    vector<ll> out(n + 1);
    vector<ll> flat_tree(n);

    dfs(1, -1, adj, in, out, flat_tree);

    return 0;
}
```

---

# Time Complexity

DFS visits every vertex and edge once.

For a tree:

```text
Number of vertices = n
Number of edges = n - 1
```

Therefore:

```text
Time Complexity = O(n)
Space Complexity = O(n)
```

The Euler Tour itself is **not** `O(n log n)`.

The `O(log n)` complexity comes later when we use a data structure such as a Segment Tree.

---

# Euler Tour + Segment Tree

This is the important combination.

After flattening:

```text
Tree
 ↓
Euler Tour
 ↓
Array
 ↓
Segment Tree
```

Then:

### Node Update

Updating node `x` means updating:

```text
position = in[x]
```

So it becomes a **point update**:

```text
O(log n)
```

### Subtree Query

Querying the subtree of `x` becomes:

```text
[in[x], out[x] - 1]
```

So it becomes a **range query**:

```text
O(log n)
```

Therefore, after preprocessing:

```text
Euler Tour       → O(n)
Segment Tree     → O(n)
Each update      → O(log n)
Each query       → O(log n)
```

For `q` queries, the overall complexity is approximately:

```text
O(n + q log n)
```

instead of potentially:

```text
O(qn)
```

---

# Key Takeaway

The most important idea I learned is:

> **Euler Tour converts a tree subtree into a continuous range of an array.**

This allows us to apply powerful array data structures such as:

```text
Euler Tour + Segment Tree
Euler Tour + Fenwick Tree
```

to solve subtree problems efficiently.

The complete pattern is:

```text
Tree
   ↓
DFS
   ↓
Euler Tour
   ↓
Flattened Array
   ↓
Subtree → Continuous Range
   ↓
Segment Tree / Fenwick Tree
   ↓
Efficient Queries
```

---

# Problem Where I Applied This

**CSES — Subtree Queries**

I used:

```text
DFS
+
Euler Tour
+
Segment Tree
```

to support:

```text
Point Update
Subtree Sum Query
```

This was my first practical application of Euler Tour.
