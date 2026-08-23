# Integer Linear Programming Notes 09: Benders Decomposition

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

This note introduces **Benders Decomposition**, one of the most important decomposition methods for large-scale mathematical programming.

The starting point is the idea of **complicating variables**.

A variable is called complicating if, after fixing its value, the remaining optimization problem becomes much easier to solve.

For example, in many MILPs:

```text
integer variables
        ↓ fixed
remaining problem
        ↓
linear program
```

This is exactly the structure that classical Benders Decomposition tries to exploit.

Instead of solving all variables together, Benders separates the original model into:

- a **Master Problem**, which contains the complicating variables;
- a **Subproblem**, which contains the remaining variables.

The Master Problem proposes a value for the complicating variables. The Subproblem then checks what happens under that decision.

If the decision makes the Subproblem infeasible, a **Benders feasibility cut** is returned to the Master Problem.

If the Subproblem is feasible but the Master Problem underestimates its true cost, a **Benders optimality cut** is returned.

Then the Master Problem is solved again.

So, at a high level:

```text
solve Master Problem
        ↓
fix complicating variables
        ↓
solve Subproblem
        ↓
use dual information
        ↓
generate a Benders cut
        ↓
add the cut to the Master Problem
        ↓
repeat
```

The important point is that **the cuts are not chosen arbitrarily**.

They come naturally from the dual of the parameterized Subproblem.

Benders originally developed this idea for mixed-variable programming problems [1]. Geoffrion later generalized the same principle to a broader class of nonlinear optimization problems [2].

---

## 2. Background and Main Idea⭐

Benders Decomposition was introduced by Jacques F. Benders in 1962 [1].

The original motivation was mixed-variable optimization: some variables are difficult, while fixing them leaves a much more tractable problem.

The most common modern example is a MILP in which:

- the Master Problem contains integer or strategic variables;
- fixing those variables leaves a continuous LP Subproblem.

This is not the only possible use of Benders Decomposition. Generalized Benders Decomposition can handle broader convex and nonlinear structures, and later variants also include combinatorial and logic-based forms [2,3].

For the classical linear case, the core logic is:

> **Project the easy variables out of the model, keep the complicating variables in the Master Problem, and use duality to describe the effect of the eliminated variables through cuts.**

This viewpoint is useful because the complete Benders reformulation may contain a very large number of constraints.

We do not generate all of them at once.

Instead, we generate only the cuts that are needed.

This is why Benders Decomposition can be viewed as a **cutting-plane method**.

---

## 3. Classical Benders Decomposition⭐⭐⭐

### 3.1 Original Problem

Consider

$$
\min \quad c^T x+f^T y
$$

subject to

$$
Ax+By=b
$$

$$
x\ge0
$$

$$
y\in Y.
$$

Here:

- $x\in\mathbb{R}^p$ contains the variables that will remain in the Subproblem;
- $y\in\mathbb{R}^q$ contains the complicating variables;
- $Y$ contains all restrictions involving only $y$;
- $A$ and $B$ are matrices of appropriate dimensions;
- $b$, $c$, and $f$ are vectors of appropriate dimensions.

In a typical MILP application, $y$ contains some or all of the integer variables.

The important assumption is not simply that $y$ is integer.

The important assumption is:

> **Once $y$ is fixed, optimization over $x$ becomes substantially easier.**

### 3.2 Fix the Complicating Variables

Suppose $y$ is fixed.

Then the original problem becomes

$$
q(y)=\min_{x\ge0}\{c^T x:Ax=b-By\}.
$$

The function $q(y)$ is the optimal value of the Subproblem for the given $y$.

The original problem can therefore be written as

$$
\min_{y\in Y}\quad f^T y+q(y).
$$

This expression already shows the basic decomposition.

The Master Problem chooses $y$.

The Subproblem tells the Master Problem the true consequence of that choice through $q(y)$.

### 3.3 Dual of the Subproblem⭐

For fixed $y$, the Subproblem is an LP:

$$
q(y)=\min \quad c^T x
$$

subject to

$$
Ax=b-By
$$

$$
x\ge0.
$$

Associate the dual vector $\alpha$ with the equality constraints.

Its dual is

$$
\max_{\alpha}\quad \alpha^T(b-By)
$$

subject to

$$
A^T\alpha\le c
$$

$$
\alpha\in\mathbb{R}^m.
$$

The key observation is extremely important:

> **The feasible region of the dual Subproblem does not depend on $y$. Only the dual objective depends on $y$.**

Define

$$
U=\{\alpha\in\mathbb{R}^m:A^T\alpha\le c\}.
$$

Then

$$
q(y)=\max_{\alpha\in U}\alpha^T(b-By)
$$

whenever the primal and dual Subproblems are feasible and have finite optimal values.

For the geometric derivation below, assume that $U$ is nonempty and pointed. If $U$ contains a lineality space, the same idea requires the corresponding lineality directions to be handled explicitly.

---

## 4. From Dual Geometry to Benders Cuts⭐⭐⭐

The entire classical Benders construction comes from the geometry of the dual feasible region $U$.

Let its extreme points be

$$
\alpha_p^1,\ldots,\alpha_p^I,
$$

and let its extreme rays be

$$
\alpha_r^1,\ldots,\alpha_r^J.
$$

The role of these two objects is different:

```text
extreme rays
→ determine whether the Subproblem is feasible

extreme points
→ determine the optimal value of a feasible Subproblem
```

This is the origin of the two families of Benders cuts [1,4].

### 4.1 Feasibility Cuts

Consider a trial value $y^k$ from the Master Problem.

Suppose there exists an extreme ray $\alpha_r^j$ such that

$$
(\alpha_r^j)^T(b-By^k)>0.
$$

Moving in this ray direction makes the dual objective increase without bound.

Therefore, the dual Subproblem is unbounded.

Under the standing assumptions above, this means that the primal Subproblem corresponding to $y^k$ is infeasible.

To prevent the Master Problem from choosing the same type of infeasible decision again, we require

$$
(\alpha_r^j)^T(b-By)\le0.
$$

This is a **Benders feasibility cut**.

More generally, every feasible Master decision must satisfy

$$
(\alpha_r^j)^T(b-By)\le0,\quad j=1,\ldots,J.
$$

The interpretation is simple:

> **A feasibility cut removes Master solutions for which the Subproblem cannot be made feasible.**

### 4.2 Optimality Cuts

Now suppose the Subproblem for $y^k$ is feasible and bounded.

By LP duality,

$$
q(y^k)=\max_{\alpha\in U}\alpha^T(b-By^k).
$$

For a pointed polyhedron with a finite optimum, an optimal dual solution can be chosen at an extreme point.

Suppose $\alpha_p^i$ is such an optimal extreme point.

Then

$$
q(y^k)=(\alpha_p^i)^T(b-By^k).
$$

Introduce a Master variable $\theta$ to represent the Subproblem value.

Since

$$
q(y)=\max_{\alpha\in U}\alpha^T(b-By),
$$

we must have

$$
\theta\ge(\alpha_p^i)^T(b-By)
$$

for every relevant extreme point.

This is a **Benders optimality cut**.

The interpretation is:

> **An optimality cut prevents the Master Problem from underestimating the cost of the Subproblem.**

### 4.3 Complete Benders Reformulation⭐

If every extreme ray and every extreme point were known, the original problem could be rewritten as

$$
\min \quad f^T y+\theta
$$

subject to

$$
(\alpha_r^j)^T(b-By)\le0,\quad j=1,\ldots,J
$$

$$
\theta\ge(\alpha_p^i)^T(b-By),\quad i=1,\ldots,I
$$

$$
y\in Y.
$$

This formulation contains the complicating variables $y$ and the auxiliary variable $\theta$, but the original $x$ variables have disappeared.

This is one of the cleanest ways to understand Benders Decomposition:

> **Benders projects the $x$ variables out of the original model and replaces their effect by a family of constraints in the $y$ space.**

The problem is that the numbers of extreme points and extreme rays can be very large.

Generating all Benders cuts in advance is therefore usually impractical.

That leads directly to the iterative algorithm.

---

## 5. Relaxed Master Problem and Iterative Cut Generation⭐⭐⭐

### 5.1 Relaxed Master Problem

Let

$$
P^k\subseteq\{1,\ldots,I\}
$$

be the set of optimality cuts currently generated, and let

$$
R^k\subseteq\{1,\ldots,J\}
$$

be the set of feasibility cuts currently generated.

At iteration $k$, the Relaxed Master Problem is

$$
\min \quad f^T y+\theta
$$

subject to

$$
(\alpha_r^j)^T(b-By)\le0,\quad j\in R^k
$$

$$
\theta\ge(\alpha_p^i)^T(b-By),\quad i\in P^k
$$

$$
y\in Y.
$$

Because only a subset of the complete Benders cuts is included, this Master Problem is a relaxation of the full Benders reformulation.

For a minimization problem, its optimal value is therefore a **lower bound** on the original optimal value.

> **Implementation note:** If no optimality cut is present and $\theta$ is unrestricted, the initial Relaxed Master Problem may be unbounded. In practice, one usually adds a valid lower bound on $\theta$, an initial optimality cut, or another initialization mechanism.

### 5.2 Lower Bound and Upper Bound

Let the current Master solution be

$$
(y^k,\theta^k).
$$

The Master objective gives

$$
LB_k=f^Ty^k+\theta^k.
$$

If the Subproblem corresponding to $y^k$ is feasible and bounded, compute

$$
q(y^k).
$$

Then

$$
f^Ty^k+q(y^k)
$$

is the objective value of a feasible solution of the original problem.

Therefore, it provides an upper bound:

$$
UB_k=\min\{UB_{k-1},f^Ty^k+q(y^k)\}.
$$

For a minimization problem,

$$
LB_k\le v^{\ast}\le UB_k,
$$

where $v^{\ast}$ is the optimal value of the original problem.

This bound interpretation is important in implementations.

The algorithm does not merely alternate between two models. It progressively closes the gap between a Master lower bound and a primal upper bound.

### 5.3 One Benders Iteration

After solving the Relaxed Master Problem, fix $y=y^k$ and solve the Subproblem.

There are two main cases.

#### Case 1: The Subproblem Is Infeasible

A dual extreme ray $\alpha_r$ is obtained with

$$
\alpha_r^T(b-By^k)>0.
$$

Add the feasibility cut

$$
\alpha_r^T(b-By)\le0.
$$

Then solve the Master Problem again.

#### Case 2: The Subproblem Is Feasible and Bounded

Let $\alpha_p$ be an optimal dual solution.

Then

$$
q(y^k)=\alpha_p^T(b-By^k).
$$

Update the upper bound.

If

$$
q(y^k)>\theta^k
$$

within the required numerical tolerance, the Master Problem is underestimating the Subproblem value.

Add the optimality cut

$$
\theta\ge\alpha_p^T(b-By).
$$

Then solve the Master Problem again.

If the global optimality gap is sufficiently small,

$$
UB-LB\le\varepsilon,
$$

the algorithm terminates.

### 5.4 Complete Procedure

```text
initialize the Relaxed Master Problem
initialize LB and UB

repeat:

    solve Relaxed Master Problem
        ↓
    obtain y^k and theta^k
        ↓
    update LB
        ↓
    fix y = y^k
        ↓
    solve Benders Subproblem

    if Subproblem is infeasible:
        obtain an extreme ray
        generate a feasibility cut
        add it to the Master Problem

    else:
        obtain q(y^k) and an optimal dual solution
        update UB

        if theta^k underestimates q(y^k):
            generate an optimality cut
            add it to the Master Problem

until UB - LB <= tolerance
```

### 5.5 Why Does Classical Benders Converge?

Under the standard polyhedral assumptions, the dual feasible region has finitely many extreme points and extreme rays.

Every nonterminal exact iteration generates a violated cut that was not already active in the Master Problem.

Therefore, the classical finite-cut setting admits finite convergence when the Master and Subproblem are solved exactly [1].

In practice, numerical tolerances, degeneracy, weak cuts, and difficult Master Problems can still make convergence slow.

Finite theoretical convergence does not mean fast computational convergence.

---

## 6. Why Benders Decomposition Is a Cutting-Plane Method⭐

This point is worth separating from the formulas.

At this stage, the connection with the earlier **Cutting Plane** note becomes very direct. If the ideas of valid inequalities, cutting planes, and iterative cut generation are no longer fresh, it may be useful to review:

- [Cutting Plane](03-cutting-plane.md)

The key difference here is that Benders cuts are generated from the Subproblem and are added in the Master-variable space.

Benders Decomposition is not simply:

```text
solve one problem
then solve another problem
```

The deeper idea is:

> **The Master Problem initially has an incomplete description of the true projected problem. The Subproblem repeatedly discovers missing constraints.**

### 6.1 Value-Function Viewpoint

For feasible $y$,

$$
q(y)=\max_{\alpha\in U}\alpha^T(b-By).
$$

For each fixed dual vector $\alpha$, the function

$$
\alpha^T(b-By)
$$

is affine in $y$.

Therefore, $q(y)$ is the pointwise maximum of affine functions.

Hence $q(y)$ is convex and piecewise linear in the classical linear setting.

An optimality cut

$$
\theta\ge\alpha^T(b-By)
$$

adds one affine lower approximation of this value function.

As more cuts are generated,

```text
rough approximation of q(y)
        ↓
more supporting affine functions
        ↓
better approximation of q(y)
        ↓
exact value function at convergence
```

### 6.2 Feasibility-Projection Viewpoint

The feasibility cuts have a different role.

The set of $y$ values for which there exists some $x\ge0$ satisfying

$$
Ax+By=b
$$

is the projection of the original feasible region onto the $y$ space.

The Master Problem does not initially know this projected feasible set exactly.

Feasibility cuts progressively describe it.

So Benders performs two tasks simultaneously:

1. **feasibility cuts** approximate the projected feasible region;
2. **optimality cuts** approximate the projected objective function.

This is why the original Benders idea is naturally a cutting-plane method.

---

## 7. A Fixed-Charge Transportation Example⭐⭐

A useful example is the fixed-charge transportation problem.

Suppose:

- $i\in I$ indexes supply nodes;
- $j\in J$ indexes demand nodes;
- $s_i$ is the supply available at source $i$;
- $d_j$ is the demand at destination $j$;
- $c_{ij}$ is the unit transportation cost;
- $F_{ij}$ is the fixed cost of activating arc $(i,j)$;
- $x_{ij}$ is the transported amount;
- $y_{ij}=1$ if arc $(i,j)$ is activated.

A natural upper bound is

$$
M_{ij}=\min\{s_i,d_j\}.
$$

The MILP is

$$
\min \quad \sum_{i\in I}\sum_{j\in J}c_{ij}x_{ij}+\sum_{i\in I}\sum_{j\in J}F_{ij}y_{ij}
$$

subject to

$$
\sum_{j\in J}x_{ij}\le s_i,\quad i\in I
$$

$$
\sum_{i\in I}x_{ij}\ge d_j,\quad j\in J
$$

$$
x_{ij}\le M_{ij}y_{ij},\quad i\in I,\ j\in J
$$

$$
x_{ij}\ge0,\quad i\in I,\ j\in J
$$

$$
y_{ij}\in\{0,1\},\quad i\in I,\ j\in J.
$$

The binary variables $y_{ij}$ are natural complicating variables.

Once they are fixed, the remaining model in $x_{ij}$ is an LP transportation problem.

### 7.1 Transportation Subproblem

For a fixed activation decision $\bar y$, solve

$$
q(\bar y)=\min \quad \sum_{i\in I}\sum_{j\in J}c_{ij}x_{ij}
$$

subject to

$$
\sum_{j\in J}x_{ij}\le s_i,\quad i\in I
$$

$$
\sum_{i\in I}x_{ij}\ge d_j,\quad j\in J
$$

$$
x_{ij}\le M_{ij}\bar y_{ij},\quad i\in I,\ j\in J
$$

$$
x_{ij}\ge0.
$$

Let:

- $u_i\le0$ be the dual variable of the supply constraint;
- $v_j\ge0$ be the dual variable of the demand constraint;
- $w_{ij}\le0$ be the dual variable of the activated-capacity constraint.

The dual Subproblem is

$$
\max \quad \sum_{i\in I}s_i u_i+\sum_{j\in J}d_j v_j+\sum_{i\in I}\sum_{j\in J}M_{ij}\bar y_{ij}w_{ij}
$$

subject to

$$
u_i+v_j+w_{ij}\le c_{ij},\quad i\in I,\ j\in J
$$

$$
u_i\le0,\quad v_j\ge0,\quad w_{ij}\le0.
$$

The dual feasible region does not depend on $\bar y$.

Only its objective does.

Therefore, an optimal dual extreme point produces an optimality cut of the form

$$
\theta\ge\sum_{i\in I}s_i u_i^k+\sum_{j\in J}d_j v_j^k+\sum_{i\in I}\sum_{j\in J}M_{ij}y_{ij}w_{ij}^k.
$$

If the transportation Subproblem is infeasible, an appropriate dual extreme ray produces a feasibility cut with the same affine expression constrained to be nonpositive.

The Master Problem then contains the binary activation variables $y_{ij}$ and the accumulated Benders cuts.

This is a typical MILP structure for Benders:

```text
binary design / activation decisions
        ↓ Master Problem

continuous flow decisions
        ↓ Subproblem
```

A classic large-scale distribution-system application of Benders Decomposition was presented by Geoffrion and Graves [5].

---

## 8. Application in Space-Air-Ground Integrated Networks

The original Chinese note also used a communication-network example.

Jia et al. studied data collection and transmission in a 6G space-air-ground integrated network with cooperative HAP and LEO satellite schemes [6].

The problem is formulated as a MILP.

The important Benders idea in that work is the same as the classical structure discussed above:

> **Treat the integer decision variables as complicating variables. Once they are fixed, the remaining continuous problem becomes a tractable LP-type Subproblem.**

The algorithm then iterates between a Master Problem and a Subproblem.

The paper also develops an accelerated variant for larger-scale systems.

This is a useful example because it shows that Benders Decomposition is not limited to classical transportation or facility-location models.

The same idea appears naturally in communication-network optimization whenever:

- some variables represent discrete architecture, association, activation, placement, or scheduling decisions;
- fixing them leaves a continuous resource-allocation or flow problem.

The modeling question is always the same:

> **Which variables should be fixed so that the residual problem becomes easy enough to solve repeatedly?**

---

## 9. Benders Decomposition vs. Dantzig-Wolfe Decomposition⭐

Benders Decomposition and Dantzig-Wolfe Decomposition are closely related, but their computational directions are almost opposite.

### Dantzig-Wolfe Decomposition

Dantzig-Wolfe reformulates a structured problem using complete block configurations.

The resulting Master Problem may have an enormous number of variables.

Column Generation then adds missing variables.

```text
Dantzig-Wolfe
→ huge variable space
→ generate columns
```

### Benders Decomposition

Benders eliminates a group of variables by projection.

The resulting Master Problem may have an enormous number of constraints.

The Subproblem then generates missing constraints.

```text
Benders
→ huge constraint space
→ generate cuts
```

A useful memory aid is:

```text
Column Generation / Dantzig-Wolfe
        → add columns

Benders Decomposition
        → add rows / cuts
```

Both methods avoid constructing a huge reformulation explicitly.

They reveal different sides of large-scale optimization structure.

---

## 10. Practical Issues and Important Extensions⭐

Classical Benders is elegant, but a direct implementation can converge slowly.

The main reason is often not the correctness of the cuts, but their **strength**.

### 10.1 Weak Cuts and Degeneracy

A Subproblem may have multiple optimal dual solutions.

Different dual optima can generate different valid Benders cuts.

Some cuts eliminate much more of the current Master relaxation than others.

Magnanti and Wong proposed a classical method for generating Pareto-optimal Benders cuts to accelerate convergence [7].

### 10.2 Multiple Cuts

If the Subproblem decomposes into several independent blocks, one can generate several cuts in one Master iteration instead of aggregating everything into a single cut.

This is often called a **multicut** strategy.

It may strengthen the Master faster, although it also increases Master size.

### 10.3 Generalized Benders Decomposition

The classical derivation above depends on LP duality.

Geoffrion generalized the idea to broader nonlinear problems in which fixing the complicating variables leaves a tractable optimization problem with suitable duality properties [2].

The core pattern remains:

```text
fix complicating variables
        ↓
solve easier Subproblem
        ↓
extract dual / multiplier information
        ↓
generate a cut in the Master space
```

### 10.4 Logic-Based and Combinatorial Variants

For some problems, the Subproblem is not naturally an LP.

Modern Benders-type methods can derive cuts from combinatorial reasoning or inference rather than standard LP dual multipliers.

These extensions are especially useful in scheduling, routing, constraint programming, and other highly discrete applications [3].

### 10.5 Strong Formulation Still Matters

Benders Decomposition is not automatically fast simply because the original model is decomposable.

Performance depends on:

- which variables are chosen as complicating variables;
- how difficult the Master Problem is;
- how difficult the Subproblem is;
- how strong the generated cuts are;
- how the Master is initialized;
- numerical tolerances;
- stabilization and cut-management strategies.

The decomposition should expose useful structure, not merely split the model into two arbitrary parts.

---

## 11. Key Takeaways

1. Benders Decomposition is designed for problems containing **complicating variables** whose fixation makes the remaining problem substantially easier.
2. In the classical MILP setting, the complicating variables are often integer variables and the Subproblem is a continuous LP.
3. Fixing the Master variables $y$ defines the value function $q(y)$ of the Subproblem.
4. LP duality converts the Subproblem into a dual problem whose feasible region is independent of $y$.
5. Extreme rays of the dual feasible region generate **Benders feasibility cuts**.
6. Extreme points of the dual feasible region generate **Benders optimality cuts**.
7. Feasibility cuts describe the projection of the original feasible region onto the Master-variable space.
8. Optimality cuts progressively approximate the Subproblem value function.
9. The Relaxed Master Problem gives a lower bound for a minimization problem, while a feasible Master decision together with its solved Subproblem gives an upper bound.
10. Benders Decomposition is fundamentally a **cutting-plane method**.
11. Dantzig-Wolfe / Column Generation generates missing variables, while Benders Decomposition generates missing constraints.
12. Exact finite convergence does not guarantee fast convergence; strong cuts and a good variable partition are often critical in practice.
13. Generalized Benders Decomposition extends the same idea beyond linear Subproblems.

## References

1. Benders, J. F. “Partitioning Procedures for Solving Mixed-Variables Programming Problems.” *Numerische Mathematik*, 4, 1962, pp. 238–252. DOI: `10.1007/BF01386316`.

2. Geoffrion, A. M. “Generalized Benders Decomposition.” *Journal of Optimization Theory and Applications*, 10(4), 1972, pp. 237–260. DOI: `10.1007/BF00934810`.

3. Rahmaniani, R., Crainic, T. G., Gendreau, M., and Rei, W. “The Benders Decomposition Algorithm: A Literature Review.” *European Journal of Operational Research*, 259(3), 2017, pp. 801–817. DOI: `10.1016/j.ejor.2016.12.005`.

4. Bertsimas, D., and Tsitsiklis, J. N. *Introduction to Linear Optimization*. Athena Scientific, 1997.

5. Geoffrion, A. M., and Graves, G. W. “Multicommodity Distribution System Design by Benders Decomposition.” *Management Science*, 20(5), 1974, pp. 822–844. DOI: `10.1287/mnsc.20.5.822`.

6. Jia, Z., Sheng, M., Li, J., and Han, Z. “Toward Data Collection and Transmission in 6G Space-Air-Ground Integrated Networks: Cooperative HAP and LEO Satellite Schemes.” *IEEE Internet of Things Journal*, 9(13), 2022, pp. 10516–10528. DOI: `10.1109/JIOT.2021.3121760`.

7. Magnanti, T. L., and Wong, R. T. “Accelerating Benders Decomposition: Algorithmic Enhancement and Model Selection Criteria.” *Operations Research*, 29(3), 1981, pp. 464–484. DOI: `10.1287/opre.29.3.464`.

## Suggested Follow-up Reading

The most relevant next topics are:

1. **Computational Complexity Theory**  
   Helps explain why decomposition is necessary for many large-scale integer programs and why worst-case complexity remains difficult even when a useful decomposition exists.

2. **Advanced Benders Cuts**  
   Pareto-optimal cuts, multicut formulations, cut selection, cut strengthening, and stabilization.

3. **Generalized Benders Decomposition**  
   Extends the classical LP-based Subproblem framework to broader nonlinear and convex structures.

4. **Logic-Based Benders Decomposition**  
   Replaces LP duality with problem-specific inference and is useful when the Subproblem is discrete or combinatorial.

5. **Stochastic Programming and the L-Shaped Method**  
   The L-shaped method can be viewed as a specialized Benders framework for two-stage stochastic linear programming.
