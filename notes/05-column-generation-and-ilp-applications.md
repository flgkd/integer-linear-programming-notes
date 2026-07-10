# Integer Linear Programming Notes 05: Column Generation and Its Applications in Integer Linear Programming

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

Finally, we arrive at **Column Generation**, one of the most important topics in this series.

Column Generation is easy to misunderstand if the connection among the simplex method, reduced costs, LP duality, and basis changes is not clear. Therefore, before reading this note, it is strongly recommended to understand the following topics:

1. linear programming and LP duality;
2. the basic idea of the simplex method, especially why and how a basis changes;
3. reduced costs and dual variables;
4. the matrix form of the simplex method and reduced costs.

A useful prerequisite is:

- [Simplex Method and Linear Programming Duality](01-simplex-method-and-lp-duality.md)

### 1.1 What Can Column Generation Do?

Before explaining the algorithm itself, let us first answer a basic question:

> What is Column Generation used for?

Column Generation is mainly used to solve **large-scale linear programming problems with an extremely large number of variables, or columns**.

A typical situation is:

- the number of constraints is moderate;
- the number of possible variables is extremely large;
- explicitly enumerating and storing all variables is difficult or impossible.

From the simplex viewpoint, Column Generation can be understood as a way of avoiding the explicit enumeration of all nonbasic variables.

Instead of constructing the full LP first, we start with only a small subset of columns and generate promising new columns when necessary.

Therefore:

> **Standard Column Generation is fundamentally an LP solution framework.**

This point is extremely important.

Column Generation itself does not directly solve an integer linear programming problem to integer optimality.

### 1.2 Why Does Column Generation Matter for ILP?⭐

If Column Generation solves LPs, why is it so important in integer linear programming?

Recall Branch and Bound.

For a maximization ILP:

- the LP relaxation of a node provides an **upper bound**;
- an integer feasible solution provides a **lower bound**.

Therefore, solving LP relaxations is a central operation in Branch and Bound.

Now suppose the LP relaxation itself contains an enormous number of variables. Even before we worry about integrality, explicitly solving the LP relaxation may already be difficult.

This is where Column Generation becomes useful.

> **One major role of Column Generation in large-scale ILP is to solve LP relaxations with a huge number of columns.**

When Column Generation is embedded into Branch and Bound, we obtain **Branch and Price**.

However, there is another important connection between Column Generation and ILP.

In many problems, we can reformulate a compact ILP into a pattern-based, path-based, schedule-based, or configuration-based master problem. Such reformulations may contain an enormous number of variables, but their LP relaxations can be substantially tighter than the LP relaxation of the original compact formulation.

This second role of Column Generation will be discussed in detail near the end of this note.

---

## 2. Basic Idea of Column Generation⭐

### 2.1 Large-Scale Linear Programs with Many Columns

Consider the following LP:

$$
\min \quad \sum_{j=1}^{n} c_j y_j
$$

subject to

$$
\sum_{j=1}^{n} a_{1j}y_j \ge b_1
$$

$$
\sum_{j=1}^{n} a_{2j}y_j \ge b_2
$$

$$
\vdots
$$

$$
\sum_{j=1}^{n} a_{mj}y_j \ge b_m
$$

$$
y_j \ge 0, \quad j=1,\ldots,n.
$$

Suppose

$$
m \ll n
$$

and $n$ is extremely large.

For example, we may have only a few hundred constraints but millions, billions, or even exponentially many possible variables.

The difficulty is not that the LP model is conceptually complicated.

The difficulty is that we may not even be able to explicitly list every column of the coefficient matrix.

### 2.2 A Simplex Viewpoint

Recall the simplex method.

At any basic feasible solution, the number of basic variables is closely related to the number of constraints.

Even if the full LP contains an enormous number of variables, only a relatively small number of variables are basic at a given simplex iteration.

This gives an important intuition:

> Why should we explicitly construct millions of columns if only a small subset of them will actively participate in the current solution?

Column Generation uses exactly this idea.

Instead of solving the full LP with all columns, we:

1. start with a small subset of columns;
2. solve the resulting restricted LP;
3. search for an omitted column with an improving reduced cost;
4. add that column;
5. repeat.

The key difference from a naive search is that we do not inspect every omitted variable one by one.

Instead, we formulate and solve a **pricing problem** to search the entire column space.

### 2.3 Basic Column Generation Procedure

For a minimization problem, the basic process is:

1. construct a Restricted Master Problem using a small set of initial columns;
2. solve the Restricted Master Problem;
3. obtain the dual variables;
4. solve a pricing problem to find a column with negative reduced cost;
5. if such a column exists, add it to the Restricted Master Problem and repeat;
6. if no column with negative reduced cost exists, the current Restricted Master Problem solution is optimal for the full LP master problem.

The logic can be summarized as:

```text
Initial columns
      ↓
Solve RMP
      ↓
Obtain dual variables
      ↓
Solve pricing problem
      ↓
Negative reduced cost?
   /             \
 Yes              No
  ↓                ↓
Add column      LP optimal
  ↓
Solve RMP again
```

---

## 3. Master Problem, Restricted Master Problem, and Pricing Problem

The most important concepts in Column Generation are:

- Master Problem, MP;
- Restricted Master Problem, RMP;
- Pricing Problem, PP, also called the Subproblem, SP.

Let us introduce them carefully.

### 3.1 Master Problem

Consider the LP

$$
\min \quad \sum_{j=1}^{n} c_j y_j
$$

subject to

$$
\sum_{j=1}^{n} a_{ij}y_j \ge b_i,
\quad i=1,\ldots,m
$$

$$
y_j \ge 0,
\quad j=1,\ldots,n.
$$

The full problem containing all possible columns is called the **Master Problem**.

In matrix form:

$$
\min \quad c^T y
$$

subject to

$$
Ay \ge b
$$

$$
y \ge 0.
$$

The difficulty is that $A$ may contain an extremely large number of columns.

### 3.2 Restricted Master Problem

Let

$$
P \subseteq \{1,\ldots,n\}
$$

be a small subset of columns.

The Restricted Master Problem is

$$
\min \quad \sum_{j\in P} c_j y_j
$$

subject to

$$
\sum_{j\in P} a_{ij}y_j \ge b_i,
\quad i=1,\ldots,m
$$

$$
y_j \ge 0,
\quad j\in P.
$$

The RMP contains only a subset of the variables in the full master problem.

From the simplex viewpoint, all omitted variables are temporarily fixed at zero.

The initial column set must usually make the RMP feasible.

Possible ways to obtain initial columns include:

- direct observation;
- a simple heuristic;
- artificial columns with large penalties;
- problem-specific initialization methods.

This is similar in spirit to the need for an initial feasible basis in the simplex method.

### 3.3 Pricing Problem / Subproblem⭐

After solving the RMP, we need to answer the following question:

> Among all columns that are not currently included in the RMP, does there exist a column that can improve the current LP solution?

For a minimization problem, this means finding a column with **negative reduced cost**.

Let $\pi$ be the optimal dual solution associated with the RMP constraints.

For a candidate column $a_j$, the reduced cost is

$$
\overline{c}_j = c_j-\pi^T a_j.
$$

Therefore, the pricing problem searches for

$$
\min_j \left\{c_j-\pi^T a_j\right\}.
$$

If

$$
\overline{c}_j < 0,
$$

then column $j$ can improve the current RMP solution and should be added.

If the minimum reduced cost satisfies

$$
\overline{c}_j \ge 0,
$$

then no omitted column can improve the current solution. The current RMP solution is therefore optimal for the full LP master problem.

This is the core of Column Generation.

> **The pricing problem does not enumerate all columns. It searches the column space through an optimization model.**

### 3.4 Why Dual Variables Are Usually Used

In principle, reduced costs can be computed directly from the simplex basis.

For a candidate column $a_j$,

$$
\overline{c}_j
=
c_j-c_B^T B^{-1}a_j.
$$

Recall that

$$
\pi^T=c_B^T B^{-1}.
$$

Therefore,

$$
\overline{c}_j
=
c_j-\pi^T a_j.
$$

The two expressions are equivalent.

In practice, Column Generation is usually described using dual variables because:

1. explicitly computing $B^{-1}$ is unnecessary;
2. LP solvers directly provide dual values;
3. the pricing objective becomes easier to interpret;
4. the notation is much cleaner in papers and implementations.

This is why dual variables appear almost everywhere in Column Generation.

### 3.5 Column Generation Rules

A candidate column cannot be an arbitrary vector.

It must represent a valid structure in the original problem.

For example:

- in a Cutting Stock Problem, a column represents a feasible cutting pattern;
- in a Vehicle Routing Problem, a column may represent a feasible vehicle route;
- in crew scheduling, a column may represent a feasible crew schedule;
- in network optimization, a column may represent a feasible path, embedding, or configuration.

The conditions that define a valid column will be called the **column generation rules** in this note.

These rules are problem-specific.

For a Cutting Stock Problem, a cutting pattern must satisfy the stock length limitation.

For a routing problem, a route may need to satisfy connectivity, capacity, time-window, or resource constraints.

Therefore:

> **The pricing problem may itself be an integer optimization problem even though the master problem solved by Column Generation is an LP.**

This does not mean Column Generation directly solves an ILP.

It means that the space of valid LP columns is described by a discrete combinatorial structure.

### 3.6 An Important Note on Heuristic Pricing⭐

To generate a useful column, we do not always need to find the most negative reduced cost.

Any column satisfying

$$
\overline{c}_j < 0
$$

can be added to the RMP.

This is why heuristics are often useful in pricing.

However, there is one important distinction:

> A heuristic pricing method can find improving columns, but failure of the heuristic to find one does not prove that no improving column exists.

Therefore, an exact Column Generation algorithm usually needs an exact pricing step, or another valid certificate, before declaring convergence.

A common strategy is:

1. run fast heuristic pricing first;
2. add good columns if found;
3. when the heuristic fails, solve the exact pricing problem;
4. terminate only when exact pricing confirms that no negative reduced-cost column exists.

### 3.7 Column Generation Algorithm

The basic algorithm for a minimization LP is:

```text
ColumnGeneration:

    construct a feasible initial RMP

    repeat:

        solve the RMP

        obtain optimal dual variables pi

        solve the pricing problem

        find a column with minimum reduced cost

        if reduced_cost < 0:
            add the column to the RMP
        else:
            stop

    return the optimal RMP solution
```

---

## 4. Cutting Stock Problem

The Cutting Stock Problem is one of the classical examples of Column Generation.

### 4.1 Problem Description

Suppose a factory has large stock rolls of fixed length

$$
W.
$$

Customers request smaller pieces of different lengths.

For item type $i$:

- the required length is $w_i$;
- the demand is $d_i$.

The goal is to cut the large stock rolls so that all customer demands are satisfied while minimizing the number of stock rolls used, or equivalently reducing material waste.

The key difficulty is that there may be an enormous number of possible cutting patterns.

### 4.2 A Direct Formulation

A direct formulation may explicitly model individual stock rolls and the number of items cut from each roll.

Such a formulation is intuitive.

However, it can have two major problems:

1. the formulation may be large;
2. its LP relaxation may be weak.

This motivates a different modeling idea.

### 4.3 Pattern-Based Formulation⭐

Instead of modeling each physical stock roll directly, consider all feasible cutting patterns.

Let

$$
P
$$

be the set of all feasible cutting patterns.

For pattern $j$:

$$
a_{ij}
$$

is the number of pieces of item type $i$ produced by pattern $j$.

Let

$$
x_j
$$

be the number of times pattern $j$ is used.

The integer master formulation is

$$
\min \quad \sum_{j\in P} x_j
$$

subject to

$$
\sum_{j\in P} a_{ij}x_j \ge d_i,
\quad i=1,\ldots,m
$$

$$
x_j \in \mathbf{Z}_+,
\quad j\in P.
$$

Each column

$$
a_j=
\begin{bmatrix}
a_{1j}\\
a_{2j}\\
\vdots\\
a_{mj}
\end{bmatrix}
$$

represents one feasible cutting pattern.

A pattern is feasible only if

$$
\sum_{i=1}^{m} w_i a_{ij}\le W
$$

and

$$
a_{ij}\in\mathbf{Z}_+.
$$

The formulation is conceptually simple.

The problem is that the number of feasible patterns can be enormous.

This is exactly the type of structure where Column Generation becomes useful.

### 4.4 LP Master Problem

Standard Column Generation solves the LP relaxation of the pattern-based master problem:

$$
\min \quad \sum_{j\in P} x_j
$$

subject to

$$
\sum_{j\in P} a_{ij}x_j \ge d_i,
\quad i=1,\ldots,m
$$

$$
x_j \ge 0,
\quad j\in P.
$$

We call this the LP Master Problem.

Its solution may be fractional.

That is expected.

Column Generation is being used to solve the LP relaxation.

### 4.5 Restricted LP Master Problem

Choose a small subset

$$
P' \subset P
$$

such that the resulting RMP is feasible.

Then solve

$$
\min \quad \sum_{j\in P'} x_j
$$

subject to

$$
\sum_{j\in P'} a_{ij}x_j \ge d_i,
\quad i=1,\ldots,m
$$

$$
x_j \ge 0,
\quad j\in P'.
$$

Let

$$
\pi_i
$$

be the dual variable associated with demand constraint $i$.

The dual of the RMP is

$$
\max \quad \sum_{i=1}^{m} d_i\pi_i
$$

subject to

$$
\sum_{i=1}^{m} a_{ij}\pi_i \le 1,
\quad j\in P'
$$

$$
\pi_i\ge 0,
\quad i=1,\ldots,m.
$$

### 4.6 Reduced Cost of a Cutting Pattern

The objective coefficient of every pattern variable $x_j$ is 1.

Therefore, the reduced cost of a candidate pattern is

$$
\overline{c}_j
=
1-\sum_{i=1}^{m}\pi_i a_{ij}.
$$

For a minimization problem, we want

$$
\overline{c}_j<0.
$$

Equivalently,

$$
\sum_{i=1}^{m}\pi_i a_{ij}>1.
$$

Therefore, the pricing problem can be written as

$$
\max \quad \sum_{i=1}^{m}\pi_i a_i
$$

subject to

$$
\sum_{i=1}^{m}w_i a_i\le W
$$

$$
a_i\in\mathbf{Z}_+,
\quad i=1,\ldots,m.
$$

This is an **integer knapsack problem**.

Let the optimal pricing objective be

$$
\theta^{opt}.
$$

Then the minimum reduced cost is

$$
1-\theta^{opt}.
$$

If

$$
\theta^{opt}>1,
$$

we have found a negative reduced-cost column.

If

$$
\theta^{opt}\le 1,
$$

no improving column exists and Column Generation terminates.

### 4.7 Why Is the Pricing Problem an ILP?⭐

This is an important point.

The master problem currently being solved by Column Generation is an LP.

However, the pricing problem is an integer knapsack problem.

Why?

Because a column represents a cutting pattern.

For example,

$$
a_i
$$

means the number of pieces of item type $i$ cut from one stock roll.

Naturally,

$$
a_i
$$

must be an integer.

Therefore, the pricing problem contains integer variables.

This leads to an important conclusion:

> **The pricing problem can be an ILP because the column generation rules are discrete. This does not change the fact that standard Column Generation is solving an LP master problem.**

In the Cutting Stock Problem, the pricing problem happens to be a knapsack problem.

Although knapsack is NP-hard, it can be solved by dynamic programming in pseudo-polynomial time when the capacity is represented numerically and is not too large.

### 4.8 Column Generation for Cutting Stock

The process is:

1. construct a feasible initial set of cutting patterns;
2. solve the RMP;
3. obtain the dual variables $\pi$;
4. solve the knapsack pricing problem;
5. if the optimal pricing value is greater than 1, add the generated pattern;
6. repeat;
7. stop when the pricing value is no greater than 1.

---

## 5. A Detailed Cutting Stock Example⭐

Consider stock rolls of length

$$
W=16.
$$

Customers require:

- 25 pieces of length 3;
- 20 pieces of length 6;
- 18 pieces of length 7.

We want to minimize the number of stock rolls used.

### 5.1 Full Master Problem

Let $P$ be the set of all feasible cutting patterns.

For pattern $j$,

$$
a_{1j}
$$

is the number of 3-unit pieces,

$$
a_{2j}
$$

is the number of 6-unit pieces,

and

$$
a_{3j}
$$

is the number of 7-unit pieces.

Let

$$
y_j
$$

be the number of times pattern $j$ is used.

The integer master problem is

$$
\min \quad \sum_{j\in P} y_j
$$

subject to

$$
\sum_{j\in P}a_{1j}y_j\ge 25
$$

$$
\sum_{j\in P}a_{2j}y_j\ge 20
$$

$$
\sum_{j\in P}a_{3j}y_j\ge 18
$$

$$
y_j\in\mathbf{Z}_+.
$$

Every pattern must satisfy

$$
3a_{1j}+6a_{2j}+7a_{3j}\le 16
$$

$$
a_{1j},a_{2j},a_{3j}\in\mathbf{Z}_+.
$$

### 5.2 Initial RMP

Choose three simple cutting patterns:

$$
a_1=
\begin{bmatrix}
5\\
0\\
0
\end{bmatrix},
\quad
a_2=
\begin{bmatrix}
0\\
2\\
0
\end{bmatrix},
\quad
a_3=
\begin{bmatrix}
0\\
0\\
2
\end{bmatrix}.
$$

The initial LP RMP is

$$
\min \quad y_1+y_2+y_3
$$

subject to

$$
5y_1\ge 25
$$

$$
2y_2\ge 20
$$

$$
2y_3\ge 18
$$

$$
y_1,y_2,y_3\ge 0.
$$

### 5.3 Iteration 1

Solving the initial RMP gives the dual solution

$$
\pi=
\begin{bmatrix}
0.2\\
0.5\\
0.5
\end{bmatrix}.
$$

The pricing problem is

$$
\max \quad 0.2a_1+0.5a_2+0.5a_3
$$

subject to

$$
3a_1+6a_2+7a_3\le 16
$$

$$
a_1,a_2,a_3\in\mathbf{Z}_+.
$$

An optimal pricing solution is

$$
a_4=
\begin{bmatrix}
1\\
2\\
0
\end{bmatrix}.
$$

Its pricing objective is

$$
0.2+2(0.5)=1.2.
$$

Therefore, its reduced cost is

$$
\overline{c}_4=1-1.2=-0.2.
$$

Since

$$
\overline{c}_4<0,
$$

add column $a_4$ to the RMP.

### 5.4 Iteration 2

The new RMP is

$$
\min \quad y_1+y_2+y_3+y_4
$$

subject to

$$
5y_1+y_4\ge 25
$$

$$
2y_2+2y_4\ge 20
$$

$$
2y_3\ge 18
$$

$$
y_1,y_2,y_3,y_4\ge 0.
$$

Solving the RMP gives

$$
\pi=
\begin{bmatrix}
0.2\\
0.4\\
0.5
\end{bmatrix}.
$$

The pricing problem becomes

$$
\max \quad 0.2a_1+0.4a_2+0.5a_3
$$

subject to

$$
3a_1+6a_2+7a_3\le 16
$$

$$
a_1,a_2,a_3\in\mathbf{Z}_+.
$$

An optimal solution is

$$
a_5=
\begin{bmatrix}
1\\
1\\
1
\end{bmatrix}.
$$

The pricing objective is

$$
0.2+0.4+0.5=1.1.
$$

Thus,

$$
\overline{c}_5=1-1.1=-0.1.
$$

Add column $a_5$ to the RMP.

### 5.5 Iteration 3

The RMP now contains five columns.

Solving it gives

$$
\pi=
\begin{bmatrix}
0.2\\
0.4\\
0.4
\end{bmatrix}.
$$

The pricing problem is

$$
\max \quad 0.2a_1+0.4a_2+0.4a_3
$$

subject to

$$
3a_1+6a_2+7a_3\le 16
$$

$$
a_1,a_2,a_3\in\mathbf{Z}_+.
$$

The optimal pricing value is

$$
1.
$$

Therefore, the minimum reduced cost is

$$
1-1=0.
$$

No negative reduced-cost column exists.

Column Generation terminates.

### 5.6 Final LP Solution

The optimal solution of the final RMP is

$$
y=
\begin{bmatrix}
1.2\\
0\\
0\\
1\\
18
\end{bmatrix}.
$$

The objective value is

$$
20.2.
$$

Notice that

$$
y_1=1.2.
$$

This is not an integer.

There is no contradiction.

We relaxed the integer master problem before applying Column Generation.

Therefore, the result is the optimal solution of the **LP relaxation**, not the optimal integer solution of the original Cutting Stock Problem.

If we solve the integer restricted master problem using the five generated columns, one integer solution is

$$
y=
\begin{bmatrix}
2\\
0\\
0\\
1\\
18
\end{bmatrix}
$$

with objective value

$$
21.
$$

For this small example, this solution is also optimal for the full integer pattern formulation.

However, this last observation should not be generalized carelessly.

This leads to the most important part of this note.

---

## 6. How Column Generation Is Used in Integer Linear Programming⭐

We have repeatedly emphasized:

> Standard Column Generation solves an LP master problem.

Then how can Column Generation be used in ILP?

There are several different approaches.

### 6.1 Approach I: Solve the LP Relaxation and Round

The simplest approach is:

1. relax the ILP;
2. solve the LP relaxation by Column Generation;
3. round or otherwise repair the fractional solution.

This is only a heuristic.

As discussed in the Branch and Bound note:

- rounding may destroy feasibility;
- a feasible rounded solution may have poor quality.

Therefore, this approach is simple but unreliable.

### 6.2 Approach II: Branch and Price

The exact approach is to embed Column Generation into Branch and Bound.

At every branch-and-bound node:

1. solve the node LP relaxation by Column Generation;
2. use the LP value as the node bound;
3. branch if necessary;
4. repeat.

This framework is called **Branch and Price**.

For a maximization problem:

- Column Generation solves the LP relaxation and provides an upper bound;
- integer feasible solutions provide lower bounds;
- Branch and Bound manages the search tree.

If implemented correctly with exact pricing and valid branching, Branch and Price can prove integer optimality.

This topic will be introduced in the next note.

### 6.3 Approach III: Generate Columns at the Root and Solve the Restricted Integer Master

Another practical approach is:

1. reformulate the original ILP as a column-based integer master problem;
2. relax the master problem;
3. run Column Generation until the LP relaxation is solved;
4. restore integrality on the generated master variables;
5. solve the resulting restricted integer master problem using Branch and Bound, a solver, or a heuristic.

This approach is often computationally attractive.

However:

> **It is generally not an exact method.**

The reason is subtle but very important.

Column Generation terminates when no omitted column has negative reduced cost for the **LP relaxation**.

This only proves that the current column set is sufficient for the LP optimum.

It does not prove that the current column set contains every column needed by the optimal integer solution.

A column with nonnegative reduced cost at the LP optimum may still be useful in a better integer combination.

Therefore, solving the integer RMP after root-node Column Generation may produce a suboptimal integer solution.

This explains a phenomenon that is sometimes confusing:

> Even after Column Generation has fully converged for the LP, solving the integer RMP over the generated columns may still fail to recover the optimal solution of the full integer master problem.

The issue is not that the solver failed.

The issue is that the restricted integer master may be missing columns that are irrelevant to LP improvement but important for integer combinatorial structure.

This approach is better understood as a **CG-based integer heuristic** or a **price-and-branch style approach**, unless additional procedures are used to guarantee exactness.

### 6.4 Why Reformulate the Original ILP?⭐

Suppose the original compact integer formulation is

$$
OP.
$$

Through a problem-specific reformulation or Dantzig-Wolfe decomposition, we obtain an equivalent integer master formulation

$$
MP.
$$

At the integer level, $MP$ can represent the same feasible solutions as $OP$.

Why do this if $MP$ contains far more variables?

Because the LP relaxation of the reformulated master problem can be much tighter.

The important relationship is often

$$
LP(MP)
$$

provides a stronger bound than

$$
LP(OP).
$$

The reason is that a column in $MP$ may already represent a complete feasible substructure.

For example:

- a feasible cutting pattern;
- a feasible vehicle route;
- a feasible crew schedule;
- a feasible network embedding;
- a feasible configuration.

The pricing problem enforces the internal feasibility of each column.

Therefore, part of the combinatorial structure of the original ILP is already captured inside the columns.

This leads to one of the most important conclusions in this note:

> **An appropriate Dantzig-Wolfe or column-based reformulation can have a substantially tighter LP relaxation than the original compact ILP formulation.**

The price is that the reformulation may contain an enormous number of columns.

Column Generation is the mechanism that makes such a formulation computationally usable.

This gives the complete picture:

```text
Compact ILP
    ↓
Column-based / Dantzig-Wolfe reformulation
    ↓
Potentially stronger LP relaxation
    ↓
Enormously many columns
    ↓
Column Generation
```

### 6.5 A General Workflow for Large-Scale ILP

A general workflow is:

#### Step 1: Reformulate the Original ILP

Convert the original problem into a column-based master formulation.

This may be derived through:

- Dantzig-Wolfe decomposition;
- a path-based formulation;
- a pattern-based formulation;
- a schedule-based formulation;
- a configuration-based formulation;
- direct problem-specific modeling.

The resulting integer master problem should represent the original problem correctly.

#### Step 2: Construct an Initial Restricted Master Problem

Select a small feasible set of columns.

The restricted master problem is still an integer formulation conceptually.

#### Step 3: Relax the Restricted Master Problem

Relax the integer master variables.

The resulting problem is an LP RMP.

#### Step 4: Solve the LP RMP and Pricing Problems

Solve the LP RMP and obtain dual variables.

Use these dual variables to construct the pricing problem.

The pricing problem may be:

- an LP;
- an ILP;
- a shortest path problem;
- a resource-constrained shortest path problem;
- a knapsack problem;
- another combinatorial optimization problem.

If the decomposition creates multiple independent pricing problems, they can often be solved in parallel.

To generate columns, it is sufficient to find improving columns.

However, to certify LP convergence, exact pricing is eventually required unless another valid optimality certificate is available.

If an improving column is found, add it and repeat.

#### Step 5: Recover an Integer Solution

After solving the LP relaxation, possible approaches include:

1. Branch and Price for exact integer optimization;
2. solve the restricted integer master as a heuristic;
3. use Branch and Bound on the restricted integer master;
4. use a commercial MILP solver on the restricted integer master;
5. design problem-specific rounding, repair, or primal heuristics.

The choice depends on whether exact optimality is required and how large the problem is.

### 6.6 A Useful Idea for Heuristic Design

There is another useful lesson.

When the full optimization problem is extremely complicated, designing a heuristic directly for the entire problem may be difficult.

However, the pricing problem often has a much clearer problem-specific meaning.

Therefore:

> Instead of designing a heuristic for the entire ILP, it may be easier to design a heuristic for the pricing problem.

For example, a pricing heuristic may exploit:

- path structure;
- network topology;
- resource ordering;
- local feasibility;
- domain-specific greedy rules.

This can make column generation much faster.

Again, heuristic pricing is excellent for discovering useful columns.

But an exact pricing step is still needed if we want to prove LP optimality.

---

## 7. Column Generation and the Simplex Method: A Final Thought

A natural question is:

> Why does the ordinary simplex method not always use Column Generation?

The answer is that Column Generation is useful only when the column space has a structure that can be searched efficiently through a pricing problem.

If all columns are already explicitly available and the LP is of manageable size, ordinary simplex methods can directly evaluate reduced costs and perform basis updates.

Column Generation becomes attractive when:

1. the number of possible columns is too large to enumerate;
2. valid columns have an implicit mathematical structure;
3. the minimum reduced-cost column can be found by solving a structured pricing problem.

Therefore, the real power of Column Generation does not come only from "adding variables gradually."

Its real power comes from:

> **replacing explicit enumeration of columns with optimization over the column space.**

---

## 8. Key Takeaways

1. Standard Column Generation is fundamentally an LP solution framework.
2. It is especially useful for LPs with an enormous number of variables or columns.
3. Column Generation solves a Restricted Master Problem and generates improving columns through a pricing problem.
4. Reduced costs connect the RMP dual variables to the pricing problem.
5. For a minimization problem, a column with negative reduced cost can improve the current RMP solution.
6. A pricing problem may itself be an ILP because valid columns can have discrete combinatorial structure.
7. Heuristic pricing can find useful columns, but exact pricing is needed to certify LP convergence.
8. In the Cutting Stock Problem, the pricing problem is an integer knapsack problem.
9. Column Generation can be embedded into Branch and Bound to form Branch and Price.
10. Solving the integer RMP after root-node Column Generation is generally a heuristic and may miss the full integer optimum.
11. A column-based or Dantzig-Wolfe reformulation can provide a much tighter LP relaxation than a compact ILP formulation.
12. The real power of Column Generation is replacing explicit column enumeration with optimization over the column space.

## References

1. Dantzig, G. B., and Wolfe, P. “Decomposition Principle for Linear Programs.” *Operations Research*, 8(1), 1960, pp. 101–111.

2. Gilmore, P. C., and Gomory, R. E. “A Linear Programming Approach to the Cutting-Stock Problem.” *Operations Research*, 9(6), 1961, pp. 849–859.

3. Barnhart, C., Johnson, E. L., Nemhauser, G. L., Savelsbergh, M. W. P., and Vance, P. H. “Branch-and-Price: Column Generation for Solving Huge Integer Programs.” *Operations Research*, 46(3), 1998, pp. 316–329.

4. Lübbecke, M. E., and Desrosiers, J. “Selected Topics in Column Generation.” *Operations Research*, 53(6), 2005, pp. 1007–1023.

## Suggested Follow-up Reading

- Dantzig-Wolfe Decomposition
- Pricing Problems
- Dual Stabilization
- Degeneracy in Column Generation
- Cutting Stock Problem
- Knapsack Problem
- Branch and Price
- Heuristic Pricing
- Parallel Pricing
- Learning-Assisted Column Generation

These topics are useful for later notes on Branch and Price, Dantzig-Wolfe decomposition, and large-scale network optimization.
