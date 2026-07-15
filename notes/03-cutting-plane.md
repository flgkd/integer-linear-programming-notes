# Integer Linear Programming Notes 03: Cutting Plane

> Author: Hang Li (@flgkd)
>
> Note: Titles marked with ⭐ highlight key concepts and important summaries.

## 1. Preface

This note introduces another method for solving integer linear programming problems: the **cutting plane method**.

We first briefly introduce the basic idea behind cutting planes.

When solving the LP relaxation of an integer linear programming problem, it is natural to think:

> How nice would it be if the optimal solution of the relaxation happened to be an integer solution?

In fact, there is indeed a class of integer linear programming problems with this property. For such problems, the coefficient matrix is **totally unimodular**.

In other words, **if the coefficient matrix is totally unimodular and the right-hand-side vector is integral, then every vertex of the LP relaxation is integral. In that case, solving the LP relaxation can directly give an optimal solution to the original integer programming problem**.

However, total unimodularity is a rather strong condition and does not apply to general integer linear programming problems. Therefore, we will not discuss it in detail here.

Now consider the figure below. Suppose the feasible region of an integer linear programming problem consists of all the green points inside region $X_2$. These green points are integer points, and we denote the set of these integer feasible points by $X_1$. The region $X_2$ is the feasible region of the LP relaxation.

<p align="center">
  <img src="../figures/chapter-03/chapter-03-fig1.png" alt="Convex Hull of Integer Feasible Points" width="600">
</p>

<p align="center">
  Convex Hull of Integer Feasible Points.
</p>

If we want the LP relaxation to directly return an optimal integer solution, then, according to the principle of the simplex method, **we only need to remove the redundant parts of $X_2$ and keep the smallest convex hull containing $X_1$**.

That is, we want to reduce $X_2$ to the convex hull of $X_1$, denoted by

$$
\mathrm{conv}(X_1).
$$

At this point, the feasible region of the relaxation changes from $X_2$ to $\mathrm{conv}(X_1)$. If we solve the LP relaxation over this new feasible region, the optimal solution will also be an optimal integer solution.

However, this idea is beautiful in theory but difficult in practice. **Directly reducing the feasible region of the LP relaxation to the smallest convex hull of the integer feasible points is very hard**. Equivalently, **finding the convex hull of an integer point set is generally difficult**.

**The basic idea of the cutting plane method** is therefore:

> Gradually remove redundant parts of the LP relaxation feasible region by adding linear inequalities, called cutting planes, until the relaxation becomes close to the convex hull of the integer feasible points.

---

## 2. Basic Idea of Cutting Plane⭐

Consider an integer programming problem

$$
(IP) \quad \min \quad c^T x
$$

subject to

$$
x \in X,
$$

where

$$
X = \{ x \mid Ax \le b, \ x \in \mathbf{Z}_+^n \}.
$$

The LP relaxation is obtained by replacing the integer restriction with continuous variables:

$$
(LP) \quad \min \quad c^T x
$$

subject to

$$
x \in P,
$$

where

$$
P = \{ x \mid Ax \le b, \ x \ge 0 \}.
$$

Clearly,

$$
X \subseteq P.
$$

The convex hull of $X$ is denoted by

$$
conv(X).
$$

The ideal linear programming formulation would be

$$
(CP) \quad \min \quad c^T x
$$

subject to

$$
x \in conv(X).
$$

If we can describe $conv(X)$ exactly, then problem $(CP)$ is equivalent to the original integer programming problem $(IP)$.

However, the difficulty is that an exact linear inequality description of $conv(X)$ is usually hard to obtain.

Therefore, the cutting plane method does the following:

1. solve the LP relaxation;
2. if the solution is fractional, find a valid inequality that cuts it off;
3. add this inequality to the LP relaxation;
4. repeat.

The added valid inequality is called a **cut** or a **cutting plane**.

The cut must satisfy two requirements:

1. it must cut off the current fractional LP solution;
2. it must not remove any integer feasible solution.

This is the core logic of cutting planes.

---

## 3. Valid Inequalities⭐

The key point of the previous section is simple:

> Obtaining the exact convex hull $conv(X)$ is usually very difficult.

Therefore, instead of trying to describe $conv(X)$ completely at once, we add valid inequalities step by step.

### 3.1 Definition of Valid Inequality

Let $X$ be a feasible set. An inequality

$$
\alpha^T x \le \beta
$$

is called a **valid inequality** for $X$ if every point in $X$ satisfies it.

That is, for every $x \in X$,

$$
\alpha^T x \le \beta.
$$

Geometrically, this means that the whole set $X$ lies in the half-space

$$
\{x \mid \alpha^T x \le \beta\}.
$$

A valid inequality is useful because it can remove fractional points from the LP relaxation while keeping all integer feasible points.

In the cutting plane method, the inequality we add should satisfy two requirements:

1. it is valid for the integer feasible set $X$;
2. it cuts off the current fractional optimal solution of the LP relaxation.

### 3.2 Several Examples of Valid Inequalities

This subsection follows the original idea of using simple examples to understand valid inequalities before introducing systematic generation methods.

#### Example 1: A 0-1 Set

Consider a 0-1 feasible set $X$.

Suppose that, by analyzing the constraints, we know that every feasible 0-1 solution must satisfy

$$
x_2 + x_4 \ge 1.
$$

Then

$$
x_2 + x_4 \ge 1
$$

is a valid inequality for $X$.

The important point is not that this inequality comes from the original formulation directly. The important point is that every feasible integer solution satisfies it.

Therefore, adding it to the LP relaxation does not remove any feasible integer solution.

#### Example 2: A Mixed-Integer Set

For a mixed-integer set, valid inequalities can also be obtained from geometric observation.

In a two-dimensional example, the feasible set may consist of several line segments corresponding to different integer values of one variable. From the figure, one can sometimes directly observe the convex hull and verify that a certain linear inequality is valid for all feasible points.

This example shows that valid inequalities are not limited to pure integer sets. They are also important for mixed-integer programming.

#### Example 3: Convex Hull of a Low-Dimensional Integer Set

For a low-dimensional integer set, one can sometimes draw all feasible integer points and obtain their convex hull manually.

In such cases, the convex hull can be described by the linear inequalities corresponding to its boundary edges.

This gives a useful geometric interpretation:

> valid inequalities are the half-space descriptions that contain all integer feasible points.

For low-dimensional examples, this process is often easy to visualize.

For high-dimensional integer sets, however, describing the convex hull exactly is usually very difficult. This is why we need systematic methods for generating valid inequalities.

### 3.3 Chvatal-Gomory Method: A Common Way to Generate Valid Inequalities⭐

The **Chvatal-Gomory method**, often abbreviated as the **C-G method**, is a common way to generate valid inequalities for integer programming problems.

It is based on a very simple observation:

> If an integer quantity is less than or equal to a real number, then it is also less than or equal to the floor of that real number.

For example, if $a$ is an integer and

$$
a \le 5.7,
$$

then we must have

$$
a \le 5.
$$

This simple rounding idea is the foundation of the Chvatal-Gomory method.

#### 3.3.1 An Illustrative Example

Suppose the integer feasible set is

$$
X = P \cap \mathbf{Z}_+^n,
$$

where $P$ is the feasible region of the LP relaxation.

The basic idea is as follows.

First, take a nonnegative linear combination of several inequalities defining $P$. This gives a new inequality that is valid for $P$.

Then, because the variables in $X$ are required to be integers, we may be able to round the coefficients or the right-hand side to obtain a stronger inequality for $X$.

More specifically, suppose we obtain an inequality

$$
\sum_{j=1}^n \gamma_j x_j \le \delta.
$$

Since $x_j \ge 0$, replacing each coefficient $\gamma_j$ by a smaller coefficient preserves validity for the LP relaxation:

$$
\sum_{j=1}^n \lfloor \gamma_j \rfloor x_j \le \delta.
$$

Now, for $x \in \mathbf{Z}_+^n$, the left-hand side is an integer. Therefore, we can round down the right-hand side and obtain

$$
\sum_{j=1}^n \lfloor \gamma_j \rfloor x_j \le \lfloor \delta \rfloor.
$$

This inequality is valid for the integer feasible set $X$.

#### 3.3.2 Three Steps of the Chvatal-Gomory Method

Consider the integer feasible set

$$
X = \{x \mid Ax \le b,\ x \in \mathbf{Z}_+^n\}.
$$

Its LP relaxation is

$$
P = \{x \mid Ax \le b,\ x \ge 0\}.
$$

Let $A_j$ denote the $j$-th column of $A$.

The Chvatal-Gomory method generates a valid inequality for $X$ in the following three steps.

**Step 1. Construct a valid inequality for $P$.**

Choose a nonnegative multiplier vector

$$
u \ge 0.
$$

Taking a nonnegative linear combination of the inequalities $Ax \le b$ gives

$$
u^T A x \le u^T b.
$$

Equivalently,

$$
\sum_{j=1}^n (u^T A_j)x_j \le u^T b.
$$

This inequality is valid for $P$.

**Step 2. Round down the coefficients.**

Since $x_j \ge 0$ and

$$
\lfloor u^T A_j \rfloor \le u^T A_j,
$$

the following inequality is also valid for $P$:

$$
\sum_{j=1}^n \lfloor u^T A_j \rfloor x_j \le u^T b.
$$

**Step 3. Round down the right-hand side.**

For every $x \in X$, the variables $x_j$ are integers. Since the coefficients

$$
\lfloor u^T A_j \rfloor
$$

are also integers, the left-hand side

$$
\sum_{j=1}^n \lfloor u^T A_j \rfloor x_j
$$

is an integer.

Therefore, we can round down the right-hand side and obtain

$$
\sum_{j=1}^n \lfloor u^T A_j \rfloor x_j \le \lfloor u^T b \rfloor.
$$

This is a valid inequality for the integer feasible set $X$.

When this inequality cuts off the current fractional LP solution, it is called a **Chvatal-Gomory cut**, or simply a **C-G cut**.

#### 3.3.3 Applying the C-G Method to an Example

The C-G method can also be understood through a simple geometric example.

Suppose that, after taking a suitable nonnegative linear combination of the original inequalities, we obtain a valid inequality for the LP relaxation:

$$
x_1 + x_2 \le \frac{76}{11}.
$$

For the LP relaxation, this inequality is valid.

However, for the integer feasible set, $x_1$ and $x_2$ are integers. Therefore,

$$
x_1 + x_2
$$

must also be an integer.

Since

$$
x_1 + x_2 \le \frac{76}{11},
$$

we can strengthen it to

$$
x_1 + x_2 \le 6.
$$

This new inequality is valid for all integer feasible points, but it may cut off fractional points in the LP relaxation.

Therefore, it is a valid cutting plane.

#### 3.3.4 Geometric Interpretation

Geometrically, the inequality

$$
x_1 + x_2 \le \frac{76}{11}
$$

defines a half-space that contains the feasible region of the LP relaxation.

Because the original problem requires integer solutions, there is no integer feasible point satisfying

$$
6 < x_1 + x_2 \le \frac{76}{11}.
$$

Therefore, the boundary can be shifted inward from

$$
x_1 + x_2 = \frac{76}{11}
$$

to

$$
x_1 + x_2 = 6
$$

without removing any integer feasible point.

This is the geometric meaning of the C-G method:

> use integrality to shift a valid inequality inward, thereby cutting away fractional points.

When the integer coefficients are relatively prime, this inward shift may make the new hyperplane touch integer feasible points. In that case, the cut can be quite strong.

If the coefficients have a common divisor, the rounded inequality is still valid, but it may not be as tight geometrically.

#### 3.3.5 Property and Corollary

The most important property is:

> A Chvatal-Gomory inequality is valid for the integer feasible set.

The reason is straightforward:

1. the aggregated inequality is valid for the LP relaxation;
2. rounding down coefficients preserves validity because the variables are assumed to be nonnegative;
3. rounding down the right-hand side is valid because the left-hand side is integer for integer solutions.

A classical theoretical result further states:

> For rational polyhedra, a classical result states that the integer hull can be obtained after finitely many rounds of Chvatal-Gomory closures. Equivalently, valid inequalities describing the integer hull can be derived through repeated applications of the Chvatal-Gomory method.

This result shows the theoretical power of the C-G method.

However, it does not mean that the method is always efficient in practice. In computation, the key challenge is not only to generate valid cuts, but to generate strong and useful cuts.

---

## 4. Gomory Cutting Plane Method⭐

### 4.1 Basic Idea

The cutting plane method is similar to Branch and Bound in one important aspect:

> both methods transform the original integer programming problem into a sequence of linear programming problems.

The basic idea of the cutting plane method is:

1. first ignore the integer restrictions and solve the LP relaxation;
2. if the LP optimum is fractional, add a linear inequality that cuts off this fractional point;
3. make sure that the added inequality does not cut off any feasible integer solution;
4. solve the tightened LP relaxation again;
5. repeat this process until an integer optimal solution is obtained.

The added linear inequality is called a **cutting plane**.

The cutting plane removes a part of the relaxation feasible region. The removed part should contain the current fractional LP solution but no feasible integer solution.

The method introduced by R. E. Gomory is called the **Gomory cutting plane method**.

### 4.2 General Steps of the Cutting Plane Algorithm

Consider the integer programming problem

$$
(IP) \quad \min \quad c^T x
$$

subject to

$$
x \in X.
$$

Let $P$ be the feasible region of its LP relaxation.

The cutting plane algorithm can be described as follows.

**Initialization.**

Set

$$
t=0,\qquad P_0=P.
$$

**Iteration.**

Solve the linear programming problem

$$
\min \quad c^T x
$$

subject to

$$
x \in P_t.
$$

Let its optimal solution be $x^t$.

If

$$
x^t \in \mathbf{Z}^n,
$$

then $x^t$ is an optimal solution of the original integer programming problem. The algorithm terminates.

Otherwise, find a valid inequality

$$
\alpha^T x \le \beta
$$

such that

$$
\alpha^T x \le \beta
$$

is valid for all integer feasible solutions, but

$$
\alpha^T x^t > \beta.
$$

That is, the inequality cuts off the current fractional LP optimum.

Then update the relaxation by adding this inequality:

$$
P_{t+1}=P_t \cap \{x \mid \alpha^T x \le \beta\}.
$$

Increase $t$ and repeat.

### 4.3 Gomory Fractional Cut: A Valid Inequality That Cuts Off the Fractional LP Optimum

The key step in the algorithm is how to construct a valid inequality that cuts off the current fractional optimal solution of the LP relaxation.

The **Gomory fractional cut** is a classical and relatively easy way to construct such a cut for pure integer programming.

Suppose a row of the final simplex tableau is written as

$$
x_i + \sum_{j \in N} a_{ij}x_j = b_i,
$$

where:

- $x_i$ is a basic variable;
- $N$ is the set of nonbasic variables;
- all variables are required to be nonnegative integers;
- $b_i$ is not an integer.

At the current LP basic solution, all nonbasic variables are zero:

$$
x_j=0,\qquad j\in N.
$$

Therefore,

$$
x_i=b_i.
$$

If $b_i$ is fractional, then the current LP solution is not integer feasible.

For any real number $r$, define its fractional part as

$$
f(r)=r-\lfloor r\rfloor.
$$

Let

$$
f_i=f(b_i)
$$

and

$$
f_{ij}=f(a_{ij}).
$$

Since $b_i$ is fractional,

$$
0<f_i<1.
$$

The Gomory fractional cut generated from this row is

$$
\sum_{j\in N} f_{ij}x_j \ge f_i.
$$

Equivalently, it can be written as

$$
-\sum_{j\in N} f_{ij}x_j \le -f_i.
$$

This inequality cuts off the current fractional LP solution because at the current basic solution,

$$
x_j=0,\qquad j\in N,
$$

so

$$
\sum_{j\in N} f_{ij}x_j=0<f_i.
$$

Therefore, the current fractional solution violates the cut.

At the same time, the inequality is valid for all integer feasible solutions. This is why it can be added safely.

### 4.4 Cutting Plane Method Based on Gomory Fractional Cuts

The Gomory cutting plane method based on Gomory fractional cuts works as follows.

Step 1. Solve the LP relaxation of the integer programming problem.

Step 2. If the optimal solution is integer feasible, stop. This solution is optimal for the original integer programming problem.

Step 3. If the optimal solution is not integer, choose a row of the final simplex tableau whose basic variable has a fractional value.

Step 4. Generate a Gomory fractional cut from this row.

Step 5. Add the cut to the LP relaxation.

Step 6. Reoptimize the LP relaxation.

Step 7. Repeat the process until an integer optimal solution is obtained.

This can be summarized as:

```text
Solve LP relaxation
→ find a fractional basic variable
→ generate a Gomory fractional cut
→ add the cut
→ reoptimize
→ repeat
```

After adding a Gomory cut, the current basis may become infeasible for the new LP relaxation. Therefore, the **dual simplex method** is often used to reoptimize the problem efficiently.

### 4.5 Gomory Cuts Are Essentially Chvatal-Gomory Valid Inequalities

Gomory fractional cuts are closely related to Chvatal-Gomory inequalities.

In fact, if a Gomory cut is rewritten in terms of the original variables, it can be interpreted as a Chvatal-Gomory valid inequality.

The reason is that a row of the final simplex tableau is itself obtained from linear combinations of the original constraints.

Therefore, when we take fractional parts and round the right-hand side, we are essentially applying the Chvatal-Gomory principle to a tableau row.

This gives the following important conceptual connection:

> Gomory fractional cuts are tableau-based Chvatal-Gomory cuts.

This also explains why Gomory cuts are valid for the integer feasible set.

### 4.6 Examples

#### 4.6.1 Example 1: A Pure Integer Programming Example

In a pure integer programming problem, after adding slack variables, all variables can be treated as integer variables if the original coefficients and right-hand sides are integers.

The general procedure is:

1. solve the LP relaxation and obtain the final simplex tableau;
2. identify a basic variable with a fractional value;
3. use the corresponding row to construct a Gomory fractional cut;
4. add the cut to the LP relaxation;
5. reoptimize;
6. repeat until the LP optimum becomes integer.

The example in the original note shows that one cut may not be enough. After the first Gomory cut is added, the new LP optimum may still be fractional. Then a second cut is generated and added.

After enough cuts are added, the relaxation may yield an integer optimal solution.

This illustrates an important point:

> the Gomory cutting plane method may need several rounds of cutting and reoptimization.

#### 4.6.2 Example 2: Geometric Interpretation

A Gomory cut can also be understood geometrically.

Suppose the LP relaxation has a fractional optimal point $A$. If we can find a line such as $CD$ that cuts off the fractional region containing $A$ while keeping all integer feasible points, then the tightened feasible region may have an integer point as a new vertex.

If the new LP optimum happens to be this integer vertex, then we have obtained the optimal integer solution.

This example emphasizes the geometric meaning of a cutting plane:

> a cut removes fractional parts of the relaxation feasible region, but preserves all integer feasible points.

#### 4.6.3 A Simple Summary of How to Derive a Cutting Equation

The derivation of a Gomory cutting equation can be summarized as follows.

Suppose a row of the final simplex tableau is

$$
x_i + \sum_{j\in N} a_{ij}x_j = b_i,
$$

where $x_i$ is a basic variable with fractional value $b_i$.

Decompose each coefficient and the right-hand side into integer and fractional parts:

$$
a_{ij}=\lfloor a_{ij}\rfloor + f_{ij},
$$

and

$$
b_i=\lfloor b_i\rfloor + f_i.
$$

Here,

$$
0\le f_{ij}<1,
$$

and

$$
0<f_i<1.
$$

Using the nonnegativity and integrality of all variables, we obtain the Gomory fractional cut

$$
\sum_{j\in N} f_{ij}x_j \ge f_i.
$$

This cut has two important properties:

1. it cuts off the current fractional LP optimum;
2. it does not cut off any integer feasible solution.

---

## 5. Mixed-Integer Cuts⭐

### 5.1 General Representation of a Mixed-Integer Linear Feasible Set

A mixed-integer linear programming feasible set can generally be written as

$$
X=\{(x,y)\mid Ax+Gy\le b,\ x\in \mathbf{Z}_+^n,\ y\in \mathbf{R}_+^p\}.
$$

Here:

- $x$ represents integer variables;
- $y$ represents continuous variables;
- $A$ and $G$ are coefficient matrices;
- $b$ is the right-hand-side vector.

For this kind of mixed-integer set, directly applying the C-G method is not always sufficient.

Therefore, we need valid inequalities that are designed for mixed-integer sets.

The main point is:

> mixed-integer cuts exploit the discreteness of integer variables while also considering the flexibility of continuous variables.

### 5.2 Low-Dimensional Two-Variable Mixed-Integer Sets

The original note first considers simple two-dimensional mixed-integer sets.

The purpose is to understand mixed-integer cuts geometrically before introducing more general formulas.

#### 5.2.1 Property 1: A Simple Mixed-Integer Cut

Consider a simple mixed-integer set

$$
X=\{(x,y)\mid x+y\ge b,\ x\in \mathbf{Z},\ y\ge 0\},
$$

where $b$ is not an integer.

Let

$$
f=b-\lfloor b\rfloor.
$$

Then

$$
0<f<1.
$$

Since $x$ must be integer, the feasible points have a special structure.

A valid inequality for this set is

$$
x+\frac{1}{f}y\ge \lceil b\rceil.
$$

This inequality is stronger than the original relaxation in the region where $x$ is below the next integer level.

It is valid because if $x\ge \lceil b\rceil$, then the inequality is immediately satisfied. If $x\le \lfloor b\rfloor$, then the original constraint requires

$$
y\ge b-x,
$$

which is strong enough to imply the cut.

#### 5.2.2 Corollary 1

The same idea can be transformed into equivalent valid inequalities for closely related mixed-integer sets.

The main message is that the fractional part

$$
f=b-\lfloor b\rfloor
$$

determines the slope of the cut.

This is the key geometric feature of simple mixed-integer cuts.

#### 5.2.3 Property 2: A Two-Variable Extension

For a set with one integer variable and two continuous variables, the same logic can be extended.

The resulting valid inequality combines:

- the integer rounding structure of the integer variable;
- the continuous compensation provided by the continuous variables.

The proof usually reduces the problem to the simple mixed-integer set in Property 1 and then rearranges terms.

### 5.3 Valid Inequalities for General Single-Constraint Mixed-Integer Sets

The previous properties explain the low-dimensional intuition.

Now consider a more general single-constraint mixed-integer set.

A typical form is

$$
X=\{(x,y) \mid \sum_{j\in I} a_jx_j+\sum_{j\in C} g_jy_j\le b,\ x_j\in \mathbf{Z}_+,\ y_j\ge 0\}.
$$

Here:

- $I$ is the index set of integer variables;
- $C$ is the index set of continuous variables.

The goal is to derive valid inequalities that are stronger than the LP relaxation.

The coefficients and the fractional part of the right-hand side determine the final form of the mixed-integer cut.

In this note, we do not reproduce all technical variants of single-constraint mixed-integer cuts. Instead, we focus on the main idea and then introduce the Gomory mixed-integer cut, which is more commonly used in MILP solvers.

### 5.4 Gomory Mixed-Integer Cut

The **Gomory mixed-integer cut**, often abbreviated as the **GMI cut**, extends Gomory fractional cuts to mixed-integer programming.

Suppose a tableau row is written as

$$
x_B=b-\sum_{j\in I} a_jx_j-\sum_{j\in C} g_jy_j.
$$

Here:

- $x_B$ is a basic integer variable;
- $x_j$ are nonbasic integer variables;
- $y_j$ are nonbasic continuous variables;
- $I$ is the set of nonbasic integer variables;
- $C$ is the set of nonbasic continuous variables;
- $b$ is fractional.

Let

$$
f_0=b-\lfloor b\rfloor.
$$

For each integer nonbasic coefficient $a_j$, define

$$
f_j=a_j-\lfloor a_j\rfloor.
$$

A common form of the GMI cut is

$$
\sum_{j\in I,\ f_j\le f_0}\frac{f_j}{f_0}x_j
+
\sum_{j\in I,\ f_j>f_0}\frac{1-f_j}{1-f_0}x_j
+
\sum_{j\in C,\ g_j\ge 0}\frac{g_j}{f_0}y_j
+
\sum_{j\in C,\ g_j<0}\frac{-g_j}{1-f_0}y_j
\ge 1.
$$

This inequality is valid for the mixed-integer feasible set and cuts off the current fractional LP solution.

Different sign conventions for the tableau row may lead to slightly different-looking formulas, but the idea is the same:

> use a fractional basic integer variable to derive a valid inequality that respects both integer and continuous variables.

### 5.5 Examples

The examples in the original note compare pure integer and mixed-integer cases.

One important observation is:

> when some variables are continuous, the pure Gomory fractional cut is no longer enough by itself; the cut must account for continuous variables as well.

This is why the Gomory mixed-integer cut is important in mixed-integer linear programming.

In practice, GMI cuts are widely used in modern MILP solvers.

---

## 6. Cutting Plane vs Branch and Bound

Both cutting planes and Branch and Bound solve ILP problems through LP relaxations, but they improve the relaxation in different ways.

| Method | Main idea | What changes? |
|---|---|---|
| Branch and Bound | Split the problem into subproblems | The feasible region is branched into smaller regions |
| Cutting Plane | Add valid inequalities | The LP relaxation is tightened by cuts |
| Branch and Cut | Combine both | Branching and cutting are used together |

Branch and Bound is easier to understand as a search-tree method.

Cutting Plane is more polyhedral: it tries to describe the convex hull more accurately by adding inequalities.

Modern MILP solvers usually combine both ideas, leading to **Branch and Cut**.

---

## 7. Key Takeaways

1. The LP relaxation of an ILP is usually too large because it contains fractional points.
2. The ideal relaxation is the convex hull of integer feasible solutions, denoted by $conv(X)$.
3. Describing $conv(X)$ exactly is usually difficult.
4. A valid inequality is an inequality satisfied by all integer feasible solutions.
5. A useful cut removes fractional LP solutions without removing integer feasible solutions.
6. The Chvatal-Gomory method generates valid inequalities by aggregating inequalities and using integrality to round.
7. Gomory fractional cuts are derived from fractional rows of the final simplex tableau.
8. A Gomory cut is valid for all integer feasible solutions but cuts off the current fractional LP optimum.
9. Gomory cuts can be viewed as tableau-based Chvatal-Gomory cuts.
10. Mixed-integer cuts extend the cutting plane idea to problems with both integer and continuous variables.
11. Gomory mixed-integer cuts are important in practical MILP solvers.
12. Modern solvers often combine cutting planes with Branch and Bound, leading to Branch and Cut.

## References

1. Textbook Editorial Group of *Operations Research*. *Operations Research*, 4th ed. Beijing: Tsinghua University Press, 2012. ISBN: 978-7-302-28879-4.（《运筹学》教材编写组：《运筹学》第4版，北京：清华大学出版社，2012年，ISBN：978-7-302-28879-4）
   
2. Sun, Xiaoling, and Duan Li. *Integer Programming*. Beijing: Science Press, 2010. ISBN: 978-7-03-029380-0.（孙小玲、李端：《整数规划》，北京：科学出版社，2010年，ISBN：978-7-03-029380-0）


## Suggested Follow-up Reading

- Chvatal-Gomory Cuts
- Gomory Fractional Cuts
- Gomory Mixed-Integer Cuts
- Facets and Convex Hulls
- Branch and Cut
- Modern MILP Solvers such as CPLEX and Gurobi

These topics are useful for later notes on Branch and Cut and Branch and Price.
