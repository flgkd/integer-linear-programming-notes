# Integer Linear Programming Notes 14: Convex Relaxation of Structured Binary Decisions via Mixed Norms

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

Many ILP and MILP models contain binary variables whose purpose is not to represent an arbitrary integer quantity, but to answer a structural question:

> **Should this resource, node, subsystem, feature group, server, sensor, or communication component be active at all?**

When this binary decision is tightly coupled to a group of continuous variables, the activation state can sometimes be represented indirectly through the **zero/nonzero support** of that continuous group.

This creates an important alternative to explicit integer optimization:

> **replace a combinatorial group-selection term by a convex mixed-norm penalty that encourages entire groups of continuous variables to become zero.**

The resulting technique is closely related to **group sparsity**, **Group LASSO**, and **mixed-norm convex regularization** [3-6].

This note studies the method as a reusable optimization tool. The main questions are:

1. When can a structured ILP or MILP be transformed into a convex mixed-norm surrogate?
2. What mathematical structure makes the transformation possible?
3. Why can a mixed norm make an entire group exactly zero?
4. What advantages are obtained after convexification?
5. What optimality guarantees are lost?
6. Which integer problems are suitable, and which are not?
7. How can the method be combined with exact MILP techniques?

The method was motivated by the group-sparse optimization construction in [2], but the discussion below is deliberately problem-independent.

The most important point is:

> **Mixed-norm convex relaxation can turn some structured binary activation problems into tractable convex optimization problems, but it is not a universal exact convexification of arbitrary ILPs.**

---

## 2. Can an ILP or MILP Be Converted into a Convex Problem?⭐⭐⭐

### 2.1 Short Answer

For **some structured ILP or MILP problems**, yes.

A particularly favorable case occurs when:

- the integer variables are binary activation indicators;
- each binary indicator corresponds to a meaningful group of continuous variables;
- an inactive resource is equivalent to that entire continuous group being zero;
- after removing the activation indicators, the remaining continuous model is convex.

In this setting, the binary activation pattern can be represented by group support and approximated through a convex mixed norm.

The general transformation is

```text
binary activation decision
→ zero/nonzero support of a continuous group
→ group-cardinality model
→ mixed-norm convex surrogate
→ convex optimization
```

However, the step

```text
group cardinality
→ mixed norm
```

is generally a **surrogate approximation**, not an exact equivalence.

Therefore:

> **the method can convert certain structured ILP/MILP models into convex surrogate problems, but it cannot exactly convert arbitrary ILPs into convex problems.**

### 2.2 Why There Cannot Be a Universal Exact Conversion

General ILP is NP-hard [1].

Suppose every ILP could be transformed, in polynomial time and with polynomial model size, into an equivalent convex optimization problem that could then be solved in polynomial time.

Then arbitrary ILPs could be solved in polynomial time.

Unless

$$
P=NP,
$$

such a universal exact transformation cannot exist.

So the correct question is not:

> “Can mixed norms convexify every ILP?”

The useful question is:

> **“Does my ILP contain a special group-activation structure whose discrete decisions can be represented by the support of continuous variables?”**

---

## 3. The Core Modeling Structure: Binary Activation as Group Support⭐⭐⭐

### 3.1 A Generic Activation MILP

Suppose the continuous variables are divided into $G$ groups:

$$
x_g\in\mathbb{R}^{d_g},\quad g=1,\ldots,G.
$$

Introduce a binary activation variable

$$
y_g\in\{0,1\}.
$$

A typical interpretation is

```text
y_g = 0  → resource g is inactive
y_g = 1  → resource g is available
```

The activation variable is linked to the continuous group through constraints such as

$$
-M_{gj}y_g\le x_{gj}\le M_{gj}y_g,\quad j=1,\ldots,d_g.
$$

If the continuous variables are nonnegative, this becomes

$$
0\le x_{gj}\le M_{gj}y_g.
$$

A generic fixed-charge MILP is

$$
\min_{x,y}\quad f(x)+\sum_{g=1}^{G}F_gy_g
$$

subject to the continuous constraints and

$$
y_g\in\{0,1\},\quad g=1,\ldots,G.
$$

Here $F_g>0$ is the cost of activating group $g$.

### 3.2 The Critical Equivalence

The mixed-norm idea becomes natural when the physical meaning satisfies

$$
y_g=0\Longleftrightarrow x_g=0.
$$

Equivalently,

$$
y_g=1\Longleftrightarrow\|x_g\|_2>0.
$$

This is the key bridge.

Examples include:

- no traffic through a node $\Longleftrightarrow$ the node is unused;
- all beamforming coefficients of a base station are zero $\Longleftrightarrow$ the base station is inactive;
- all workloads assigned to a server are zero $\Longleftrightarrow$ the server is unused;
- all coefficients of a feature group are zero $\Longleftrightarrow$ that feature group is excluded.

### 3.3 When Can the Binary Variable Be Eliminated?

Suppose $y_g$ appears only as an activation indicator:

- it links the continuous variables to an on/off decision;
- it contributes a positive activation cost;
- it does not participate in other logical constraints.

Then at an optimal solution,

$$
y_g=\mathbf{1}\{\|x_g\|_2>0\}.
$$

The reason is simple:

- if $x_g\ne0$, the linking constraints force $y_g=1$;
- if $x_g=0$, choosing $y_g=0$ avoids paying an unnecessary positive activation cost.

The binary variable can therefore be eliminated conceptually.

This does **not** work automatically when $y_g$ is also involved in constraints such as:

```text
y_1 + y_2 = 1
y_3 ≤ y_4
y_5 + y_6 ≤ 1
```

Those constraints contain genuine Boolean structure that cannot be represented merely by the magnitude of $x_g$.

---

## 4. From Integer Decisions to Group Cardinality⭐⭐⭐

### 4.1 The Mixed $\ell_{2,0}$ Quantity

Define

$$
\|x\|_{2,0}=\sum_{g=1}^{G}\mathbf{1}\{\|x_g\|_2>0\}.
$$

This counts the number of active groups.

Despite the notation, $\|\cdot\|_{2,0}$ is not a norm.

It is a discontinuous, nonconvex group-cardinality measure.

With group-dependent activation costs,

$$
\sum_{g=1}^{G}F_g\mathbf{1}\{\|x_g\|_2>0\}
$$

is the exact weighted activation cost.

The original binary model can therefore be viewed as the continuous but nonconvex problem

$$
\min_{x\in\mathcal{C}}\quad f(x)+\sum_{g=1}^{G}F_g\mathbf{1}\{\|x_g\|_2>0\},
$$

where $\mathcal{C}$ contains the remaining continuous constraints.

This reformulation does not yet make the problem easier.

The combinatorial difficulty has simply moved from explicit binary variables into the support-counting term.

### 4.2 The Real Source of Difficulty

This perspective reveals an important modeling principle:

> **for activation-type MILPs, the hard discrete object is often the support pattern of continuous groups rather than the numerical symbols 0 and 1 themselves.**

This is exactly why sparse optimization becomes relevant.

---

## 5. Convex Relaxation with a Mixed $\ell_2/\ell_1$ Norm⭐⭐⭐

### 5.1 Replacing Group Cardinality

The group-cardinality term can be approximated by the weighted mixed norm

$$
\|x\|_{2,1,\omega}=\sum_{g=1}^{G}\omega_g\|x_g\|_2,
$$

with

$$
\omega_g>0.
$$

With equal weights,

$$
\|x\|_{2,1}=\sum_{g=1}^{G}\|x_g\|_2.
$$

This is the basic Group LASSO regularizer [3].

The combinatorial model

$$
\min_{x\in\mathcal{C}}\quad f(x)+\lambda\|x\|_{2,0}
$$

is replaced by the convex surrogate

$$
\min_{x\in\mathcal{C}}\quad f(x)+\lambda\sum_{g=1}^{G}\omega_g\|x_g\|_2.
$$

For a maximization problem with utility $U(x)$, the corresponding form is

$$
\max_{x\in\mathcal{C}}\quad U(x)-\lambda\sum_{g=1}^{G}\omega_g\|x_g\|_2.
$$

Group-sparse formulations of this type are widely used in statistical learning, communications, signal processing, and network optimization [2-6].

### 5.2 Why Is It Called a Mixed Norm?

The regularizer contains two levels.

Inside each group:

$$
\|x_g\|_2=\sqrt{\sum_{j=1}^{d_g}x_{gj}^2}.
$$

Across groups:

$$
\sum_{g=1}^{G}\omega_g\|x_g\|_2.
$$

So:

- the inner $\ell_2$ norm treats all variables of one group as a single structural unit;
- the outer $\ell_1$ aggregation encourages sparsity across those group magnitudes.

This is why the construction is often described as an $\ell_2/\ell_1$ or $\ell_{2,1}$ mixed norm.

### 5.3 Why Not Use Ordinary $\ell_1$?

An ordinary coordinatewise $\ell_1$ penalty is

$$
\sum_{g=1}^{G}\sum_{j=1}^{d_g}|x_{gj}|.
$$

It encourages individual coordinates to become zero.

But the activation decision concerns the entire group.

The mixed norm instead encourages

$$
x_g=0
$$

for complete groups.

This distinction is fundamental.

### 5.4 When Is the New Problem Convex?

For the minimization form, the mixed-norm model is convex if:

1. $f(x)$ is convex;
2. $\mathcal{C}$ is a convex feasible set;
3. the weights satisfy $\omega_g>0$.

For the maximization form, $U(x)$ should be concave and $\mathcal{C}$ should be convex.

Therefore the method does not create convexity from nothing.

It removes one particular source of nonconvexity — group activation — only when the remaining continuous core is already convex or has been convexified separately.

---

## 6. Why Can Entire Groups Become Exactly Zero?⭐⭐⭐

### 6.1 Intuition: A Nonsmooth Corner at Zero

Suppose a resource carries only a tiny amount of activity.

Without a sparsity penalty, the optimizer may keep that tiny value because there is no reason to eliminate it completely.

With

$$
\lambda\|x_g\|_2,
$$

the objective has a nonsmooth point at

$$
x_g=0.
$$

That nonsmooth point acts as a threshold.

If the benefit of keeping the group active is too small, the optimal solution can move the entire group exactly to zero.

### 6.2 Subgradient Explanation

Consider

$$
F(x)=f(x)+\lambda\sum_{g=1}^{G}\omega_g\|x_g\|_2,
$$

where $f$ is differentiable and convex.

For $x_g\ne0$,

$$
\partial\|x_g\|_2=\{\frac{x_g}{\|x_g\|_2}\}.
$$

At zero,

$$
\partial\|x_g\|_2=\{u:\|u\|_2\le1\}.
$$

For an unconstrained problem, the first-order optimality condition for

$$
x_g^{\ast}=0
$$

can hold whenever

$$
\|\nabla_gf(x^{\ast})\|_2\le\lambda\omega_g.
$$

Interpretation:

> if the potential improvement from activating group $g$ is not large enough to overcome the sparsity penalty, the entire group can optimally remain zero.

With convex constraints, a normal-cone term enters the optimality condition, but the same group-sparsity mechanism remains.

### 6.3 Group Soft Thresholding⭐⭐⭐

The mechanism becomes even clearer through the proximal operator.

For

$$
h(x_g)=\lambda\omega_g\|x_g\|_2,
$$

the proximal mapping is

$$
\mathrm{prox}_{th}(v_g)=(1-\frac{t\lambda\omega_g}{\|v_g\|_2})_+v_g,
$$

where

$$
(a)_+=\max(a,0).
$$

If

$$
\|v_g\|_2\le t\lambda\omega_g,
$$

then

$$
\mathrm{prox}_{th}(v_g)=0.
$$

The entire group disappears in one operation.

This is **group soft thresholding** or **block soft thresholding** [6].

It is fundamentally different from arbitrary rounding.

---

## 7. Why This Is More Than Naive Binary Relaxation⭐⭐⭐

A naive LP relaxation changes

$$
y_g\in\{0,1\}
$$

into

$$
0\le y_g\le1.
$$

The resulting model may produce

$$
y_g=0.37.
$$

Nothing in the relaxation necessarily prefers the endpoints 0 and 1.

Mixed-norm modeling follows a different strategy:

```text
do not approximate the binary value itself
→ represent activation by group support
→ penalize nonzero support through a sparsity-inducing convex function
```

The distinction can be summarized as follows.

| Aspect | Naive Binary Relaxation | Mixed-Norm Group-Sparse Method |
|:---:|:---|:---|
| What is relaxed? | binary indicator value | group-cardinality structure |
| Main variable | $y_g\in[0,1]$ | continuous vector $x_g$ |
| Structural signal | fractional indicator | zero/nonzero group support |
| Sparsity mechanism | usually none | nonsmooth mixed norm |
| Typical behavior | fractional activations | complete groups may become exactly zero |
| Convexity | often convex if the remaining model is linear | convex if the continuous core is convex |
| Exact original integer optimum | not guaranteed | not guaranteed |

The method should therefore be understood as **support-based convexification**, not as simple fractional relaxation.

---

## 8. What Does the Convexification Guarantee?⭐⭐⭐

### 8.1 Global Optimum of the Convex Problem

If

$$
\min_{x\in\mathcal{C}}\quad f(x)+\lambda\sum_g\omega_g\|x_g\|_2
$$

is convex, then a correct convex optimization algorithm can find a globally optimal solution of this model.

That is a strong guarantee.

### 8.2 But Not Automatically the Original ILP Optimum

The convex mixed-norm objective and the original activation objective are different.

The exact fixed activation cost is

$$
F_g\mathbf{1}\{\|x_g\|_2>0\}.
$$

The mixed-norm penalty is

$$
\lambda\omega_g\|x_g\|_2.
$$

The first depends only on whether the group is active.

The second also depends on how large the active group is.

Therefore:

> **global optimality of the convex surrogate does not automatically imply global optimality of the original ILP or MILP.**

### 8.3 Sparsity Is Not the Same as Exact Support Recovery

Two questions must be separated.

**Question 1: Does the method produce many zero groups?**

Often yes. That is the purpose of the mixed norm.

**Question 2: Are those exactly the same groups selected by the optimal integer solution?**

Not in general.

Exact support recovery can occur under special structural assumptions, but it is not a universal property of mixed-norm relaxation.

### 8.4 Why the Surrogate Can Still Be Valuable

In many engineering problems, the objective is not necessarily to certify the exact combinatorial activation pattern.

The real goal may be:

> **obtain a high-quality solution with a small active set fast enough to be useful.**

For that objective, a convex surrogate can be extremely attractive.

---

## 9. Advantages of the Method⭐⭐⭐

### 9.1 Eliminating Explicit Binary Activation Variables

If $G$ activation binaries are successfully replaced by group support, the convex surrogate no longer explicitly searches among

$$
2^G
$$

binary patterns.

This does not mean the convex problem is free, but it removes the combinatorial branching caused by those activation variables.

### 9.2 Polynomial-Time Solvable Convex Structure

When the resulting model is a standard convex problem, it can be attacked with mature convex optimization methods.

Depending on the structure, suitable approaches include:

- interior-point methods;
- proximal gradient methods;
- accelerated first-order methods;
- primal-dual methods;
- ADMM-type methods when their assumptions hold;
- block-coordinate and BSUM-type methods [6-8].

### 9.3 Natural Group-Level Selection

The method directly produces structural information:

```text
x_g = 0  → resource g inactive
x_g ≠ 0  → resource g active
```

This avoids an arbitrary post-processing rule such as thresholding a relaxed binary at $0.5$.

### 9.4 A Continuous Performance-Sparsity Trade-off

The parameter $\lambda$ controls how aggressively sparsity is encouraged.

At a high level:

```text
small λ  → prioritize the original performance objective
large λ  → prioritize fewer or weaker active groups
```

This creates a natural trade-off between performance and structural simplicity.

### 9.5 Exploiting Large-Scale Structure

Once the model is convex and group separable, it may support:

- local block updates;
- closed-form proximal mappings;
- parallelization;
- distributed implementations;
- sparse linear algebra.

This can make the approach substantially more scalable than solving a large activation MILP directly.

---

## 10. When Can This Method Be Used?⭐⭐⭐

The following conditions provide a practical screening rule.

### 10.1 Condition 1: The Integer Decision Is Mainly Activation or Selection

Good semantics include:

```text
active / inactive
use / do not use
open / close
enable / disable
select group / discard group
```

The method is not naturally designed for an integer variable representing a numerical count such as:

```text
buy 7 vehicles
produce 14 units
schedule 3 identical machines
```

### 10.2 Condition 2: Each Activation Has a Meaningful Continuous Group

There should exist

$$
x_g
$$

such that

$$
x_g=0\Longleftrightarrow\text{group }g\text{ is inactive}.
$$

This is the most important condition.

Without it, there is no valid support representation of the binary decision.

### 10.3 Condition 3: The Binary Variable Does Not Carry Essential Extra Logic

The method is most natural when $y_g$ is essentially an indicator.

It becomes much less suitable when $y_g$ also appears in constraints expressing:

- mutual exclusion;
- precedence;
- minimum up/down time;
- sequencing;
- exact cardinality;
- logical implications.

These structures are genuinely discrete.

### 10.4 Condition 4: The Remaining Continuous Core Is Convex

After eliminating the activation indicators, the remaining objective and feasible set should be convex.

Typical favorable ingredients include:

- linear flow conservation;
- linear capacity constraints;
- convex quadratic costs;
- norm constraints;
- second-order cone constraints.

### 10.5 Condition 5: Approximate Structural Selection Is Acceptable

If the application requires a sparse, high-quality active set rather than a certificate of exact integer optimality, mixed-norm convex relaxation is especially attractive.

If exact integer optimality is mandatory, the method can still be used for screening or warm starting.

### 10.6 Condition 6: Group Scales Can Be Normalized Meaningfully

If groups have very different dimensions or units, raw norms may be misleading.

Weights $\omega_g$ should account for:

- group dimension;
- units;
- expected magnitude;
- capacity;
- activation cost;
- normalization.

---

## 11. Typical Suitable and Unsuitable Problems⭐⭐⭐

### 11.1 Strong Candidates

**Base-Station or Radio-Unit Activation**

Let $x_g$ contain all transmission variables associated with station $g$.

Then

$$
x_g=0
$$

naturally means the station contributes nothing.

Group-sparse beamforming is a well-established example [4,5].

**Sensor Selection**

Let $x_g$ contain all decision variables associated with sensor $g$.

A zero group means the sensor is unnecessary.

**Feature-Group Selection**

Group LASSO was originally developed for selecting grouped explanatory variables [3].

**Server or Computing-Resource Activation**

Let $x_g$ contain all workloads assigned to server $g$.

If zero workload genuinely means the server can be inactive, group sparsity is a natural surrogate.

**Flow-Based Node or Link Activation**

Let $x_g$ contain all flows using one candidate network resource.

If zero group flow is equivalent to resource inactivity, the structure is highly suitable [2].

### 11.2 Weak Candidates

**Knapsack**

The binary variable means whether an indivisible item is selected.

There is usually no natural continuous group whose zero support exactly represents the full combinatorial structure.

**Assignment**

The discrete structure is a one-to-one correspondence.

Its special tractability comes from assignment-polytope integrality and the Hungarian algorithm, not group sparsity.

**Vehicle Routing**

Routing contains:

- visit decisions;
- sequencing;
- route continuity;
- subtour elimination.

Group support cannot replace these logical requirements.

**Unit Commitment**

Generator commitment involves:

- startup and shutdown decisions;
- minimum up/down time;
- ramping;
- minimum generation levels.

Power equal to zero captures only part of the discrete logic.

**Scheduling with Precedence**

A schedule may require ordering, machine assignment, and non-overlap.

Those decisions cannot generally be represented by whether one continuous group is zero.

---

## 12. Regularization Strength, Weights, and Practical Use⭐⭐⭐

### 12.1 Effect of $\lambda$

Consider

$$
\min_{x\in\mathcal{C}}\quad f(x)+\lambda\sum_g\omega_g\|x_g\|_2.
$$

If

$$
\lambda=0,
$$

there is no explicit sparsity pressure.

As $\lambda$ increases, more emphasis is placed on eliminating or shrinking groups.

If the all-zero solution is feasible and $\lambda$ becomes sufficiently large, the trivial solution may become optimal.

So $\lambda$ is not merely a solver parameter.

It changes the optimization model.

### 12.2 Pareto-Frontier Tuning

A practical strategy is:

1. choose a range of $\lambda$ values;
2. solve the convex model for each value;
3. record the original performance metric;
4. record the number of active groups;
5. examine the performance-versus-sparsity frontier;
6. choose the desired operating point.

This is often more informative than selecting one $\lambda$ arbitrarily.

### 12.3 Group Weights

Use

$$
\sum_{g=1}^{G}\omega_g\|x_g\|_2
$$

when groups have different sizes, costs, or scales.

One statistical normalization is

$$
\omega_g=\sqrt{d_g}.
$$

In engineering models, better weights may come from:

- fixed activation costs;
- capacities;
- normalized resource usage;
- physical units.

There is no universally optimal weighting rule.

### 12.4 Shrinkage Bias

The mixed norm penalizes the magnitude of active groups.

This can distort the continuous solution even after the correct support has been identified.

A useful remedy is **support selection followed by re-optimization**.

---

## 13. A Recommended Workflow for New Problems⭐⭐⭐

Suppose a new MILP contains many activation binaries.

A systematic workflow is:

### Step 1: Identify Activation Variables

Find binary variables whose main meaning is

```text
resource used / resource unused
```

Do not include binaries that encode ordering, precedence, or complex logic.

### Step 2: Construct Continuous Groups

For every activation variable $y_g$, identify a vector $x_g$ containing all continuous activity associated with that resource.

### Step 3: Verify the Support Equivalence

Check whether

$$
x_g=0\Longleftrightarrow y_g=0
$$

is physically and mathematically valid.

If not, stop. The mixed-norm route is not justified.

### Step 4: Eliminate the Activation Indicators Conceptually

Rewrite the fixed activation structure using

$$
\mathbf{1}\{\|x_g\|_2>0\}.
$$

This exposes the group-cardinality problem.

### Step 5: Check the Continuous Core

Determine whether the remaining feasible set and objective are convex.

If they are not, mixed-norm regularization alone will not create a convex problem.

### Step 6: Replace Group Cardinality by a Mixed Norm

Use

$$
\sum_g\omega_g\|x_g\|_2
$$

as the convex group-sparsity surrogate.

### Step 7: Solve for Several Regularization Strengths

Do not trust one arbitrary value of $\lambda$.

Explore the performance-sparsity trade-off.

### Step 8: Extract the Support

Define an active set using a numerical tolerance:

$$
\mathcal{S}=\{g:\|x_g\|_2>\varepsilon\}.
$$

The tolerance is necessary because numerical solvers operate in finite precision.

### Step 9: Re-Optimize Without the Sparsity Penalty

Fix

$$
x_g=0,\quad g\notin\mathcal{S},
$$

and solve the original continuous objective over the selected support.

This reduces shrinkage bias.

### Step 10: If Exact Integer Optimality Is Required, Return to MILP

Use the mixed-norm solution to reduce the candidate set and then solve the reduced exact MILP.

This gives the hybrid workflow

```text
structured MILP
→ mixed-norm convex surrogate
→ support screening
→ reduced MILP
→ exact solver
```

The mixed-norm stage is then used as an intelligent structural filter rather than as a replacement for exact optimization.

---

## 14. Conic Form and Solver Choices

### 14.1 SOCP Representation

Introduce an auxiliary variable $t_g$ and impose

$$
\|x_g\|_2\le t_g.
$$

Then

$$
\min_{x,t}\quad f(x)+\lambda\sum_{g=1}^{G}\omega_gt_g
$$

subject to

$$
x\in\mathcal{C}
$$

and

$$
\|x_g\|_2\le t_g,\quad g=1,\ldots,G.
$$

Each norm constraint is a second-order cone constraint.

Therefore, if the rest of the model is linear or conic-representable, the mixed-norm problem can often be formulated as an SOCP [7].

### 14.2 Proximal Methods

When the model has the form

$$
\min_x\quad f(x)+\lambda\sum_g\omega_g\|x_g\|_2
$$

with smooth $f$, proximal methods are natural because the nonsmooth mixed-norm term has a closed-form group-thresholding operator [6].

This avoids treating nonsmoothness as an obstacle.

### 14.3 Large-Scale Structured Methods

If variables and constraints are separable into blocks, methods such as:

- primal-dual splitting;
- ADMM variants;
- block coordinate methods;
- BSUM-type algorithms;

may exploit the problem structure [8].

The appropriate solver depends on the exact coupling constraints and convergence assumptions.

---

## 15. Relationship to Exact ILP Methods⭐⭐⭐

Mixed-norm convex relaxation and exact decomposition methods attack computational difficulty in fundamentally different ways.

| Property | Mixed-Norm Convex Relaxation | Dantzig-Wolfe / Column Generation |
|:---:|:---|:---|
| Core idea | replace combinatorial support by a convex surrogate | reformulate a large LP relaxation using generated columns |
| Binary structure | often approximated or removed | preserved in the original MILP |
| Objective | generally modified | reformulated LP objective remains equivalent |
| Convex subproblem | yes, when assumptions hold | restricted master problem is an LP |
| Exact MILP optimum | not generally guaranteed | possible through Branch-and-Price |
| Main advantage | speed and scalable structural selection | exactness with decomposition |
| Best fit | activation-dominated models | large structured models with decomposable columns |

Relevant earlier notes are:

- [Column Generation and Its Applications in Integer Linear Programming](05-column-generation-and-ilp-applications.md)
- [Branch and Price](06-branch-and-price.md)
- [Dantzig-Wolfe Decomposition](08-dantzig-wolfe-decomposition.md)

These approaches can also complement each other.

A mixed-norm model can screen thousands of candidate resources before an exact Branch-and-Price or MILP model is solved on the reduced set.

---

## 16. Decision Checklist⭐⭐⭐

Before using mixed-norm convex relaxation, ask:

| Question | Favorable Answer |
|:---:|:---|
| Is the integer variable mainly an activation or selection indicator? | Yes |
| Is there a meaningful continuous vector associated with that activation? | Yes |
| Does zero activity mean the resource is genuinely inactive? | Yes |
| Does the binary variable avoid essential extra Boolean logic? | Yes |
| Is the remaining continuous core convex? | Yes |
| Is a high-quality sparse solution useful even without exact integer certification? | Yes |
| Can the groups be normalized and weighted meaningfully? | Yes |

The strongest candidate has the form

$$
\text{resource }g\text{ active}\Longleftrightarrow\|x_g\|_2>0
$$

with an otherwise convex optimization model.

If several answers are “No,” forcing the problem into a mixed-norm formulation is likely to destroy important structure rather than simplify it.

---

## 17. Key Takeaways⭐⭐⭐

1. Some structured ILP and MILP problems can be transformed into **convex mixed-norm surrogate problems**.
2. The method is especially suitable when binary variables represent **group activation or group selection**.
3. The central modeling requirement is

$$
x_g=0\Longleftrightarrow\text{group }g\text{ is inactive}.
$$

4. The exact discrete structure can be expressed through the nonconvex group-cardinality quantity

$$
\|x\|_{2,0}.
$$

5. Replacing $\ell_{2,0}$ by the convex mixed $\ell_{2,1}$ norm gives the key convex relaxation:

$$
\sum_g\mathbf{1}\{\|x_g\|_2>0\}\quad\longrightarrow\quad\sum_g\omega_g\|x_g\|_2.
$$

6. The inner $\ell_2$ norm treats one group as a unit, while the outer $\ell_1$ structure promotes sparsity across groups.
7. Exact zero groups arise from the nonsmooth geometry of the norm and can be understood rigorously through subgradients and group soft thresholding.
8. The resulting model is convex only if the remaining continuous objective and feasible set are convex.
9. A global optimum of the convex surrogate is not automatically a global optimum of the original ILP or MILP.
10. The method cannot exactly convexify arbitrary ILPs unless additional special structure exists.
11. It is poorly suited to problems dominated by sequencing, routing logic, permutation structure, exact cardinality, or arbitrary Boolean constraints.
12. Its main advantages are scalability, elimination of explicit activation binaries, natural group-level sparsity, and access to mature convex optimization algorithms.
13. Regularization strength and group scaling strongly influence the selected support.
14. A practical workflow is **mixed-norm optimization → support extraction → unpenalized re-optimization**.
15. If exact integer optimality is required, use **mixed-norm screening → reduced exact MILP**.
16. Mixed-norm convex relaxation and Dantzig-Wolfe / column generation solve scale in different ways: the former changes the combinatorial selection model into a convex surrogate, while the latter preserves the optimization structure and decomposes it.

---

## References

1. Sun, Xiaoling, and Duan Li. *Integer Programming*. Beijing: Science Press, 2010. ISBN: 978-7-03-029380-0.（孙小玲、李端：《整数规划》，北京：科学出版社，2010年，ISBN：978-7-03-029380-0）
2. Yang, Huiting, Wei Liu, Xiangfeng Wang, and Jiandong Li. “Group Sparse Space Information Network With Joint Virtual Network Function Deployment and Maximum Flow Routing Strategy.” *IEEE Transactions on Wireless Communications* 22(8), 2023, 5291–5305. DOI: 10.1109/TWC.2022.3233067.
3. Yuan, Ming, and Yi Lin. “Model Selection and Estimation in Regression with Grouped Variables.” *Journal of the Royal Statistical Society: Series B (Statistical Methodology)* 68(1), 2006, 49–67. DOI: 10.1111/j.1467-9868.2005.00532.x.
4. Hong, Mingyi, Ruoyu Sun, Hadi Baligh, and Zhi-Quan Luo. “Joint Base Station Clustering and Beamformer Design for Partial Coordinated Transmission in Heterogeneous Networks.” *IEEE Journal on Selected Areas in Communications* 31(2), 2013, 226–240. DOI: 10.1109/JSAC.2013.130211.
5. Shi, Yuanming, Jun Zhang, and Khaled B. Letaief. “Group Sparse Beamforming for Green Cloud-RAN.” *IEEE Transactions on Wireless Communications* 13(5), 2014, 2809–2823. DOI: 10.1109/TWC.2014.040214.131770.
6. Bach, Francis, Rodolphe Jenatton, Julien Mairal, and Guillaume Obozinski. “Optimization with Sparsity-Inducing Penalties.” *Foundations and Trends in Machine Learning* 4(1), 2012, 1–106. DOI: 10.1561/2200000015.
7. Boyd, Stephen, and Lieven Vandenberghe. *Convex Optimization*. Cambridge: Cambridge University Press, 2004. ISBN: 978-0-521-83378-3.
8. Hong, Mingyi, Tsung-Hui Chang, Xiangfeng Wang, Meisam Razaviyayn, Shiqian Ma, and Zhi-Quan Luo. “A Block Successive Upper-Bound Minimization Method of Multipliers for Linearly Constrained Convex Optimization.” *Mathematics of Operations Research* 45(3), 2020, 833–861. DOI: 10.1287/moor.2019.1010.

---

## Suggested Follow-up Reading

The most relevant next topics are:

1. **Sparse Optimization and Proximal Algorithms**  
   Develops the subgradient and proximal-operator viewpoint behind $\ell_1$, Group LASSO, group soft thresholding, and scalable first-order optimization.

2. **Exact Convexifications of On/Off Constraints**  
   Studies perspective formulations, convex hull descriptions, and other techniques that strengthen or exactly characterize special mixed-integer structures without replacing them by heuristic sparsity penalties.

3. **Reweighted Sparse Optimization**  
   Uses iterative weighting to approximate group cardinality more aggressively than a single mixed-norm penalty.

4. **Hybrid Convex Screening and Exact MILP**  
   Uses a convex group-sparse model to reduce candidate variables before applying branch-and-bound, branch-and-cut, or Branch-and-Price to a smaller exact model.
