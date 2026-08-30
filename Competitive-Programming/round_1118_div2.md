# Codeforces Round 1118 (Div. 2)

**Date:** August 29, 2026
**Rating before contest:** 1088
**Rating after contest:** 1107
**Change:** `+14`
**Solved:** A, B1
**Attempted:** B2, C, D

---

## Contest Experience

This was a useful contest for identifying where I currently stand.

I was able to solve **A and B1** during the contest.

* **A:** Very easy once the main observation was found.
* **B1:** Also relatively straightforward after recognizing the prefix-sum / frequency-counting approach.
* **B2:** Hard version of B. I have not upsolved it yet and need to understand the underlying idea.
* **C:** The problem felt doable, but I did not solve it during the contest.
* **D:** I went directly to D instead of focusing on C first. This was a mistake in contest strategy.

My rating increased by only `+14`, but the contest was still useful because it showed me what I need to improve.

### Main lesson

> **Don't jump to D when C is still unsolved.**

A better strategy would have been:

```text
A → B1 → B2 → C → D
```

Instead, I spent time looking at D before properly finishing C.

---

# Problem A

## Idea

The important observation is that the first and last elements always remain.

For the operation in the problem, adding more elements between two numbers cannot increase their GCD:

gcd(a,b) >= gcd(a,b,c)

```text
Time:  O(log(max(ai)))
Space: O(1)
```

## Code

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

ll gcd(ll a, ll b)
{
    if (b == 0)
        return a;

    return gcd(b, a % b);
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;

    while (t--)
    {
        int n;
        cin >> n;

        vector<ll> a(n);

        for (auto &x : a)
            cin >> x;

        cout << gcd(a[0], a[n - 1]) << '\n';
    }

    return 0;
}
```

# Problem B1

## Idea

This problem can be solved using:

* frequency counting
* prefix sums

First store:

```cpp
hash[x] = number of occurrences of x
```

Then construct:

```cpp
prefix[i] = number of elements <= i
```

This allows us to quickly calculate how many elements lie in any value range.

The solution checks every possible value `i` and calculates how many elements can satisfy the required condition.

The important optimization is that instead of checking every element for every `i`, we use the frequency array and prefix sums.

## Complexity

```text
Time:  O(n + m)
Space: O(m)
```

## Code

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;

    while (t--)
    {
        int n, m;
        cin >> n >> m;

        vector<ll> a(n);

        for (auto &x : a)
            cin >> x;

        vector<ll> freq(m + 1);

        for (ll x : a)
            freq[x]++;

        vector<ll> prefix(m + 1);

        for (int i = 0; i <= m; i++)
        {
            prefix[i] = freq[i];

            if (i > 0)
                prefix[i] += prefix[i - 1];
        }

        ll ans = 0;

        for (int i = 1; i <= m; i++)
        {
            if (i % 2)
            {
                ans = max(ans, prefix[m] - prefix[i - 1]);
            }
            else
            {
                ans = max(
                    ans,
                    prefix[m] - prefix[i / 2 - 1] + freq[i]
                );
            }
        }

        cout << ans << '\n';
    }

    return 0;
}
```



# Problem B2 — Hard Version

## Status

**Not solved / Not upsolved yet.**

The hard version extends B1 by requiring answers for different values of `k`.

This is the problem I need to study after the contest.



### Status

```text
B2 → NEED UPSOLVE
```

---

# Problem C — Interactive

## Main Idea

The problem is based on finding the **diameter of a tree**.

A standard tree-diameter technique is:

1. Start from an arbitrary node.
2. Find the farthest node from it.
3. That node is one endpoint of the diameter.
4. Start from that endpoint.
5. Find the farthest node again.
6. The distance obtained is the diameter.

So conceptually:

```text
arbitrary node
      ↓
farthest node = one endpoint
      ↓
farthest node from endpoint
      ↓
diameter
```

In this interactive problem, instead of directly traversing a known tree, queries are used to determine the distances.

---

## My Approach

### First phase

Start from node `1`.

For every node `j`, repeatedly query the distance until the maximum distance from node `1` is found.

The node with the maximum distance becomes:

```text
node = one endpoint of diameter
```

### Second phase

Now start from `node`.

For every other node, repeatedly query its distance from `node`.

The farthest node becomes the other endpoint:

```text
fnode = second endpoint
```

and the maximum distance is the diameter.

---

## Complexity

time---> O(n) // order of n only

---

## Code

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;

    while (t--)
    {
        int n;
        cin >> n;

        int node = 0;
        int dist = 0;

        for (int j = 2; j <= n; j++)
        {
            int to = 1;

            while (to)
            {
                cout << "? 1 " << j << " " << dist + 1 << endl;

                int num;
                cin >> num;

                if (num)
                {
                    node = j;
                    dist++;
                }

                to = num;
            }
        }

        int fnode = 0;

        for (int j = 1; j <= n; j++)
        {
            if (j != node)
            {
                int to = 1;

                while (to)
                {
                    cout << "? " << node << " "
                         << j << " " << dist << endl;

                    int num;
                    cin >> num;

                    if (num)
                    {
                        dist++;
                        fnode = j;
                    }

                    to = num;
                }
            }
        }

        cout << "! " << node << " "
             << fnode << " "
             << dist - 1 << endl;
    }

    return 0;
}
```

---

# Contest Lessons

## 1. Solve in order of difficulty

My biggest mistake was jumping to D.

I should have spent more time on C before moving on.

Current priority should be:

```text
A → B1 → B2 → C
```

rather than:

```text
A → B1 → D → C
```

---

## 2. B2 is the next important target

First two are becoming comfortable.

The next step is making **third problems** understandable and eventually routine.

---

## 3. Don't judge a contest only by rating change

Rating:

```text
Before: 1088
After : 1107
Change: +14
```

The increase is small, but the contest revealed specific weaknesses:

* harder B problems
* recognizing the intended approach for C
* contest time management
* not jumping unnecessarily to harder problems

---

# Current CP Position

```text
A        → Comfortable
B1       → Comfortable
B2       → Need improvement
C        → Can understand idea, need better contest execution
D        → Not priority yet
```

### Next Focus

```text
1. Upsolve
2. Upsolve C properly
3. Study why B2 works
4. Practice more 1300–1900 rated problems


---

## Final Reflection

This contest was not a great rating jump, but it was a useful learning experience.

I was able to solve A and B1, and C was within reach conceptually. The biggest mistake was spending time on D before properly attempting C.

The goal now is not just to increase the number of solved problems, but to make the jump from **B1 → B2 → C** more natural.

**Current rating: 1107**
