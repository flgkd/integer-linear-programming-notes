# Integer Linear Programming Notes 07: Lagrangian Relaxation and Duality

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

This note introduces **Lagrangian Relaxation** and **Lagrangian Duality**, with emphasis on their use in integer linear programming.

The central idea is simple:

> **Move difficult constraints into the objective function by multiplying them by Lagrangian multipliers, while keeping the remaining easy constraints explicitly in the model.**

This produces a relaxation that may be much easier to solve.

**By optimizing the multipliers, we obtain the best bound available from this family of Lagrangian relaxations.**

Lagrangian duality is a broad topic. A systematic treatment can be found in Chapter 5 of *Convex Optimization* by Boyd and Vandenberghe [1]. The integer-programming presentation in this note also follows the treatment of Sun and Li [2].

**This note focuses on the following questions:**

1. What is Lagrangian Relaxation?
2. Why does it provide a valid bound?
3. Why is the Lagrangian dual problem a convex optimization problem?
4. How is Lagrangian Relaxation used in integer programming?
5. How should we choose which constraints to relax?
6. How can the multiplier problem be solved in practice?

Useful prerequisites are:

- [Simplex Method and Linear Programming Duality](01-simplex-method-and-lp-duality.md)
- [Branch and Bound](02-branch-and-bound.md)
- [Column Generation and Its Applications in Integer Linear Programming](05-column-generation-and-ilp-applications.md)

---

## 2. General Idea of Lagrangian Relaxation⭐

### 2.1 A General Optimization Problem

Consider the optimization problem

$$
\min \quad f_0(x)
$$

subject to

$$
g_i(x)\le 0, \quad i=1,\ldots,m
$$

$$
h_j(x)=0, \quad j=1,\ldots,p
$$

$$
x\in X.
$$

Here:

- $f_0(x)$ is the objective function;
- $g_i(x)\le0$ are inequality constraints;
- $h_j(x)=0$ are equality constraints;
- $X$ contains the remaining restrictions on $x$.

The functions and the set $X$ may have very different structures in different applications.

Suppose the constraints

$$
g_i(x)\le0
$$

and

$$
h_j(x)=0
$$

are difficult to handle, while optimization over

$$
x\in X
$$

is relatively easy.

The basic idea of Lagrangian Relaxation is to **move the difficult constraints into the objective function**.

### 2.2 The Lagrangian Function

Introduce multipliers

$$
\lambda_i \ge 0,\quad i=1,\ldots,m
$$

for the inequality constraints, and unrestricted multipliers

$$
\nu_j \in \mathbb{R},\quad j=1,\ldots,p
$$

for the equality constraints.

The **Lagrangian function** is

$$
L(x,\lambda,\nu)=f_0(x)+\sum_{i=1}^{m}\lambda_i g_i(x)+\sum_{j=1}^{p}\nu_j h_j(x).
$$

For fixed multipliers $(\lambda,\nu)$, the Lagrangian relaxation is

$$
q(\lambda,\nu)=\inf_{x\in X}L(x,\lambda,\nu).
$$

The function $q(\lambda,\nu)$ is called the **Lagrangian dual function**.

The difficult constraints no longer appear as explicit constraints.

Their violations are instead reflected in the objective through the multipliers.

Therefore:

> **Lagrangian Relaxation trades difficult constraints for multiplier-dependent penalty terms in the objective.**

The word “penalty” is useful for intuition, but these are not arbitrary penalty coefficients. The multipliers are decision variables of the dual problem and are optimized systematically.

### 2.3 Why Does the Dual Function Provide a Lower Bound?⭐

Let $x$ be feasible for the original minimization problem.

Then

$$
g_i(x)\le0
$$

and

$$
h_j(x)=0.
$$

Since

$$
\lambda_i\ge0,
$$

we have

$$
\sum_{i=1}^{m}\lambda_i g_i(x)\le0.
$$

Therefore,

$$
L(x,\lambda,\nu)\le f_0(x).
$$

Since the dual function minimizes the Lagrangian over all $x\in X$,

$$
q(\lambda,\nu) \le L(x,\lambda,\nu) \le f_0(x).
$$

This holds for every feasible solution $x$.

Hence,

$$
q(\lambda,\nu)\le p^*,
$$

where $p^*$ is the optimal value of the original minimization problem.

Thus every valid multiplier vector produces a lower bound.

### 2.4 The Lagrangian Dual Problem

Since every feasible multiplier vector provides a lower bound, the best Lagrangian lower bound is obtained by maximizing the dual function:

$$
\max_{\lambda,\nu} \quad q(\lambda,\nu)
$$

subject to

$$
\lambda\ge0.
$$

This is the **Lagrangian dual problem**.

The logic is:

```text
fixed multipliers
        ↓
solve an easier Lagrangian relaxation
        ↓
obtain one lower bound
        ↓
optimize the multipliers
        ↓
obtain the best Lagrangian lower bound
```

For a maximization primal problem, the corresponding construction produces upper bounds and the dual direction is reversed.

---

## 3. Why Is the Lagrangian Dual a Convex Optimization Problem?⭐

For every fixed $x\in X$, the function

$$
L(x,\lambda,\nu)
$$

is affine in $(\lambda,\nu)$.

The dual function is the pointwise infimum of these affine functions:

$$
q(\lambda,\nu) = \inf_{x\in X} L(x,\lambda,\nu).
$$

A pointwise infimum of affine functions is concave.

Therefore:

> **The Lagrangian dual function is concave, even when the original primal problem is nonconvex or contains integer variables.**

The dual problem maximizes this concave function over the convex set

$$
\lambda\ge0.
$$

Under the standard convention of convex optimization, maximizing a concave function over a convex feasible set is a convex optimization problem [1].

This does **not** mean that the dual function is necessarily smooth.

For a linear integer program with a finite feasible set $X$, the dual function is concave and piecewise linear, and it is generally nondifferentiable.

It also does not mean that the Lagrangian dual always has the same optimal value as the primal problem.

A positive duality gap may remain.

### 3.1 Minimum, Infimum, and Bounds

A useful preliminary fact is:

> **The infimum of a set is its greatest lower bound, and the supremum is its least upper bound.**

If the infimum is attained, it is the minimum.

If the supremum is attained, it is the maximum.

Therefore, solving a minimization problem can be viewed as finding the greatest attainable lower bound together with a point that attains it.

This viewpoint is central to duality.

---

## 4. Lagrangian Duality for Integer Linear Programming⭐⭐

Throughout Sections 4–9, we assume that the displayed minima and maxima are attained. Otherwise, `min` and `max` should be replaced by `inf` and `sup`, respectively.

### 4.1 Original Integer Linear Program

Consider the integer linear program

$$
v(IP) = \min \quad c^T x
$$

subject to

$$
Dx\le d
$$

$$
x\in X,
$$

where $X$ is an integer set that is assumed to be easier to optimize over.

For example,

$$
X= \{x\in\mathbb{Z}^n:Ax\le b\}.
$$

The constraints

$$
Dx\le d
$$

are treated as the difficult or coupling constraints.

The restrictions defining $X$ are retained explicitly.

This division is problem-dependent:

```text
difficult constraints:
Dx <= d

easy structured set:
x in X
```

The word “easy” does not necessarily mean polynomial-time solvable.

It means that the resulting subproblem is easier than the original problem or has exploitable structure.

### 4.2 Lagrangian Relaxation

Introduce a multiplier vector

$$
u\ge0
$$

for

$$
Dx\le d.
$$

For fixed $u$, define

$$
z(u) = \min_{x\in X} \{ c^T x+u^T(Dx-d) \}.
$$

This is the **Lagrangian relaxation** of the integer program.

The vector $u$ is the Lagrangian multiplier vector.

The function $z(u)$ is the Lagrangian dual function.

For any feasible solution $x$ of the original problem,

$$
Dx-d\le0.
$$

Since $u\ge0$,

$$
u^T(Dx-d)\le0.
$$

Therefore,

$$
z(u)\le v(IP).
$$

Thus, for every

$$
u\ge0,
$$

the value $z(u)$ is a valid lower bound on the optimal integer objective value.

### 4.3 Lagrangian Dual Problem

The best lower bound from this family is

$$
v(LD) = \max_{u\ge0} z(u).
$$

Equivalently,

$$
v(LD) = \max_{u\ge0} \min_{x\in X} \{ c^T x+u^T(Dx-d) \}.
$$

This is the Lagrangian dual problem associated with the relaxed constraints.

If the relaxed constraints are equalities,

$$
Dx=d,
$$

then the corresponding multipliers are unrestricted:

$$
u\in\mathbb{R}^m.
$$

### 4.4 Weak Duality⭐

For the minimization problem above,

$$
v(LD)\le v(IP).
$$

This is the weak duality relation.

#### Proof

For every

$$
u\ge0
$$

and every feasible $x$ of $(IP)$,

$$
c^T x+u^T(Dx-d)\le c^T x.
$$

Therefore,

$$
z(u) = \min_{x\in X} \{ c^T x+u^T(Dx-d) \} \le c^T x.
$$

Taking the minimum over all feasible primal solutions gives

$$
z(u)\le v(IP).
$$

Finally, maximizing over all valid multipliers gives

$$
v(LD)\le v(IP).
$$

**Therefore, the Lagrangian dual provides a lower bound for a minimization integer program**.

This bound can be used in:

- Branch and Bound;
- lower-bound evaluation;
- exact decomposition algorithms;
- heuristic algorithms;
- optimality-gap estimation.

The classical role of Lagrangian Relaxation in integer programming is surveyed by Fisher [3].

### 4.5 A Sufficient Condition for Zero Duality Gap⭐

Integer programs do not generally satisfy strong Lagrangian duality.

However, suppose there exist $u^{\ast}\ge 0$ and $x^{\ast}\in X$ such that:

1. $x^{\ast}$ is optimal for the Lagrangian relaxation at $u^{\ast}$:

$$
z(u^{\ast})=c^T x^{\ast}+(u^{\ast})^T(Dx^{\ast}-d)=\min_{x\in X}\{c^T x+(u^{\ast})^T(Dx-d)\}.
$$

2. $x^{\ast}$ is feasible for the original integer program:

$$
Dx^{\ast}\le d.
$$

3. Complementary slackness holds:

$$
u_i^{\ast}(D_i x^{\ast}-d_i)=0,\quad i=1,\ldots,m.
$$

Since $x^{\ast}$ is feasible and $u^{\ast}\ge 0$,

$$
(u^{\ast})^T(Dx^{\ast}-d)\le 0.
$$

By complementary slackness,

$$
(u^{\ast})^T(Dx^{\ast}-d)=0.
$$

Therefore,

$$
z(u^{\ast})=c^T x^{\ast}.
$$

By weak duality,

$$
z(u^{\ast})\le v(IP)\le c^T x^{\ast}.
$$

Hence,

$$
z(u^{\ast})=v(IP)=c^T x^{\ast}.
$$

Thus:

- $x^{\ast}$ is an optimal solution of the integer program;
- $u^{\ast}$ is an optimal solution of the Lagrangian dual;
- the Lagrangian duality gap is zero.

This is a sufficient optimality condition.

It should not be interpreted as saying that every integer program satisfies strong duality.

### 4.6 Relationship with Linear Programming Duality

Linear programming duality can be derived as a special case of Lagrangian duality.

For a feasible and bounded LP, under the usual assumptions,

$$
v(LD)=v(LP).
$$

This is the familiar strong duality result for linear programming.

For an integer program, however, the feasible set is nonconvex.

Therefore, in general,

$$
v(LD)<v(IP)
$$

may occur.

The difference

$$
v(IP)-v(LD)
$$

is called the **Lagrangian duality gap** for this minimization problem.

---

## 5. Choosing the Constraints to Relax⭐

Lagrangian Relaxation is useful only when the relaxed problem becomes easier.

A good choice of relaxed constraints should satisfy two competing goals:

1. removing them should reveal useful structure or decomposition;
2. retaining them through multipliers should still produce a strong bound.

This creates a trade-off:

```text
relax more constraints
        ↓
easier subproblem
but possibly weaker bound

relax fewer constraints
        ↓
harder subproblem
but possibly stronger bound
```

Good candidates for relaxation are often:

- coupling constraints linking otherwise independent blocks;
- assignment constraints;
- resource-capacity constraints;
- complicating side constraints;
- constraints that prevent decomposition by customer, machine, facility, flow, or time period.

### 5.1 Lagrangian Relaxation Is Not the Same as LP Relaxation

An LP relaxation usually removes integrality restrictions:

$$
x\in\mathbb{Z}^n \quad\longrightarrow\quad x\in\mathbb{R}^n.
$$

Lagrangian Relaxation instead removes selected constraints and moves them into the objective.

The integrality and combinatorial structure inside $X$ may remain.

Therefore, a Lagrangian relaxation can preserve much more discrete structure than a simple LP relaxation.

This is one reason why a Lagrangian bound can sometimes be stronger than the bound obtained from an intuitive compact LP relaxation.

### 5.2 Convexification Viewpoint

Assume $X$ is finite.

Since a linear function has the same minimum over $X$ and its convex hull,

$$
\min_{x\in X} \{ c^T x+u^T(Dx-d) \} = \min_{x\in\mathrm{conv}(X)} \{ c^T x+u^T(Dx-d) \}.
$$

Under standard LP duality assumptions, the Lagrangian dual bound corresponds to the LP

$$
\min \quad c^T x
$$

subject to

$$
Dx\le d
$$

$$
x\in\mathrm{conv}(X).
$$

This explains why the choice of $X$ matters.

If $\mathrm{conv}(X)$ captures strong local integer structure, the Lagrangian bound can be strong.

This convexification viewpoint is also closely connected to Dantzig-Wolfe Decomposition, which will be introduced in the next note.

---

## 6. Solving the Lagrangian Dual: Projected Supergradient Ascent⭐

The dual function is concave but often nondifferentiable.

Therefore, ordinary smooth gradient methods may not apply.

A classical approach is **projected supergradient ascent**. In the Lagrangian Relaxation literature, this procedure is also commonly referred to as the **subgradient method**.

For a given multiplier vector $u^k$, solve the Lagrangian relaxation and obtain

$$
x^k \in \arg\min_{x\in X} \{ c^T x+{u^k}^T(Dx-d) \}.
$$

A supergradient of the concave dual function at $u^k$ is

$$
s^k=Dx^k-d.
$$

The multiplier update is

$$
u^{k+1} = \max \{ 0, u^k+\alpha_k s^k \},
$$

where the maximum is applied componentwise and $\alpha_k>0$ is a step size.

The interpretation is intuitive:

- if a relaxed constraint is violated, its multiplier tends to increase;
- if it has large slack, its multiplier may decrease;
- the projection ensures $u^{k+1}\ge0$.

When $\|s^k\|>0$, a common Polyak-type step size is

$$
\alpha_k = \theta_k \frac{UB-z(u^k)} {\|s^k\|^2},
$$

where:

- $UB$ is the value of a known primal feasible solution;
- $z(u^k)$ is the current lower bound;
- $0<\theta_k<2$ is a control parameter.

The exact convergence conditions depend on the step-size rule.

In practice, multiplier optimization is often one of the most difficult parts of Lagrangian Relaxation.

### 6.1 Lagrangian Solutions May Be Primal-Infeasible

The solution $x^k$ of a Lagrangian relaxation need not satisfy the relaxed constraints.

Therefore, it may not be feasible for the original integer program.

The Lagrangian relaxation directly provides a bound, not necessarily a feasible primal solution.

A common practical framework is:

```text
solve Lagrangian relaxation
        ↓
obtain lower bound
        ↓
repair or transform the relaxed solution
        ↓
obtain primal feasible solution
        ↓
update upper bound
```

For a minimization problem:

- the Lagrangian dual provides lower bounds;
- primal heuristics provide upper bounds.

Together, they can be used to evaluate and reduce the optimality gap.

---

## 7. Application I: A Binary Integer Program

Consider

$$
\min \quad c^T x
$$

subject to

$$
Ax\le b
$$

$$
x\in\{0,1\}^n.
$$

Relax the constraints

$$
Ax\le b
$$

using multipliers

$$
u\ge0.
$$

The Lagrangian relaxation is

$$
z(u) = \min_{x\in\{0,1\}^n} \{ c^T x+u^T(Ax-b) \}.
$$

Rearranging,

$$
z(u) = -u^T b + \min_{x\in\{0,1\}^n} \{ (c+A^T u)^T x \}.
$$

Because the binary variables are now separable,

$$
z(u) = -u^T b + \sum_{j=1}^{n} \min_{x_j\in\{0,1\}} \{ ( c_j+(A^T u)_j )x_j \}.
$$

Therefore,

$$
z(u) = -u^T b + \sum_{j=1}^{n} \min \{ 0, c_j+(A^T u)_j \}.
$$

An optimal variable value is

$$
x_j(u) = \begin{cases} 1, & c_j+(A^T u)_j<0,\\ 0, & c_j+(A^T u)_j>0. \end{cases}
$$

When the coefficient is zero, either value is optimal.

Once the vector $A^T u$ has been computed, all binary decisions can be made independently in $O(n)$ time.

For a dense matrix $A$, computing $A^T u$ itself requires $O(mn)$ operations.

The same idea extends to bounded integer variables because each variable can be optimized independently over its allowed interval.

---

## 8. Application II: Uncapacitated Facility Location

Consider a profit-maximization version of the **Uncapacitated Facility Location Problem (UFL)**.

Let:

- $M=\{1,\ldots,m\}$ be the customer set;
- $N=\{1,\ldots,n\}$ be the candidate facility set;
- $f_j$ be the fixed cost of opening facility $j$;
- $c_{ij}$ be the profit from serving customer $i$ from facility $j$;
- $y_j=1$ if facility $j$ is opened;
- $x_{ij}$ be the fraction of customer $i$ served by facility $j$.

A formulation is

$$
\max \quad \sum_{i\in M}\sum_{j\in N}c_{ij}x_{ij} - \sum_{j\in N}f_j y_j
$$

subject to

$$
\sum_{j\in N}x_{ij}=1, \quad i\in M
$$

$$
0\le x_{ij}\le y_j, \quad i\in M,\ j\in N
$$

$$
y_j\in\{0,1\}, \quad j\in N.
$$

Although $x_{ij}$ is written as a continuous assignment variable, for any fixed facility-opening decision $y$, an optimal customer assignment can be chosen integral. Therefore, this formulation has the same optimal value as the standard binary-assignment formulation.

The customer-assignment constraints couple all facilities.

Relax them using unrestricted multipliers

$$
u_i\in\mathbb{R}, \quad i\in M.
$$

Since this is a maximization problem, define

$$
z(u) = \max \{ \sum_{i\in M}\sum_{j\in N}c_{ij}x_{ij} - \sum_{j\in N}f_j y_j + \sum_{i\in M}u_i ( 1-\sum_{j\in N}x_{ij} ) \}
$$

subject to

$$
0\le x_{ij}\le y_j
$$

and

$$
y_j\in\{0,1\}.
$$

Rearranging,

$$
z(u) = \sum_{i\in M}u_i + \sum_{j\in N}z_j(u),
$$

where

$$
z_j(u) = \max \{ \sum_{i\in M} (c_{ij}-u_i)x_{ij} - f_j y_j \}
$$

subject to

$$
0\le x_{ij}\le y_j, \quad i\in M
$$

$$
y_j\in\{0,1\}.
$$

The problem decomposes by facility.

For a fixed facility $j$:

- if it is closed, its contribution is zero;
- if it is opened, customer $i$ is included whenever $c_{ij}-u_i>0$.

Therefore,

$$
z_j(u) = \max \{ 0, \sum_{i\in M} \max \{ c_{ij}-u_i, 0 \} -f_j \}.
$$

Thus the Lagrangian relaxation is easy to evaluate.

Because the original problem is a maximization problem, $z(u)$ is an upper bound, and the Lagrangian dual minimizes this upper bound:

$$
\min_{u\in\mathbb{R}^m} z(u).
$$

The important structural effect is:

```text
customer-assignment coupling constraints
        ↓ relaxed
independent facility subproblems
```

---

## 9. Application III: Generalized Assignment Problem⭐⭐

### 9.1 Original Formulation

Consider $m$ machines and $n$ jobs.

Let:

- $c_{ij}$ be the cost of assigning job $j$ to machine $i$;
- $a_{ij}$ be the resource consumption of job $j$ on machine $i$;
- $b_i$ be the capacity of machine $i$;
- $x_{ij}=1$ if job $j$ is assigned to machine $i$.

The Generalized Assignment Problem is

$$
\min \quad \sum_{i=1}^{m} \sum_{j=1}^{n} c_{ij}x_{ij}
$$

subject to

$$
\sum_{i=1}^{m}x_{ij}=1, \quad j=1,\ldots,n
$$

$$
\sum_{j=1}^{n}a_{ij}x_{ij}\le b_i, \quad i=1,\ldots,m
$$

$$
x_{ij}\in\{0,1\}, \quad i=1,\ldots,m,\ j=1,\ldots,n.
$$

There are at least two natural Lagrangian relaxations.

### 9.2 Relaxation I: Relax the Assignment Constraints

Relax

$$
\sum_{i=1}^{m}x_{ij}=1
$$

using unrestricted multipliers

$$
u_j\in\mathbb{R}.
$$

The relaxation is

$$
z_1(u) = \min \{ \sum_{i=1}^{m}\sum_{j=1}^{n}c_{ij}x_{ij} + \sum_{j=1}^{n}u_j ( 1-\sum_{i=1}^{m}x_{ij} ) \}
$$

subject to

$$
\sum_{j=1}^{n}a_{ij}x_{ij}\le b_i, \quad i=1,\ldots,m
$$

$$
x_{ij}\in\{0,1\}.
$$

Rearranging,

$$
z_1(u) = \sum_{j=1}^{n}u_j + \sum_{i=1}^{m}z_i(u),
$$

where

$$
z_i(u) = \min \quad \sum_{j=1}^{n} (c_{ij}-u_j)x_{ij}
$$

subject to

$$
\sum_{j=1}^{n}a_{ij}x_{ij}\le b_i
$$

$$
x_{ij}\in\{0,1\}, \quad j=1,\ldots,n.
$$

Thus, evaluating $z_1(u)$ requires solving $m$ independent binary knapsack problems.

If the capacities and resource consumptions are integral, dynamic programming can solve subproblem $i$ in pseudo-polynomial time

$$
O(nb_i).
$$

The total complexity is therefore

$$
O ( n\sum_{i=1}^{m}b_i ).
$$

If

$$
B=\max_i b_i,
$$

this can be bounded by

$$
O(mnB).
$$

### 9.3 Relaxation II: Relax the Capacity Constraints

Instead, relax

$$
\sum_{j=1}^{n}a_{ij}x_{ij}\le b_i
$$

using nonnegative multipliers

$$
v_i\ge0.
$$

The relaxation is

$$
z_2(v) = \min \{ \sum_{i=1}^{m}\sum_{j=1}^{n}c_{ij}x_{ij} + \sum_{i=1}^{m}v_i ( \sum_{j=1}^{n}a_{ij}x_{ij}-b_i ) \}
$$

subject to

$$
\sum_{i=1}^{m}x_{ij}=1, \quad j=1,\ldots,n
$$

$$
x_{ij}\in\{0,1\}.
$$

Rearranging,

$$
z_2(v) = -\sum_{i=1}^{m}v_i b_i + \min \sum_{j=1}^{n} \sum_{i=1}^{m} (c_{ij}+v_i a_{ij})x_{ij}.
$$

The problem decomposes by job.

For each job $j$, choose the machine with minimum adjusted cost:

$$
\min_{i=1,\ldots,m} \{ c_{ij}+v_i a_{ij} \}.
$$

Therefore,

$$
z_2(v) = -\sum_{i=1}^{m}v_i b_i + \sum_{j=1}^{n} \min_{i=1,\ldots,m} \{ c_{ij}+v_i a_{ij} \}.
$$

This value can be computed in

$$
O(mn)
$$

time.

### 9.4 Comparing the Two Relaxations

The two relaxations reveal different structures:

| Relaxation | Constraints retained | Decomposition | Typical evaluation effort |
|---|---|---|---|
| Relax assignment constraints | Machine capacities | One binary knapsack per machine | Pseudo-polynomial |
| Relax capacity constraints | Job assignments | One minimum-cost choice per job | $O(mn)$ |

Relaxation II is computationally cheaper to evaluate.

However, the cheaper relaxation is not automatically the stronger one.

The quality of the bound depends on:

- which constraints are relaxed;
- which combinatorial structure remains in $X$;
- the instance data;
- how well the multipliers are optimized.

This illustrates the central design trade-off in Lagrangian Relaxation:

> **A useful relaxation should be easy enough to solve repeatedly, but strong enough to provide informative bounds.**

---

## 10. Lagrangian Relaxation as an Algorithmic Framework⭐

Lagrangian Relaxation is not merely a single relaxation formula.

In practical integer optimization, it often forms a complete algorithmic framework:

```text
choose complicating constraints
        ↓
move them into the objective
        ↓
solve decomposed subproblems
        ↓
obtain a dual bound
        ↓
update multipliers
        ↓
construct or repair a primal solution
        ↓
update the primal bound
        ↓
repeat
```

This framework can be combined with:

- Branch and Bound;
- primal heuristics;
- dynamic programming;
- shortest-path algorithms;
- knapsack algorithms;
- decomposition across machines, facilities, flows, or time periods;
- subgradient, bundle, or cutting-plane methods for multiplier optimization.

The main benefit is often decomposition.

The main difficulty is often coordination through the multipliers.

---

## 11. Key Takeaways

1. Lagrangian Relaxation moves selected difficult constraints into the objective using multipliers while retaining an easier structured feasible set.
2. For a minimization problem with relaxed inequalities of the form $Dx\le d$, every multiplier vector $u\ge0$ produces a valid lower bound.
3. The Lagrangian dual maximizes the dual function to obtain the best bound in the chosen family of relaxations.
4. The dual function is concave even when the primal problem is nonconvex or integer, so the Lagrangian dual is a convex optimization problem.
5. Weak duality always holds, but a positive duality gap may remain for integer programs.
6. If a Lagrangian minimizer is primal-feasible and satisfies complementary slackness with its multiplier vector, zero duality gap and primal-dual optimality follow.
7. Lagrangian Relaxation is different from LP relaxation because it can retain integrality and combinatorial structure inside the remaining set $X$.
8. The choice of relaxed constraints determines both the difficulty of the subproblem and the strength of the bound.
9. The dual function is usually nondifferentiable, so subgradient or bundle-type methods are commonly used to optimize the multipliers.
10. Lagrangian solutions may violate the relaxed constraints; primal heuristics are often needed to construct feasible integer solutions.
11. In the Generalized Assignment Problem, relaxing different constraint families produces very different decompositions and computational costs.
12. The convexification viewpoint connects Lagrangian Relaxation with Dantzig-Wolfe Decomposition and Column Generation.

## References

1. Boyd, S. P., and Vandenberghe, L. *Convex Optimization*. Cambridge University Press, 2004. Chapter 5.

2. Sun, Xiaoling, and Duan Li. *Integer Programming*. Beijing: Science Press, 2010. ISBN: 978-7-03-029380-0.（孙小玲、李端：《整数规划》，北京：科学出版社，2010年，ISBN：978-7-03-029380-0）

3. Fisher, M. L. “The Lagrangian Relaxation Method for Solving Integer Programming Problems.” *Management Science*, 27(1), 1981, pp. 1–18. DOI: `10.1287/mnsc.27.1.1`.

## Suggested Follow-up Reading

The most relevant next topics are:

1. **Dantzig-Wolfe Decomposition**  
   Explains how a structured feasible set can be represented through extreme-point or configuration columns and connects directly to the convexification interpretation of Lagrangian Relaxation.

2. **Subgradient and Bundle Methods**  
   Important for solving nonsmooth Lagrangian dual problems efficiently.

3. **Lagrangian Heuristics**  
   Methods for repairing relaxed solutions and constructing high-quality primal feasible solutions.

4. **Augmented Lagrangian Methods**  
   Add quadratic or other stabilization terms to improve multiplier coordination and primal behavior.

5. **Surrogate and Decomposition-Based Relaxations**  
   Related approaches for combining, relaxing, or coordinating difficult constraints.

For this note series, **Dantzig-Wolfe Decomposition** is the next topic and provides a natural continuation of the convexification and decomposition viewpoints introduced here.
