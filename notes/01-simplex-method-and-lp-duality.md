# Integer Linear Programming Notes 01: Simplex Method and Linear Programming Duality

> Author: Hang Li (@flgkd)

## 1. Preface

Integer Linear Programming (ILP) can be viewed as a special form of Linear Programming (LP) with additional integrality restrictions on some or all decision variables.

In many ILP algorithms, the integer restrictions are first relaxed, and the corresponding LP relaxation is solved. Therefore, before studying ILP algorithms in depth, it is necessary to understand the basic theory and solution methods of linear programming, especially the simplex method.

This is also important because some later algorithms, such as **column generation**, are closely related to the simplex method.

The simplex method is a general-purpose algorithm for solving linear programming problems. However, it is **not a polynomial-time algorithm in the worst case**. Polynomial-time algorithms for LP include the **ellipsoid method** and **interior-point methods**. Even so, the simplex method is still widely used in practice because of its intuitive principle, clear geometric interpretation, and strong practical performance.

## 2. Simplex Method

The simplex method contains many technical details. This note does not attempt to give a full textbook-style derivation of the simplex algorithm. Instead, it records several key concepts in linear programming and the simplex method that are especially important for later study of integer programming and decomposition-based optimization methods.

---

## 3. Key Concepts

## 3.1 Linear Programming

### 3.1.1 Possible Solution Outcomes

When solving a linear programming problem, the result may fall into one of the following cases:

1. Unique optimal solution
2. Multiple optimal solutions
3. Infinitely many optimal solutions
4. Unbounded solution
5. No feasible solution

These cases are important because simplex-based algorithms do not merely compute a numerical solution. They also provide criteria for identifying optimality, infeasibility, unboundedness, and alternative optima.

### 3.1.2 Standard Form

A linear programming problem is often converted into standard form before applying the simplex method.

A typical standard form is:

```math
\max \quad c^\top x
```

```math
\text{s.t.} \quad Ax = b,
```

```math
x \ge 0.
```

For inequality constraints, slack variables, surplus variables, or artificial variables may be introduced depending on the form of the original constraints.

### 3.1.3 Concepts of Solutions

Several basic concepts are central to the simplex method:

1. **Feasible solution**  
   A solution satisfying all constraints, including non-negativity constraints.

2. **Basis**  
   A set of linearly independent columns selected from the constraint matrix.

3. **Basic solution**  
   A solution obtained by setting non-basic variables to zero and solving for the basic variables.

4. **Basic feasible solution**  
   A basic solution that also satisfies the non-negativity constraints.

5. **Feasible basis**  
   A basis corresponding to a basic feasible solution.

The relationship among general solutions, feasible solutions, basic solutions, and basic feasible solutions is illustrated in the original note as follows:

![Relationship among infeasible solutions, feasible solutions, basic solutions, and basic feasible solutions](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651298994732-3fdadf99-be03-4c1d-8639-a52f7287ae37.png)

### 3.1.4 Geometric Interpretation

Linear programming has an important geometric interpretation:

1. The feasible region of an LP is a **convex set**.
2. A basic feasible solution corresponds to a **vertex** or **extreme point** of the feasible region.

The simplex method can therefore be understood as an algorithm that moves from one vertex of the feasible region to another adjacent vertex, improving the objective value until no further improvement is possible.

---

## 3.2 Simplex Method

### 3.2.1 Meaning of “Simplex”

A simplex is a generalization of geometric objects such as points, line segments, triangles, and tetrahedra:

- In 0-dimensional space, a simplex is a point.
- In 1-dimensional space, a simplex is a line segment.
- In 2-dimensional space, a simplex is a triangle.
- In 3-dimensional space, a simplex is a tetrahedron.
- In \(n\)-dimensional space, a simplex is a polytope with \(n+1\) vertices.

### 3.2.2 Determining an Initial Basic Feasible Solution

The simplex method starts from an initial basic feasible solution.

For some LPs, an initial basic feasible solution can be obtained directly from slack variables. However, when such an initial feasible basis is not obvious, artificial variables may need to be introduced. This leads to methods such as:

1. The Big-M method
2. The two-phase method

These methods are especially important when the original constraints do not naturally provide an identity matrix as the initial basis.

### 3.2.3 Optimality Test and Solution Classification

One of the most important parts of the simplex method is the **optimality test**.
The key idea is to rewrite the objective function in terms of the non-basic variables. Then the coefficient of each non-basic variable indicates whether introducing that variable into the basis can improve the objective value.

After several simplex iterations, the current system can be written as

$$
x_i = b_i^{\prime} - \sum_{j=m+1}^{n} a_{ij}^{\prime}x_j,
\qquad i = 1,2,\ldots,m.
\qquad \text{(2-24)}
$$

Here, $x_1,\ldots,x_m$ are the current basic variables, while $x_{m+1},\ldots,x_n$ are the current non-basic variables.

Substituting Eq. (2-24) into the objective function gives

$$
z =
\sum_{i=1}^{m} c_i b_i^{\prime}
+
\sum_{j=m+1}^{n}
\left(
c_j - \sum_{i=1}^{m} c_i a_{ij}^{\prime}
\right)x_j.
\qquad \text{(2-25)}
$$

Define

$$
z_0 = \sum_{i=1}^{m} c_i b_i^{\prime},
\qquad
z_j = \sum_{i=1}^{m} c_i a_{ij}^{\prime},
\qquad j=m+1,\ldots,n.
$$

Then Eq. (2-25) can be rewritten as

$$
z =
z_0
+
\sum_{j=m+1}^{n}
(c_j - z_j)x_j.
\qquad \text{(2-26)}
$$

Let

$$
\sigma_j = c_j - z_j,
\qquad j=m+1,\ldots,n.
$$

Then the objective function becomes

$$
z =
z_0
+
\sum_{j=m+1}^{n}
\sigma_j x_j.
\qquad \text{(2-27)}
$$

The quantity $\sigma_j = c_j - z_j$ is called the **reduced cost** or **optimality indicator** of the non-basic variable $x_j$.

For a maximization problem, the interpretation is as follows:

* If $\sigma_j > 0$, increasing $x_j$ may increase the objective value, so $x_j$ can be selected as an entering variable.
* If $\sigma_j < 0$, increasing $x_j$ would decrease the objective value.
* If $\sigma_j = 0$, increasing $x_j$ does not change the objective value, which may indicate the existence of alternative optimal solutions.

#### Optimality Criterion

For a maximization problem, if the current solution is a basic feasible solution and

$$
\sigma_j \le 0,
\qquad j=m+1,\ldots,n,
$$

then no non-basic variable can enter the basis to further improve the objective value. Therefore, the current basic feasible solution is optimal.

#### Criterion for Infinitely Many Optimal Solutions

If the current solution is a basic feasible solution,

$$
\sigma_j \le 0,
\qquad j=m+1,\ldots,n,
$$

and there exists at least one non-basic variable $x_k$ such that

$$
\sigma_k = 0,
$$

then introducing $x_k$ into the basis may produce another basic feasible solution with the same objective value. In this case, the linear programming problem has infinitely many optimal solutions, because every convex combination of two optimal solutions is also optimal.

#### Criterion for an Unbounded Solution

If there exists a non-basic variable $x_k$ such that

$$
\sigma_k > 0,
$$

and all coefficients in its corresponding pivot column satisfy

$$
a_{ik}^{\prime} \le 0,
\qquad i=1,2,\ldots,m,
$$

then no valid leaving variable can be selected by the minimum ratio test. Therefore, $x_k$ can increase indefinitely while maintaining feasibility, and the objective value can increase without bound.

In this case, the linear programming problem is unbounded.

#### Key Point

The reduced cost $\sigma_j$ measures the marginal change in the objective value when a non-basic variable $x_j$ is introduced into the basis.

This idea is especially important for later topics such as **column generation**, where the pricing subproblem searches for new variables, or columns, with favorable reduced costs.



### 3.2.4 Artificial Variable Methods

When an initial basic feasible solution is not directly available, artificial variables can be added.

Two commonly used methods are:

1. **Big-M method**  
   Artificial variables are penalized by a very large coefficient \(M\) in the objective function.

2. **Two-phase method**  
   Phase I minimizes the sum of artificial variables to find an initial feasible basis. Phase II then solves the original objective function starting from the feasible basis obtained in Phase I.

### 3.2.5 Criterion for Infeasibility

When artificial variables are introduced, infeasibility can be detected after simplex iterations.

If, in the final tableau, all optimality conditions are satisfied but at least one artificial variable remains in the basis with a nonzero value, then the original problem has no feasible solution.

The original note includes the following textbook explanation:

![Criterion for infeasibility with artificial variables](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651299771586-b0e0b9ca-d7ba-4c8a-9a7f-91e8cc79bb8f.png)

### 3.2.6 Degeneracy

Degeneracy occurs when at least one basic variable in a basic feasible solution is equal to zero.

Degeneracy is important because it may cause simplex iterations that do not improve the objective value. In extreme cases, cycling may occur if pivot rules are not chosen carefully.

### 3.2.7 Matrix Description of the Simplex Method

The matrix form of the simplex method is extremely important. It helps clarify the relationship among simplex iterations, duality theory, sensitivity analysis, and later decomposition methods such as column generation.

Consider the standard-form LP:

```math
\max \quad c^\top x
```

```math
\text{s.t.} \quad Ax = b,
```

```math
x \ge 0.
```

Suppose the columns of \(A\) are divided into a basis matrix \(B\) and a non-basis matrix \(N\). The corresponding variables and cost coefficients are denoted by:

```math
x =
\begin{bmatrix}
x_B \\
x_N
\end{bmatrix},
\quad
c =
\begin{bmatrix}
c_B \\
c_N
\end{bmatrix}.
```

The constraint can be written as:

```math
Bx_B + Nx_N = b.
```

Solving for the basic variables gives:

```math
x_B = B^{-1}b - B^{-1}Nx_N.
```

Substituting this into the objective function gives:

```math
z = c_B^\top B^{-1}b
+ \left(c_N^\top - c_B^\top B^{-1}N\right)x_N.
```

Therefore, the reduced cost vector of non-basic variables is:

```math
\bar{c}_N^\top = c_N^\top - c_B^\top B^{-1}N.
```

For a maximization problem, the current basic feasible solution is optimal if:

```math
\bar{c}_N \le 0.
```

The original note includes the corresponding textbook matrix derivation:

![Matrix description of the simplex method](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300088271-2ffd8110-1f6a-4c00-9ff1-ad918f918c0a.png)

![Simplex tableau and matrix representation](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651307275410-b851cf42-5f1e-45b1-892b-0954dd0ecb79.png)

#### Matrix Representation of Reduced Costs

A particularly important formula is:

```math
\bar{c}_N^\top = c_N^\top - c_B^\top B^{-1}N.
```

If slack variables are included, their reduced costs can also be expressed using \(B^{-1}\). This is why the inverse of the basis matrix plays such a central role in simplex computations.

The original note highlights this point:

![Matrix representation of reduced costs](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300145425-2f1c6558-932c-4f44-83b1-1a65537741f7.png)

This formula is also the bridge to column generation:

> In column generation, the pricing subproblem essentially tries to find a new column whose reduced cost can improve the master problem.

### 3.2.8 Dual Simplex Method

#### Principle

In the primal simplex method, the algorithm maintains primal feasibility and gradually improves dual feasibility through pivot operations.

The dual simplex method works in the opposite way:

- It maintains dual feasibility.
- It starts from a primal infeasible basis.
- Through pivot operations, it gradually restores primal feasibility.
- Once both primal feasibility and dual feasibility hold, optimality is obtained.

The original note gives the textbook explanation:

![Principle of the dual simplex method](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300524325-0ecbefa7-7dba-4c98-8988-54a1c0b68b8a.png)

#### Advantages

The dual simplex method has several advantages:

1. The initial solution does not need to be primal feasible.
2. In some cases, artificial variables can be avoided.
3. When the number of variables is smaller than the number of constraints, solving the dual problem may reduce computational effort.
4. It is useful in sensitivity analysis.
5. It is often used in cutting-plane methods for integer programming.

However, the dual simplex method is not always easy to apply independently, because it may be difficult to find an initial dual feasible basis.

The original note includes the following summary:

![Advantages of the dual simplex method](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300600003-63deac18-d0ea-4f5e-bfb8-6c939bb8f01b.png)

---

## 3.3 Duality Theory of Linear Programming

### 3.3.1 Essence of LP Duality

Linear programming duality can be viewed as a special equivalent form of **Lagrangian duality** applied to linear programming problems.

This viewpoint is helpful because it connects LP duality with more general optimization theory.

### 3.3.2 Relationship Between Primal and Dual Problems

For a maximization primal problem with “less than or equal to” constraints, the corresponding dual problem is usually a minimization problem.

A typical primal-dual pair is:

Primal problem:

```math
\max \quad c^\top x
```

```math
\text{s.t.} \quad Ax \le b,
```

```math
x \ge 0.
```

Dual problem:

```math
\min \quad b^\top y
```

```math
\text{s.t.} \quad A^\top y \ge c,
```

```math
y \ge 0.
```

The original note includes several textbook tables summarizing the primal-dual correspondence:

![Primal-dual relationship](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300708147-0585e6f2-a02a-483d-bf57-6719dd881de8.png)

![Primal-dual correspondence table](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300731415-b22e5f14-b12d-45af-b2da-d3b68ed34fc1.png)

![General transformation rules between primal and dual problems](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651300798370-f394f190-f0d9-4b31-8903-d6f4dc58c542.png)

> Note: If some of the image links from the original Yuque export fail after migration, these tables should be rewritten manually using LaTeX or Markdown tables.

### 3.3.3 Basic Properties of LP Duality

The following properties are valid for linear programming problems.

#### 1. Symmetry

The dual of the dual problem is the primal problem.

#### 2. Weak Duality

For a primal maximization problem and its dual minimization problem, if \(x\) is primal feasible and \(y\) is dual feasible, then:

```math
c^\top x \le b^\top y.
```

The original note includes the corresponding textbook statement:

![Weak duality](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651301035940-268cdedf-7bfa-4bec-b3e4-85676b5ef6cb.png)

#### 3. Unboundedness

If the primal problem is unbounded, then the dual problem is infeasible.

Similarly, if the dual problem is unbounded, then the primal problem is infeasible.

#### 4. Optimality from Equal Objective Values

If \(x\) is primal feasible, \(y\) is dual feasible, and:

```math
c^\top x = b^\top y,
```

then both \(x\) and \(y\) are optimal solutions.

The original note includes the following statement:

![Optimality from equal objective values](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651301106362-500410ba-6da2-42ed-aebf-3ebc57cb0347.png)

#### 5. Strong Duality Theorem

If the primal problem has an optimal solution, then the dual problem also has an optimal solution, and their optimal objective values are equal.

#### 6. Complementary Slackness

Complementary slackness describes the relationship between primal slack variables and dual variables at optimality.

For a primal-dual optimal pair, if a primal constraint is not tight, then the corresponding dual variable must be zero. Conversely, if a dual variable is positive, then the corresponding primal constraint must be tight.

The original note includes the following textbook statement:

![Complementary slackness](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651301205685-6ab49667-56e8-44ea-bba6-b41a4d8eb381.png)

#### 7. Relationship Between Primal Reduced Costs and Dual Basic Solutions

This is one of the most important points in this note:

> The reduced-cost row of the primal simplex tableau corresponds to a basic solution of the dual problem.

The original note refers to the following paper for a more detailed explanation:

> Zhan Bingjun, *On the Role of Reduced Costs in Solving Dual Problems*.

The original note includes the following figure:

![Relationship between reduced costs and dual basic solutions](https://cdn.nlark.com/yuque/0/2022/png/12963648/1651301245595-49b6ce64-b8f8-462e-8b79-1ff04208b9f2.png)

This observation is important because it explains why simplex optimality can be interpreted through primal-dual relationships.

In the primal simplex method:

- The right-hand side column represents a primal basic solution.
- The reduced-cost row represents information related to a dual basic solution.
- When both primal feasibility and dual feasibility are satisfied, the current solution is optimal.

This perspective is also useful for understanding:

1. Dual simplex method
2. Sensitivity analysis
3. Cutting-plane methods
4. Column generation
5. Reduced-cost-based pricing problems

### 3.3.4 Shadow Price

A shadow price describes how the optimal objective value changes when the right-hand side of a constraint changes marginally.

In LP duality, the optimal value of a dual variable can often be interpreted as the shadow price of the corresponding primal constraint.

For example, if a resource constraint has a positive shadow price, then increasing the availability of that resource may improve the optimal objective value. If the shadow price is zero, then that resource is not currently limiting the optimal solution.

---

## 4. Key Takeaways

This note records the LP foundations needed for studying integer linear programming and decomposition algorithms.

The most important points are:

1. The simplex method moves among basic feasible solutions.
2. A basic feasible solution corresponds to a vertex of the feasible region.
3. Reduced costs determine whether a non-basic variable can improve the objective value.
4. The matrix form of reduced costs is:

   ```math
   \bar{c}_N^\top = c_N^\top - c_B^\top B^{-1}N.
   ```

5. Artificial variables are used when an initial feasible basis is not directly available.
6. The dual simplex method maintains dual feasibility and restores primal feasibility.
7. LP duality is closely related to Lagrangian duality.
8. Reduced costs in the primal simplex tableau are closely connected to basic solutions of the dual problem.
9. Shadow prices are interpretations of optimal dual variables.
10. These ideas are fundamental for column generation, cutting-plane methods, branch-and-bound, and other ILP algorithms.

---

## 5. Suggested Follow-up Reading

- Simplex Method
- Dual Simplex Method
- Linear Programming Duality
- Sensitivity Analysis
- Column Generation
- Dantzig-Wolfe Decomposition
- Branch-and-Bound
- Cutting-Plane Methods
