# 🚀 Competitive Programming — Daily Progress

Today I solved **2 Codeforces problems** and learned two completely different approaches.

---

## 🧠 Problem 1 — F1. Pictures with Kittens

**Topic:** Dynamic Programming
**Rating:** 1900
**Contest:** Codeforces Round 521 (Div. 3)
**Version:** Easy (F1)

### Problem Idea

We have `n` pictures with beauty values `a[i]`.

We need to repost exactly `x` pictures such that **every segment of `k` consecutive pictures contains at least one reposted picture**.

The goal is to maximize the sum of the beauty values of the reposted pictures.

### My Thought Process

At first, I thought about using a greedy approach, but it was not obvious which picture should always be selected.

Since there are multiple possibilities of selecting or not selecting each picture, **DP** immediately came to my mind.

For the easy version:

```text
n <= 200
```

so an `O(n³)` solution is acceptable.

I decided to keep three state variables.

### DP State

```text
dp[i][j][x]
```

where:

* `i` = current picture we are processing
* `j` = position of the last reposted picture
* `x` = number of pictures still left to repost

The important observation is:

```text
i - j - 1
```

This represents the number of consecutive pictures skipped after the last repost.

This value must never become `>= k`.

Otherwise, there would be a segment of `k` consecutive pictures without any repost.

### Transitions

For every picture `i`, there are two choices.

**1. Repost picture `i`**

We gain its beauty and make `i` the new last reposted position:

```text
a[i] + solve(i + 1, i, x - 1)
```

**2. Don't repost picture `i`**

The last reposted position remains `j`:

```text
solve(i + 1, j, x)
```

Therefore:

```text
dp[i][j][x] =
    max(
        a[i] + dp[i+1][i][x-1],
        dp[i+1][j][x]
    )
```

### Complexity

There are approximately:

```text
O(n² × x)
```

states.

Each state takes `O(1)` work.

Therefore:

```text
Time:  O(n² × x) = O(n³)
Space: O(n² × x) = O(n³)
```

For the easy constraints `n <= 200`, this is acceptable.

### What I Learned

The important observation was converting:

> Every `k` consecutive pictures must contain at least one repost.

into:

> The distance between consecutive reposted pictures can never exceed `k`.

This made the constraint easy to track in the DP state.

### 💻 My Code

```cpp
#include <bits/stdc++.h>
using namespace std;

long long solve(int i, int j, vector<long long> &a, int x, int k, int n, vector<vector<vector<long long>>> &dp)
{
    if (i == a.size())
        return 0;
    if (i - j - 1 >= k)
        return -1e15;
    if (x == 0 && n + 1 - i >= k)
        return -1e15;
    if (x == 0)
        return 0;
    if (dp[i][j][x] != -1)
        return dp[i][j][x];

    long long pk = a[i] + solve(i + 1, i, a, x - 1, k, n, dp);
    long long npk = solve(i + 1, j, a, x, k, n, dp);

    return dp[i][j][x] = max(pk, npk);
}

int main()
{
    long long n, k, x;
    cin >> n >> k >> x;

    vector<long long> a(n + 1);

    for (long long i = 1; i <= n; i++)
        cin >> a[i];

    vector<vector<vector<long long>>> dp(
        n + 2,
        vector<vector<long long>>(
            n + 2,
            vector<long long>(x + 1, -1)
        )
    );

    long long num = solve(1, 0, a, x, k, n, dp);

    if (num < 0)
        cout << -1 << "\n";
    else
        cout << num << "\n";
}
```

---

# 🌳 Problem 2 — Range GCD + Segment Tree

**Topics:** Number Theory + Difference Array + GCD + Segment Tree**

### Problem Idea

We are given an array `a` and `q` queries `[l, r]`.

For each query, we need to find the maximum possible `m` such that:

```text
a[l] % m = a[l+1] % m = ... = a[r] % m
```

If `m` can be infinite, we print `0`.

### My Thought Process

First, I tried to solve the **number theory part**.

Suppose:

```text
a[l] = a₁m + r
a[l+1] = a₂m + r
```

Subtracting the two equations:

```text
a[l+1] - a[l] = (a₂ - a₁)m
```

Therefore:

```text
m divides (a[l+1] - a[l])
```

So `m` must divide every consecutive difference in the range.

This made me think of a **difference array**:

```text
diff[i] = a[i] - a[i-1]
```

For a query `[l, r]`, we need the GCD of:

```text
|a[l+1] - a[l]|
|a[l+2] - a[l+1]|
...
|a[r] - a[r-1]|
```

Therefore, the answer is:

```text
gcd(
    |a[l+1] - a[l]|,
    |a[l+2] - a[l+1]|,
    ...
    |a[r] - a[r-1]|
)
```

Now the problem becomes a **range GCD query**.

Since we have up to `2 * 10^5` elements and queries, I used a **Segment Tree** to answer each GCD query efficiently.

### Key Pattern

```text
Modulo condition
       ↓
Subtract consecutive equations
       ↓
Consecutive differences
       ↓
GCD
       ↓
Range GCD Query
       ↓
Segment Tree
```

### Complexity

Building the difference array and segment tree takes:

```text
O(n)
```

Each query takes:

```text
O(log n)
```

Therefore:

```text
Time: O(n + q log n)
Space: O(n)
```

### 💻 My Code

```cpp
#include <bits/stdc++.h>

using namespace std;
using ll = long long;

#define f(i, a, b) for (int i = a; i < b; i++)
#define br(i, a, b) for (int i = a; i >= b; i--)

void build(int idx, int l, int r,
           vector<int> &segtree,
           vector<int> &a)
{
    if (l == r)
    {
        segtree[idx] = a[l];
        return;
    }

    int mid = (l + r) / 2;

    build(2 * idx + 1, l, mid, segtree, a);
    build(2 * idx + 2, mid + 1, r, segtree, a);

    segtree[idx] =
        gcd(segtree[2 * idx + 1],
            segtree[2 * idx + 2]);
}

int querry(int idx, int l, int r,
           vector<int> &segtree,
           vector<int> &a,
           int ql, int qr)
{
    if (r < ql || l > qr)
        return 0;

    else if (ql <= l && r <= qr)
        return segtree[idx];

    int mid = (l + r) / 2;

    return gcd(
        querry(2 * idx + 1, l, mid,
               segtree, a, ql, qr),

        querry(2 * idx + 2, mid + 1, r,
               segtree, a, ql, qr)
    );
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;

    while (t--)
    {
        int n, q;
        cin >> n >> q;

        vector<int> a(n);

        f(i, 0, n)
            cin >> a[i];

        vector<int> temp;

        f(i, 1, n)
            temp.push_back(abs(a[i] - a[i - 1]));

        vector<int> segtree(
            4 * (int)temp.size() + 1
        );

        if (n != 1)
            build(
                0,
                0,
                (int)temp.size() - 1,
                segtree,
                temp
            );

        while (q--)
        {
            int l, r;
            cin >> l >> r;

            l--, r--;

            if (l == r)
                cout << 0 << " ";
            else
                cout <<
                    querry(
                        0,
                        0,
                        (int)temp.size() - 1,
                        segtree,
                        temp,
                        l,
                        r - 1
                    )
                    << " ";
        }

        cout << "\n";
    }

    return 0;
}
```

---

## 📌 Today's Overall Takeaway

Today's two problems were a good combination of different types of thinking:

**Problem 1:**

```text
Choices
   ↓
State
   ↓
DP
```

**Problem 2:**

```text
Modulo condition
   ↓
Algebra
   ↓
Difference Array
   ↓
GCD
   ↓
Segment Tree
```

The goal is not just to solve problems, but to become better at recognizing these patterns quickly.

**2 problems solved today ✅**

**Keep learning. Keep solving. Keep improving. 🚀**
