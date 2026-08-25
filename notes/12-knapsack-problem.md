# Integer Linear Programming Notes 12: Knapsack Problem

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

This note studies the **knapsack problem**, one of the most classical problems in combinatorial optimization and integer programming.

This note is primarily based on Sun and Li's *Integer Programming* [1], together with standard references on knapsack problems [2,3].

The knapsack problem is important for at least three reasons:

1. it is a standard model for **resource allocation under capacity constraints**;
2. it is a central example in **dynamic programming**;
3. it plays an important role in **computational complexity theory**.

A point that should be stated carefully is the following:

> **The decision version of knapsack is NP-complete, while the optimization version is NP-hard.**

So when people casually say that “the knapsack problem is NP-complete,” what they usually mean is its **decision form**.

This note mainly focuses on several standard variants:

- 0-1 knapsack;
- complete / unbounded knapsack;
- multiple / bounded knapsack;
- grouped knapsack.

Since the previous note already introduced the general idea of dynamic programming, this note will emphasize how those ideas are specialized to knapsack-type models.

For a review of dynamic programming, see:

- [Dynamic Programming](11-dynamic-programming.md)

For a review of complexity issues such as NP-completeness and pseudo-polynomial time, see:

- [Computational Complexity Theory](10-computational-complexity-theory.md)

---

## 2. Introduction to the Knapsack Problem⭐⭐⭐

### 2.1 Basic Description

In its most intuitive form, the knapsack problem asks:

> Given a set of items, each with a weight (or volume) and a value, how should we choose items so that the total weight does not exceed the capacity of the knapsack and the total value is as large as possible?

This is a **maximization problem under a capacity constraint**.

A standard 0-1 formulation is:

$$
\begin{aligned}
\max \quad & \sum_{j=1}^{n} c_j x_j \\
\text{s.t.} \quad & \sum_{j=1}^{n} a_j x_j \le B, \\
& x_j \in \{0,1\}, \quad j=1,\dots,n.
\end{aligned}
$$

Here:

- $a_j$ is the weight / volume of item $j$;
- $c_j$ is the value / profit of item $j$;
- $B$ is the capacity of the knapsack;
- $x_j=1$ means item $j$ is selected, and $x_j=0$ means it is not selected.

### 2.2 Optimization Form vs. Decision Form

The optimization form asks for the **maximum achievable value**.

The corresponding decision form asks:

> Is there a feasible selection whose total weight is at most $B$ and whose total value is at least some target value $V$?

This distinction is important in complexity theory:

- the **decision version** is NP-complete;
- the **optimization version** is NP-hard.

### 2.3 Why Dynamic Programming Applies

The knapsack problem is a very typical **dynamic programming (DP)** problem.

Its structure is suitable for DP because:

1. we need to consider **all feasible possibilities** while still computing the true optimum;
2. many subproblems repeat, so intermediate results should be stored;
3. the problem can be decomposed into smaller subproblems through a **state transition equation**.

The key structural property is **optimal substructure**.

In practice, for a dynamic-programming formulation, the most important tasks are:

1. defining the **state** clearly;
2. describing the **optimal substructure**;
3. deriving the **recurrence relation**.

---

## 3. 0-1 Knapsack Problem⭐⭐⭐

### 3.1 Problem Description

There are $N$ items and a knapsack with capacity $V$.

Each item can be used **at most once**.

The volume of item $i$ is $v_i$, and its value is $w_i$, where all parameters are positive integers.

We want to select a subset of items so that:

- the total volume does not exceed $V$;
- the total value is maximized.

### 3.2 State Definition and Recurrence

Let

$$
f(i,j)
$$

denote the maximum total value obtainable by choosing from the **first $i$ items** with total volume **not exceeding $j$**.

Then the state transition is

$$
f(i,j)=\max\bigl(f(i-1,j),\; f(i-1,j-v_i)+w_i\bigr), \qquad j\ge v_i.
$$

If $j<v_i$, then item $i$ cannot be taken, so

$$
f(i,j)=f(i-1,j).
$$

This recurrence has a very simple interpretation:

- **do not take item $i$**: value $f(i-1,j)$;
- **take item $i$**: value $f(i-1,j-v_i)+w_i$.

So every state splits naturally into two cases: **take it** or **do not take it**.

<p align="center">
  <img src="../figures/chapter-12/chapter-12-fig1.png" alt="0-1 knapsack DP state interpretation" width="700">
</p>

<p align="center">
  A state-based interpretation of the 0-1 knapsack recurrence.
</p>

### 3.3 Naive Dynamic-Programming Algorithm

The direct 2D dynamic program is:

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)

    f = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(m + 1):
            f[i][j] = f[i - 1][j]
            if j >= v[i]:
                f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i])

    print(f[n][m])


if __name__ == "__main__":
    solve()
```

The time complexity is

$$
O(NV),
$$

and the space complexity is also

$$
O(NV).
$$

### 3.4 Space Optimization⭐

The 2D recurrence only uses the previous row, so we can reduce the state dimension from 2D to 1D.

Let

$$
f(j)
$$

be the best value for capacity $j$ after processing some prefix of items.

Then the 1D transition is

$$
f(j)=\max\bigl(f(j),\; f(j-v_i)+w_i\bigr).
$$

However, the **order of the capacity loop is crucial**.

For 0-1 knapsack, we must traverse $j$ **from large to small**:

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)

    f = [0] * (m + 1)

    for i in range(1, n + 1):
        for j in range(m, v[i] - 1, -1):
            f[j] = max(f[j], f[j - v[i]] + w[i])

    print(f[m])


if __name__ == "__main__":
    solve()
```

Why must the loop go from large to small?

Because we need the transition to use the value from the **previous layer**:

$$
f(i-1,j-v_i),
$$

not the already updated current-layer value.

If we loop $j$ from small to large, then $f(j-v_i)$ may already have been updated using item $i$, which would incorrectly allow item $i$ to be used multiple times.

So for **0-1 knapsack**:

> **Descending capacity order ensures that each item is used at most once.**

---

## 4. Complete / Unbounded Knapsack Problem⭐⭐⭐

### 4.1 Problem Description

There are $N$ types of items and a knapsack with capacity $V$.

Each type of item can be used **infinitely many times**.

The volume of item type $i$ is $v_i$, and its value is $w_i$, where all parameters are positive integers.

We want to maximize the total value subject to the capacity constraint.

### 4.2 State Definition and Naive Recurrence

Let

$$
f(i,j)
$$

denote the maximum value obtainable by using the first $i$ item types with total volume not exceeding $j$.

Then the most direct recurrence is

$$
f(i,j)=\max_{0\le k\le \lfloor j/v_i\rfloor}\Bigl(f(i-1,j-kv_i)+kw_i\Bigr).
$$

This means that for item type $i$, we may choose:

- $0$ copies,
- $1$ copy,
- $2$ copies,
- and so on,

as long as the total volume remains feasible.

### 4.3 Naive Dynamic-Programming Algorithm

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)

    f = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(m + 1):
            k = 0
            while k * v[i] <= j:
                f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + k * w[i])
                k += 1

    print(f[n][m])


if __name__ == "__main__":
    solve()
```

The time complexity is

$$
O\!\left(NV\cdot \max_i \frac{V}{v_i}\right),
$$

which is often written more simply as a three-loop dynamic program.

### 4.4 First Optimization: A Better Recurrence⭐

The naive recurrence can be rewritten more efficiently.

Expand the formula:

$$
\begin{aligned}
f(i,j)=\max\{&f(i-1,j), \\
&f(i-1,j-v_i)+w_i, \\
&f(i-1,j-2v_i)+2w_i, \\
&\dots\}.
\end{aligned}
$$

Now consider

$$
f(i,j-v_i)+w_i.
$$

Using the definition of $f(i,j-v_i)$, this becomes

$$
\max\{f(i-1,j-v_i)+w_i,\; f(i-1,j-2v_i)+2w_i,\; \dots\}.
$$

Therefore we obtain the important simplified recurrence:

$$
f(i,j)=\max\bigl(f(i-1,j),\; f(i,j-v_i)+w_i\bigr), \qquad j\ge v_i.
$$

This is the key difference from 0-1 knapsack.

Here the second term uses

$$
f(i,j-v_i),
$$

not

$$
f(i-1,j-v_i).
$$

This reflects the fact that item type $i$ may be used repeatedly.

### 4.5 First Optimized 2D Algorithm

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)

    f = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(m + 1):
            f[i][j] = f[i - 1][j]
            if j >= v[i]:
                f[i][j] = max(f[i][j], f[i][j - v[i]] + w[i])

    print(f[n][m])


if __name__ == "__main__":
    solve()
```

### 4.6 Second Optimization: 1D DP⭐

The same idea can be compressed into 1D.

Now the capacity loop must go **from small to large**:

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)

    f = [0] * (m + 1)

    for i in range(1, n + 1):
        for j in range(v[i], m + 1):
            f[j] = max(f[j], f[j - v[i]] + w[i])

    print(f[m])


if __name__ == "__main__":
    solve()
```

Why ascending order this time?

Because we want the transition to use the **current-layer** value

$$
f(i,j-v_i),
$$

which already accounts for the possibility of using item type $i$ multiple times.

So for **complete knapsack**:

> **Ascending capacity order allows an item type to be reused.**

---

## 5. Multiple / Bounded Knapsack Problem⭐⭐⭐

### 5.1 Problem Description

There are $N$ types of items and a knapsack with capacity $V$.

Item type $i$ has:

- volume $v_i$,
- value $w_i$,
- quantity limit $s_i$.

We want to maximize total value subject to the capacity constraint.

### 5.2 State Definition and Naive Recurrence

Let

$$
f(i,j)
$$

denote the maximum value obtainable by using the first $i$ item types with total volume not exceeding $j$.

Then the direct recurrence is

$$
f(i,j)=\max_{0\le k\le \min(s_i,\lfloor j/v_i\rfloor)}\Bigl(f(i-1,j-kv_i)+kw_i\Bigr).
$$

This is almost the same as the complete knapsack recurrence, except that now the number of copies is limited by $s_i$.

### 5.3 Naive Dynamic-Programming Algorithm

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    v = [0] * (n + 1)
    w = [0] * (n + 1)
    s = [0] * (n + 1)
    for i in range(1, n + 1):
        v[i] = next(it)
        w[i] = next(it)
        s[i] = next(it)

    f = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(m + 1):
            k = 0
            while k <= s[i] and k * v[i] <= j:
                f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + k * w[i])
                k += 1

    print(f[n][m])


if __name__ == "__main__":
    solve()
```

### 5.4 Why the Complete-Knapsack Optimization Does Not Directly Work

At first glance, the multiple knapsack problem looks very similar to the complete knapsack problem, so one may try to reuse the recurrence

$$
f(i,j)=\max\bigl(f(i-1,j),\; f(i,j-v_i)+w_i\bigr).
$$

But this is generally **not valid**.

The reason is that in the complete knapsack problem, item type $i$ may be chosen **arbitrarily many times**, so the recursive expansion is consistent.

In the multiple knapsack problem, however, the number of copies of type $i$ is capped at $s_i$. If we write the complete-knapsack-style recurrence blindly, then repeated reuse of $f(i,j-v_i)$ may allow more than $s_i$ copies.

So the complete-knapsack simplification does not preserve the quantity limit in general.

### 5.5 Binary Splitting Optimization⭐⭐⭐

A standard optimization is to transform the bounded problem into a 0-1 knapsack problem by **binary decomposition**.

Suppose type $i$ has $s_i$ copies.

We split these copies into groups of sizes

$$
1,2,4,8,\dots
$$

until the remaining number is smaller than the next power of two.

For example, if $s_i=13$, we can split it as

$$
13=1+2+4+6.
$$

Each group becomes an artificial 0-1 item:

- group size $k$ has volume $k v_i$;
- group size $k$ has value $k w_i$.

Then choosing a subset of these artificial items is equivalent to choosing any number from $0$ to $s_i$ copies of the original item type.

So the multiple knapsack problem becomes a 0-1 knapsack problem after decomposition.

This reduces the complexity from roughly

$$
O(NVs)
$$

to roughly

$$
O\!\left(V\sum_{i=1}^N \log s_i\right)
$$

after splitting.

### 5.6 Binary-Optimized Algorithm

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    items = []
    for _ in range(n):
        a = next(it)  # volume
        b = next(it)  # value
        s = next(it)  # quantity

        k = 1
        while k <= s:
            items.append((k * a, k * b))
            s -= k
            k <<= 1
        if s > 0:
            items.append((s * a, s * b))

    f = [0] * (m + 1)

    for vol, val in items:
        for j in range(m, vol - 1, -1):
            f[j] = max(f[j], f[j - vol] + val)

    print(f[m])


if __name__ == "__main__":
    solve()
```

---

## 6. Grouped Knapsack Problem⭐⭐⭐

### 6.1 Problem Description

There are $N$ groups of items and a knapsack with capacity $V$.

Each group contains several candidate items, and **at most one item may be selected from each group**.

If item $k$ in group $i$ is chosen, its volume is $v_{ik}$ and its value is $w_{ik}$.

We want to maximize total value subject to the capacity constraint.

### 6.2 State Definition and Recurrence

Let

$$
f(i,j)
$$

denote the maximum value obtainable from the first $i$ groups with total volume not exceeding $j$.

Then the recurrence is

$$
f(i,j)=\max\left\{f(i-1,j),\; \max_{1\le k\le s_i,\; v_{ik}\le j}\bigl(f(i-1,j-v_{ik})+w_{ik}\bigr)\right\},
$$

where $s_i$ is the number of items in group $i$.

The first term corresponds to **choosing nothing from group $i$**.

The inner maximization corresponds to **choosing one specific item from group $i$**.

So for each group, we choose **at most one** item.

### 6.3 Naive Dynamic-Programming Algorithm

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    groups = [[] for _ in range(n + 1)]
    for i in range(1, n + 1):
        s = next(it)
        for _ in range(s):
            vol = next(it)
            val = next(it)
            groups[i].append((vol, val))

    f = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(m + 1):
            f[i][j] = f[i - 1][j]
            for vol, val in groups[i]:
                if j >= vol:
                    f[i][j] = max(f[i][j], f[i - 1][j - vol] + val)

    print(f[n][m])


if __name__ == "__main__":
    solve()
```

### 6.4 1D Space Optimization

The grouped knapsack problem can also be compressed to 1D.

The key point is the same as in 0-1 knapsack:

> **The capacity index must be scanned from large to small, so that the values used by the current group still come from the previous group.**

```python
import sys


def solve() -> None:
    data = sys.stdin.read().strip().split()
    if not data:
        return

    it = iter(map(int, data))
    n = next(it)
    m = next(it)

    groups = []
    for _ in range(n):
        s = next(it)
        group = []
        for _ in range(s):
            vol = next(it)
            val = next(it)
            group.append((vol, val))
        groups.append(group)

    f = [0] * (m + 1)

    for group in groups:
        for j in range(m, -1, -1):
            for vol, val in group:
                if j >= vol:
                    f[j] = max(f[j], f[j - vol] + val)

    print(f[m])


if __name__ == "__main__":
    solve()
```

---

## 7. Comparing the Four Variants⭐⭐⭐

A useful summary is the following.

| Variant | Item availability | State transition idea | Capacity loop direction in 1D DP |
| :-- | :-- | :-- | :--: |
| 0-1 knapsack | each item at most once | take / do not take | descending |
| complete knapsack | unlimited copies | reuse current item type | ascending |
| multiple knapsack | limited copies | enumerate copies or split into binary groups | descending after binary splitting |
| grouped knapsack | at most one item per group | choose none or one from current group | descending |

This table captures the most important algorithmic difference among the standard knapsack variants.

---

## 8. Complexity and Theory Notes⭐

### 8.1 Pseudo-Polynomial Nature

The dynamic-programming algorithms above often run in time polynomial in $N$ and $V$.

However, this does **not** mean they are polynomial in the formal complexity-theoretic sense.

The reason is that the capacity $V$ is a **numerical value**, while the input length only needs

$$
O(\log V)
$$

bits to encode it.

So an $O(NV)$ algorithm is generally **pseudo-polynomial**, not polynomial in the binary input length.

This is exactly why 0-1 knapsack can be NP-hard even though it admits a practical DP algorithm.

### 8.2 Weak NP-Hardness

The ordinary 0-1 knapsack problem is a classical example of a **weakly NP-hard** optimization problem.

This means:

- it is NP-hard in general;
- but it admits a pseudo-polynomial dynamic program.

That is one of the main reasons the knapsack problem is used so often in algorithm courses and integer-programming courses.

---

## 9. Key Takeaways⭐⭐⭐

1. The knapsack problem is a classical resource-allocation problem under a capacity constraint.
2. The **decision** version is NP-complete, while the **optimization** version is NP-hard.
3. The central DP task is to define a good state and derive the correct recurrence.
4. For **0-1 knapsack**, the 1D capacity loop must go **from large to small**.
5. For **complete knapsack**, the 1D capacity loop must go **from small to large**.
6. **Multiple knapsack** can be efficiently reduced to a 0-1 knapsack problem by **binary splitting**.
7. **Grouped knapsack** chooses at most one item from each group and also uses a descending capacity loop in the 1D version.
8. The usual DP algorithms for knapsack are typically **pseudo-polynomial**, not polynomial in the encoded input length.

---

## Suggested Follow-up Reading

The most relevant next topics are:

1. **Assignment Problem**  
   A useful contrast to knapsack: it is also a structured combinatorial optimization problem, but unlike knapsack it admits polynomial-time algorithms.

2. **Approximation Algorithms**  
   Especially relevant for NP-hard problems such as knapsack, where exact polynomial-time algorithms may not exist but high-quality solutions with provable guarantees can still be obtained.

3. **Branch and Bound / Branch and Cut**  
   Useful for understanding how more general integer-programming methods solve knapsack-type models beyond pseudo-polynomial dynamic programming.

4. **Column Generation and Decomposition Methods**  
   Helpful for seeing how very different large-scale optimization techniques compare with dynamic programming on special structures.

---

## References

1. Sun, Xiaoling, and Duan Li. *Integer Programming*. Beijing: Science Press, 2010. ISBN: 978-7-03-029380-0.
2. Martello, Silvano, and Paolo Toth. *Knapsack Problems: Algorithms and Computer Implementations*. Chichester: John Wiley & Sons, 1990. ISBN: 978-0-471-92420-3.
3. Kellerer, Hans, Ulrich Pferschy, and David Pisinger. *Knapsack Problems*. Berlin: Springer, 2004. DOI: 10.1007/978-3-540-24777-7.
4. Bellman, Richard. *Dynamic Programming*. Princeton, NJ: Princeton University Press, 1957.
