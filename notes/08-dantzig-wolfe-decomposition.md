# Integer Linear Programming Notes 08: Dantzig-Wolfe Decomposition

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

In the previous note on Column Generation, the Cutting Stock Problem was reformulated from an intuitive roll-based model into a pattern-based model.

The pattern-based formulation has a much larger number of variables, but each variable represents a complete feasible cutting pattern. This structure makes Column Generation natural.

A practical question then arises:

> **If an optimization model is not naturally written in a column-based form, how can we systematically transform it into one?**

**Dantzig-Wolfe Decomposition** provides a classical answer for models with a suitable block-angular structure [1].

At a high level,

```text
structured original model
        ↓
identify independent blocks and linking constraints
        ↓
represent each block by its extreme points
or feasible integer configurations
        ↓
construct a Master Problem
        ↓
solve the huge Master Problem by Column Generation
```

Therefore:

> **Dantzig-Wolfe Decomposition is primarily a reformulation and decomposition framework. Column Generation is the algorithmic mechanism commonly used to solve the resulting large Master Problem.**

This note focuses on the relationship among:

- block-angular structure;
- the Minkowski-Weyl theorem;
- the Dantzig-Wolfe Master Problem;
- the Restricted Master Problem;
- Pricing Problems;
- Column Generation;
- Lagrangian Relaxation.

Useful prerequisites are:

- [Column Generation and Its Applications in Integer Linear Programming](05-column-generation-and-ilp-applications.md)
- [Branch and Price](06-branch-and-price.md)
- [Lagrangian Relaxation and Duality](07-lagrangian-relaxation-and-duality.md)

---

## 2. Applicable Structure: Block-Angular Models⭐

### 2.1 A Block-Angular Model

Consider the following structured optimization problem:

$$
\min \quad \sum_{k=1}^{K} c_k^T x_k
$$

subject to

$$
\sum_{k=1}^{K} A_k x_k=b
$$

$$
x_k\in X_k,\quad k=1,\ldots,K.
$$

For clarity, the linking constraints are written as equalities. Inequality linking constraints can be handled similarly, with the corresponding sign restrictions on their dual variables.

Each vector $x_k$ represents one subsystem or block.

A typical structured set is

$$
X_k=\{x_k\in\mathbb{R}^{n_k}:D_kx_k\le d_k\},
$$

or, in an integer model,

$$
X_k=\{x_k\in\mathbb{Z}^{n_k}:D_kx_k\le d_k\}.
$$

The corresponding constraint matrix has the block-angular form

```text
          x1       x2       ...       xK

linking   A1       A2       ...       AK
          --------------------------------
local     D1        0       ...        0
           0       D2       ...        0
          ...      ...      ...       ...
           0        0       ...       DK
```

The important structural property is:

> **The local constraints of block $k$ involve only $x_k$, while the linking constraints couple different blocks together.**

Examples of blocks include:

- one commodity in a multicommodity network;
- one service flow;
- one machine;
- one customer group;
- one vehicle;
- one time period;
- one independent subsystem.

### 2.2 What Makes the Structure Decomposable?

Without the linking constraints,

$$
\sum_{k=1}^{K}A_kx_k=b,
$$

the blocks are independent:

$$
x_k\in X_k,\quad k=1,\ldots,K.
$$

The difficult interaction among blocks is concentrated in a relatively small set of global constraints.

This is precisely the structure Dantzig-Wolfe Decomposition exploits.

A global linking row is not required in every definition of block-angular structure. If no linking constraints are present, the problem becomes block diagonal, which is an even simpler special case.

The sets $X_k$ may be:

1. continuous polyhedra;
2. bounded polytopes;
3. finite discrete or integer sets.

The type of $X_k$ determines how the Dantzig-Wolfe reformulation is written.

---

## 3. Minkowski-Weyl Theorem and the Representation of a Block⭐⭐

The mathematical foundation of Dantzig-Wolfe Decomposition is the Minkowski-Weyl theorem [2].

### 3.1 General Polyhedral Representation

Let $X$ be a nonempty pointed polyhedron. By the Minkowski-Weyl theorem, $X$ can be represented using its extreme points and extreme rays [2].

Let the extreme points of a polyhedron $X$ be

$$
v^1,\ldots,v^T,
$$

and let its extreme rays be

$$
r^1,\ldots,r^S.
$$

A point $x\in X$ can be represented as

$$
x=\sum_{t=1}^{T}\lambda_t v^t+\sum_{s=1}^{S}\mu_s r^s
$$

subject to

$$
\sum_{t=1}^{T}\lambda_t=1
$$

$$
\lambda_t\ge0,\quad t=1,\ldots,T
$$

$$
\mu_s\ge0,\quad s=1,\ldots,S.
$$

This representation separates two different components:

- a convex combination of points;
- a conic combination of rays.

### 3.2 Bounded Polyhedron

If $X_k$ is bounded, it has no recession rays.

Let its extreme points be

$$
x_k^1,\ldots,x_k^{T_k}.
$$

Then every point $x_k\in X_k$ can be written as

$$
x_k=\sum_{t=1}^{T_k}\lambda_{kt}x_k^t
$$

subject to

$$
\sum_{t=1}^{T_k}\lambda_{kt}=1
$$

$$
\lambda_{kt}\ge0,\quad t=1,\ldots,T_k.
$$

Thus,

$$
X_k=\mathrm{conv}\{x_k^1,\ldots,x_k^{T_k}\}.
$$

This is the key representation used in the continuous Dantzig-Wolfe reformulation.

### 3.3 Finite Discrete or Integer Set

Suppose instead that $X_k$ is a finite discrete set:

$$
X_k=\{x_k^1,\ldots,x_k^{T_k}\}.
$$

An original integer decision chooses exactly one feasible point from this set.

It can therefore be represented by

$$
x_k=\sum_{t=1}^{T_k}\lambda_{kt}x_k^t
$$

subject to

$$
\sum_{t=1}^{T_k}\lambda_{kt}=1
$$

$$
\lambda_{kt}\in\{0,1\}.
$$

Exactly one $\lambda_{kt}$ equals 1.

After relaxing integrality,

$$
\lambda_{kt}\ge0,
$$

the master variables describe a convex combination of complete feasible integer configurations.

This distinction between the integer Master Problem and its LP relaxation is important.

---

## 4. Constructing the Dantzig-Wolfe Master Problem⭐⭐⭐

### 4.1 Substitute the Block Representation

For simplicity, the following derivation focuses on the case in which each block $X_k$ is bounded or consists of a finite set of feasible configurations. If a continuous block is unbounded, the full Dantzig-Wolfe reformulation must also include variables associated with its extreme rays.

Consider first the case in which each $X_k$ is represented by a finite collection of feasible points

$$
x_k^1,\ldots,x_k^{T_k}.
$$

Substitute

$$
x_k=\sum_{t=1}^{T_k}\lambda_{kt}x_k^t
$$

into the objective:

$$
\sum_{k=1}^{K}c_k^Tx_k=\sum_{k=1}^{K}\sum_{t=1}^{T_k}(c_k^Tx_k^t)\lambda_{kt}.
$$

The linking constraints become

$$
\sum_{k=1}^{K}\sum_{t=1}^{T_k}(A_kx_k^t)\lambda_{kt}=b.
$$

For each block, we also have a convexity constraint:

$$
\sum_{t=1}^{T_k}\lambda_{kt}=1,\quad k=1,\ldots,K.
$$

Therefore, the Dantzig-Wolfe Master Problem is

$$
\min \quad \sum_{k=1}^{K}\sum_{t=1}^{T_k}(c_k^Tx_k^t)\lambda_{kt}
$$

subject to

$$
\sum_{k=1}^{K}\sum_{t=1}^{T_k}(A_kx_k^t)\lambda_{kt}=b
$$

$$
\sum_{t=1}^{T_k}\lambda_{kt}=1,\quad k=1,\ldots,K.
$$

If $X_k$ is a finite integer set and the reformulation is intended to be exactly equivalent to the original integer model, then

$$
\lambda_{kt}\in\{0,1\}.
$$

If $X_k$ is a continuous bounded polytope, then

$$
\lambda_{kt}\ge0
$$

from the beginning.

### 4.2 The LP Master Problem

For an integer model, relaxing the master-variable integrality gives

$$
\lambda_{kt}\ge0.
$$

The resulting LP is the **LP Master Problem**.

Its variables may be interpreted as weights assigned to feasible block configurations.

The key structural change is:

```text
original formulation:
variables describe local decisions directly

Dantzig-Wolfe formulation:
variables describe complete feasible block configurations
```

### 4.3 Why Does the Master Problem Have So Many Columns?

The set $X_k$ may contain a huge number of extreme points or integer-feasible configurations.

Therefore,

$$
T_k
$$

may be extremely large.

Explicitly enumerating all variables

$$
\lambda_{kt}
$$

can be impossible.

This is not a failure of the reformulation.

It is exactly why Column Generation becomes useful.

> **Dantzig-Wolfe Decomposition creates the structured column space; Column Generation searches that column space without enumerating it explicitly.**

The original Dantzig-Wolfe paper already describes this alternating structure between a coordinating problem and independent subproblems that generate new activities, or columns [1].

---

## 5. Solving the Dantzig-Wolfe Master Problem by Column Generation⭐⭐⭐

### 5.1 Restricted Master Problem

Suppose only a subset

$$
P_k\subseteq\{1,\ldots,T_k\}
$$

of the columns of block $k$ is currently available.

The Restricted Master Problem is

$$
\min \quad \sum_{k=1}^{K}\sum_{t\in P_k}(c_k^Tx_k^t)\lambda_{kt}
$$

subject to

$$
\sum_{k=1}^{K}\sum_{t\in P_k}(A_kx_k^t)\lambda_{kt}=b
$$

$$
\sum_{t\in P_k}\lambda_{kt}=1,\quad k=1,\ldots,K
$$

$$
\lambda_{kt}\ge0,\quad t\in P_k.
$$

The initial column sets should make the RMP feasible, or a Phase I / artificial-column procedure is required.

Let

$$
\pi
$$

be the dual vector associated with the linking constraints, and let

$$
u_k
$$

be the dual variable associated with the convexity constraint of block $k$.

Because the constraints are written as equalities, these dual variables are unrestricted.

### 5.2 Reduced Cost of a Candidate Column

Consider any feasible point

$$
x\in X_k.
$$

If this point is added as a new column for block $k$, its objective coefficient is

$$
c_k^Tx.
$$

Its coefficient vector in the linking constraints is

$$
A_kx,
$$

and its coefficient in the block-$k$ convexity constraint is 1.

Therefore, its reduced cost is

$$
r_k(x)=c_k^Tx-\pi^TA_kx-u_k.
$$

Equivalently,

$$
r_k(x)=(c_k-A_k^T\pi)^Tx-u_k.
$$

For a minimization problem, a column can improve the current RMP if

$$
r_k(x)<0.
$$

### 5.3 Pricing Problem

Instead of enumerating every point in $X_k$, solve the Pricing Problem

$$
s_k=\min_{x\in X_k}\{(c_k-A_k^T\pi)^Tx-u_k\}.
$$

Since $u_k$ is constant with respect to $x$, the same minimizer can be found by solving

$$
\min_{x\in X_k}(c_k-A_k^T\pi)^Tx.
$$

The solution of this subproblem is a complete feasible configuration of block $k$.

Its corresponding master column is

$$
\begin{bmatrix} A_kx\\ e_k \end{bmatrix},
$$

with objective coefficient

$$
c_k^Tx,
$$

where $e_k$ represents the coefficient 1 in the convexity constraint of block $k$.

This gives the central Dantzig-Wolfe interpretation:

> **The Pricing Problem does not search for an individual original variable. It searches for an entire feasible block solution that can become one new master column.**

### 5.4 Stopping Criterion

For each block, solve its Pricing Problem.

If

$$
s_k\ge0,\quad k=1,\ldots,K,
$$

then no block contains a negative reduced-cost column.

Therefore, the current RMP solution is optimal for the full LP Master Problem.

If

$$
s_k<0
$$

for some block $k$, the corresponding pricing solution generates an improving column.

Add the column to the RMP and reoptimize.

The complete process is

```text
solve RMP
    ↓
obtain dual variables
    ↓
solve one Pricing Problem for each block
    ↓
negative reduced-cost column exists?
    ├── Yes → add column(s) → solve RMP again
    └── No  → full LP Master Problem solved
```

This is the standard connection between Dantzig-Wolfe Decomposition and Column Generation [3].

### 5.5 Relation Between the RMP and the Full Master Problem

Every feasible RMP solution can be extended to the full Master Problem by assigning zero to columns that are not currently present.

Therefore, for a minimization problem,

$$
v(RMP)\ge v(MP).
$$

As more improving columns are generated, the RMP objective can decrease.

At exact Column Generation convergence,

$$
v(RMP)=v(MP).
$$

### 5.6 A Dual Lower Bound Before Full Convergence⭐

The Pricing Problems can also provide a valid lower bound on the full LP Master Problem before Column Generation has converged.

For block $k$, let

$$
s_k=\min_{x\in X_k}\{c_k^Tx-\pi^TA_kx-u_k\}.
$$

By definition,

$$
c_k^Tx-\pi^TA_kx-u_k\ge s_k,\quad x\in X_k.
$$

Therefore,

$$
\pi^TA_kx+(u_k+s_k)\le c_k^Tx,\quad x\in X_k.
$$

This means that

$$
(\pi,u_1+s_1,\ldots,u_K+s_K)
$$

is dual feasible for the full LP Master Problem.

Hence,

$$
b^T\pi+\sum_{k=1}^{K}(u_k+s_k)\le v(MP).
$$

This is a useful certified lower bound for a minimization problem.

When all

$$
s_k\ge0,
$$

the current RMP dual solution itself is feasible for the full master dual, and Column Generation has converged.

---

## 6. What Is Dantzig-Wolfe Decomposition Really Doing?⭐⭐⭐

The formulas above describe the procedure.

This section gives a more intuitive interpretation.

### 6.1 From Variable-Level Decisions to Configuration-Level Decisions

In a compact model, one complete local solution may be represented by many individual variables.

Examples include:

- all edges of one route;
- all placement and routing decisions of one service flow;
- all assignments associated with one machine;
- one complete schedule;
- one feasible cutting pattern.

Dantzig-Wolfe Decomposition packages such a complete feasible local structure into one master column.

Therefore, the reformulated model asks:

> **Which complete local configurations should be combined to satisfy the global linking constraints?**

rather than only:

> Which individual original variables should be set to 0, 1, or some continuous value?

This change of viewpoint is one of the most useful ways to understand Dantzig-Wolfe Decomposition.

### 6.2 Why Extreme Points?

For a bounded polyhedron, every point is a convex combination of its extreme points.

Moreover, if a linear program over a nonempty polytope has an optimal solution, at least one optimal extreme point exists.

Therefore, using extreme points as columns does not discard information from the continuous block polytope.

For a finite integer set, each column can instead represent one complete integer-feasible block configuration.

The Master Problem then coordinates these configurations globally.

### 6.3 Why Are the Master Variables Fractional in the LP Relaxation?

In the integer Master Problem for a finite discrete block,

$$
\lambda_{kt}\in\{0,1\}
$$

selects one configuration for block $k$.

In the LP relaxation,

$$
\lambda_{kt}\ge0
$$

and

$$
\sum_t\lambda_{kt}=1.
$$

Therefore, one block may be represented by a fractional convex combination such as

```text
0.3 × configuration A
+
0.7 × configuration B
```

This is not necessarily a physically realizable integer solution.

It is an LP-relaxation solution.

The important point is that the combination occurs among **complete feasible local configurations**.

### 6.4 An Important Benefit: A Stronger LP Relaxation⭐

One of the most important benefits of Dantzig-Wolfe Decomposition in integer programming is that it can provide a **stronger LP relaxation** when the problem has a suitable block-angular structure and each block is represented by its integer-feasible configurations [4].

This stronger relaxation can be extremely valuable in exact algorithms.

A stronger LP bound may:

- reduce the optimality gap at the root node;
- improve lower bounds in Branch and Bound for minimization problems;
- reduce the number of nodes that must be explored;
- and therefore potentially accelerate the overall solution process.

This is one of the main reasons Dantzig-Wolfe reformulation and Branch and Price are so effective for many large-scale structured integer programs [3,4].

#### Why Can the Relaxation Be Stronger?

Suppose block $k$ has the integer-feasible set

$$
X_k^I=\{x_k\in\mathbb{Z}^{n_k}:D_kx_k\le d_k\}.
$$

A naive compact LP relaxation removes the integrality restriction and uses

$$
P_k=\{x_k\in\mathbb{R}^{n_k}:D_kx_k\le d_k\}.
$$

In general,

$$
\mathrm{conv}(X_k^I)\subseteq P_k.
$$

A Dantzig-Wolfe Master Problem whose columns represent integer-feasible points of $X_k^I$ works, at the LP level, over

$$
\mathrm{conv}(X_k^I).
$$

Therefore, the Dantzig-Wolfe LP relaxation preserves the convex hull of the local integer-feasible configurations rather than simply replacing the integer variables by continuous variables.

For a minimization problem, this implies that the Dantzig-Wolfe LP bound is at least as strong as the corresponding naive compact LP bound:

$$
v(DW\text{-}LP)\ge v(Compact\text{-}LP).
$$

For a maximization problem, the inequality is reversed:

$$
v(DW\text{-}LP)\le v(Compact\text{-}LP).
$$

The intuition is:

```text
naive compact LP relaxation
→ relax integrality inside each block
→ may allow locally unrealistic fractional points

Dantzig-Wolfe LP relaxation
→ convex combinations of integer-feasible block configurations
→ local integer structure is preserved through conv(X_k^I)
```

This is the precise mathematical basis for saying that Dantzig-Wolfe reformulation can produce a stronger relaxation in integer programming.

Vanderbeck [4] explicitly describes Dantzig-Wolfe decomposition for integer programming as a reformulation aimed at obtaining a tighter LP relaxation bound.

#### Important Qualification: Stronger Does Not Mean Strictly Stronger in Every Case

The previous conclusion must be interpreted carefully.

Dantzig-Wolfe Decomposition does **not** guarantee a strictly better LP bound for every model.

If

$$
\mathrm{conv}(X_k^I)=P_k,
$$

for every block, then the Dantzig-Wolfe LP relaxation and the compact LP relaxation may have the same bound.

More generally, even if

$$
\mathrm{conv}(X_k^I)\subsetneq P_k,
$$

the optimal objective values can still coincide for a particular instance.

Therefore, the rigorous statement is:

> **For an integer program with an appropriate Dantzig-Wolfe block decomposition, convexifying the integer-feasible block sets produces an LP relaxation that is no weaker than the corresponding naive compact LP relaxation, and it can be strictly stronger.**

There is also an important continuous-LP special case.

If the original problem is already a continuous LP and Dantzig-Wolfe merely rewrites each polyhedral block using its extreme points and rays, then the reformulation is an equivalent extended formulation of the same LP.

In that case,

$$
v(DW\text{-}LP)=v(Original\text{-}LP),
$$

so there is no strengthening of the LP bound.

Finally, a stronger relaxation does not automatically guarantee shorter running time.

The stronger bound must be balanced against the additional computational cost of:

- solving the Restricted Master Problem;
- solving Pricing Problems repeatedly;
- handling degeneracy and stabilization;
- and, in an exact integer algorithm, performing Branch and Price.

Thus:

> **A stronger Dantzig-Wolfe relaxation can substantially accelerate exact optimization, but the overall computational benefit depends on both bound strength and decomposition cost.**

---

## 7. Connection with Lagrangian Relaxation⭐

The previous note introduced Lagrangian Relaxation.

The connection with Dantzig-Wolfe Decomposition is very close.

Consider again

$$
\min \quad \sum_{k=1}^{K}c_k^Tx_k
$$

subject to

$$
\sum_{k=1}^{K}A_kx_k=b
$$

$$
x_k\in X_k.
$$

Dualize the linking constraints using a multiplier vector $\pi$.

The Lagrangian function is

$$
L(x,\pi)=b^T\pi+\sum_{k=1}^{K}(c_k-A_k^T\pi)^Tx_k.
$$

For fixed $\pi$, the Lagrangian relaxation decomposes into independent block problems:

$$
q(\pi)=b^T\pi+\sum_{k=1}^{K}\min_{x_k\in X_k}(c_k-A_k^T\pi)^Tx_k.
$$

Notice the block minimization term:

$$
\min_{x_k\in X_k}(c_k-A_k^T\pi)^Tx_k.
$$

This is essentially the same optimization problem that appears in Dantzig-Wolfe pricing.

Therefore:

```text
Lagrangian Relaxation
→ dualize the linking constraints
→ independent block subproblems

Dantzig-Wolfe / Column Generation
→ dual variables come from the Master Problem
→ independent block Pricing Problems
```

Under the standard polyhedral assumptions and strong LP duality, the Lagrangian dual obtained by dualizing the linking constraints has the same optimal bound as the LP relaxation of the corresponding Dantzig-Wolfe Master Problem [3].

This is one reason Column Generation, Dantzig-Wolfe Decomposition, and Lagrangian Relaxation repeatedly appear together in large-scale optimization.

---

## 8. A Network Optimization Interpretation

The original notes referenced applications in software-defined LEO satellite networks.

These applications provide a useful mental model.

Suppose the original model contains many service flows.

For each flow $k$, local variables may describe:

- VNF placement;
- routing;
- resource usage;
- service-chain order;
- other flow-specific decisions.

The global model also contains shared-resource constraints such as:

- link capacities;
- node capacities;
- computing capacities.

This often creates a structure of the form

```text
flow 1 local constraints
flow 2 local constraints
...
flow K local constraints

          +
shared network resource constraints
```

A Dantzig-Wolfe reformulation can represent one complete feasible local deployment for flow $k$ as one column.

The Master Problem then chooses and coordinates these columns subject to shared network resources.

Conceptually:

```text
one master column
=
one complete feasible local deployment
for one flow
```

This interpretation is closely related to the VNF orchestration application studied by Jia et al. at IEEE ICC 2020, where Dantzig-Wolfe Decomposition, Column Generation, and Branch and Bound are combined for a large-scale ILP [5]. A related VNF service-provision model for software-defined LEO satellite networks appears in [6].

The purpose of these examples is not that every network problem should use Dantzig-Wolfe Decomposition.

The important modeling question is:

> **Can the original variables be partitioned into useful local blocks whose feasible configurations can be generated efficiently while a relatively small set of linking constraints coordinates the blocks?**

If yes, Dantzig-Wolfe Decomposition may be a natural modeling framework.

---

## 9. Dantzig-Wolfe Decomposition, Column Generation, and Branch and Price⭐

These three concepts play different roles.

### Dantzig-Wolfe Decomposition

Reformulates a structured model into a Master Problem whose variables represent complete block configurations.

### Column Generation

Solves the huge LP Master Problem without enumerating all columns.

### Branch and Price

Embeds Column Generation inside Branch and Bound to enforce the required integrality conditions and prove integer optimality.

Therefore,

```text
structured ILP
    ↓
Dantzig-Wolfe reformulation
    ↓
huge column-based integer Master Problem
    ↓
LP relaxation solved by Column Generation
    ↓
if exact integer optimality is required:
Branch and Price
```

This relationship also explains the progression of Notes 05–08:

```text
Note 05: Column Generation
        ↓
how to solve a huge column-based LP

Note 06: Branch and Price
        ↓
how to make Column Generation exact for ILP

Note 07: Lagrangian Relaxation
        ↓
dualize complicating constraints and obtain decomposable bounds

Note 08: Dantzig-Wolfe Decomposition
        ↓
how to systematically create the column-based formulation
```

---

## 10. Key Takeaways

1. Dantzig-Wolfe Decomposition is designed for models with decomposable block structure and a relatively small set of linking constraints.
2. The Minkowski-Weyl theorem allows a polyhedral block to be represented through extreme points and, when necessary, extreme rays.
3. For a finite integer block, a master column can represent one complete integer-feasible local configuration.
4. Substituting these block representations produces a Master Problem with convexity constraints and potentially an enormous number of columns.
5. Column Generation solves the LP Master Problem without explicitly enumerating all columns.
6. The Pricing Problem for block $k$ searches the entire feasible set $X_k$ for a new complete configuration with negative reduced cost.
7. Dantzig-Wolfe reformulation and Column Generation are not the same thing: Dantzig-Wolfe creates the reformulation, while Column Generation solves it.
8. In integer programming, Dantzig-Wolfe convexification of integer-feasible block sets yields an LP relaxation that is no weaker than the corresponding naive compact LP relaxation and can be strictly stronger; for a purely continuous LP, the reformulation is LP-equivalent.
9. Lagrangian Relaxation of the linking constraints and Dantzig-Wolfe pricing lead to closely related block subproblems.
10. For an integer Master Problem, exact integer optimality generally requires an integer method such as Branch and Price.

## References

1. Dantzig, G. B., and Wolfe, P. “Decomposition Principle for Linear Programs.” *Operations Research*, 8(1), 1960, pp. 101–111. DOI: `10.1287/opre.8.1.101`.

2. Schrijver, A. *Theory of Linear and Integer Programming*. Wiley, 1986.

3. Lübbecke, M. E., and Desrosiers, J. “Selected Topics in Column Generation.” *Operations Research*, 53(6), 2005, pp. 1007–1023. DOI: `10.1287/opre.1050.0234`.

4. Vanderbeck, F. “On Dantzig-Wolfe Decomposition in Integer Programming and Ways to Perform Branching in a Branch-and-Price Algorithm.” *Operations Research*, 48(1), 2000, pp. 111–128. DOI: `10.1287/opre.48.1.111.12453`.

5. Jia, Z., Sheng, M., Li, J., Zhu, Y., Bai, W., and Han, Z. “Virtual Network Functions Orchestration in Software Defined LEO Small Satellite Networks.” *2020 IEEE International Conference on Communications (ICC)*, 2020. DOI: `10.1109/ICC40277.2020.9148906`.

6. Jia, Z., Sheng, M., Li, J., Zhou, D., and Han, Z. “VNF-Based Service Provision in Software Defined LEO Satellite Networks.” *IEEE Transactions on Wireless Communications*, 20(9), 2021, pp. 6139–6153. DOI: `10.1109/TWC.2021.3072155`.

## Suggested Follow-up Reading

The most relevant next topics are:

1. **Benders Decomposition**  
   The next decomposition framework in this note series. Unlike Dantzig-Wolfe Decomposition, which is naturally associated with variable/column generation, Benders Decomposition iteratively generates constraints or cuts.

2. **Extended Formulations and Convex Hulls**  
   Useful for understanding why configuration-based formulations can provide stronger LP relaxations in integer programming.

3. **Advanced Column Generation**  
   Includes dual stabilization, degeneracy, multiple-column generation, column management, and advanced pricing strategies.

4. **Branch-and-Price Branching Strategies**  
   Important when the Dantzig-Wolfe Master Problem is integer and exact optimality is required.

5. **Lagrangian Relaxation and Decomposition Duality**  
   Useful for understanding the close connection between dualized linking constraints and Dantzig-Wolfe Pricing Problems.

For this note series, **Benders Decomposition** is the next major topic.
