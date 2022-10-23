---
layout: page
title: Dynamic programming
summary: "An advanced way for designing solutions. What features does a problem need to display
so that this type of solution can be applied? If those are satisfied we can transform algorithms with
exponential time complexity to polynomial ones, by sacrificing on space."
---

## Name

The origin of the term dynamic programming is an interesting one. Two legends are
in circulation. One from the author [Richard Bellman](https://en.wikipedia.org/wiki/Richard_E._Bellman)
stating that it was designed to hide the mathematical background of
his work from his employer at the time. Choosing dynamic as it has no negative
connotations and to refer to the time varying, multistage nature of the problems.
Programming was chosen because it didn't contain the word mathematics, and it
sounded better than planning.
The other legend refers to an attempt to one up the previously existing methods
named linear programming.

## Applicability

Dynamic programming is not an algorithm that solves a problem. It is a design
technique that speeds up a specific set of problems. When properly applied it can
transform algorithms from exponential 𝒪(xⁿ) to polynomial 𝒪(n<sup>x</sup>) time
complexity for a comparably small, but usually not negligible space requirement of
around 𝒪(n<sup>x</sup>).

It is most commonly used for optimization problems. In these problems the goal is
to either maximize of minimize the value function in question. The result is
one of the possible optimal solutions in a space where more than one can exist.

There are 2 requirements that need to be satisfied before dynamic programming can
be applied:

- The problem has to have an **optimal substructure**
- The solution must be constructed of **overlapping subproblems**

### Optimal substructure

If an optimal solution to the problem consists of <u>independent</u> and
<u>optimal solutions</u> to its subproblems then we say the problem exhibits an
optimal substructure.

There are 3 steps used while testing for optimal substructure:

1. Show that the solution to the problem consists of making a choice after which
the problem space separates into one or more independent subproblems, where each
exhibits the same properties as the original problem.
2. Assume that for a given problem there exists an optimal choice.
3. Show using the "cut-and-paste" method that given an optimal solution to a
problem the resulting subproblem solutions have to be optimal as well.

> Independent optimal solutions: For a problem **A** comprised of subproblems
**B** and **C**, the optimal solutions for **B** and **C** are said to be
independent if they do not use up resources from on one
another. I.e. the solution for B does note use sub-solutions from C and vice-versa.

> Cut-and-paste: Suppose that each subproblem solution is non-optimal. By cutting
them out and substituting an optimal solution, demonstrate that you get a better
solution to the original problem, which contradicts your original assumption of
already having an optimal solution to your original problem.

### Overlapping subproblems

Subproblems are considered overlapping when the solution for them share a common
set of subproblems among them.

The implication of this requirement is that the set of distinct subproblems
is small compared to the total set of subproblems that would need to be visited
while applying a brute force approach in solving the problem.

## Design steps

Developing an algorithm using dynamic programming is not a simple task. Especially
as dynamic programming itself isn't a way of designing per se, more of a way of
redesigning an already existing solution to a format which is more efficient.

The steps for applying dynamic programming:

1. Understand and solve the problem in a recursive fashion.
2. Reformat the recursive algorithm to use dynamic programming.
3. (Optional) Reconstruct the optimal solution.

### 1. Understand and solve the problem in a recursive fashion

The problem we are trying to solve should lead us to an observation such as this:

"To solve (optimize) **problem** **X** I have to make a globally optimal **choice**
while considering the result of all **subproblems X<sub>i</sub>** for any of which
I do not know the answers for."

In other words we can define how an optimal solution will look like from the
top-down. The solution on the top level relies on the solutions of the
subproblems having the same basic format on lower levels.

### 2. Reformat the recursive algorithm to use dynamic programming

There are two ways to do this. Use either a top-down approach or a bottom-up one.
In essence both of them are the same, they both solve the problems from the bottom
up, they are just defined differently. In practice the bottom-up approach is
preferable as it will most likely need fewer supporting structures to be
maintained, thus reducing time and space complexity.

#### Top-down

A top-down approach is the same as the above recursive implementation with the
added step of **memoization**. The method entails memoizing (memorizing) the
solution to an already solved sub-problem, so when it is encountered again, it
needs not be recomputed.

> Memoization could be called memorization as the values are memorized, but
historically this step was called memoization, based on the Latin word [memorandum](https://en.wikipedia.org/wiki/Memoization#Etymology).

#### Bottom-up

The bottom-up approach is a bit different as the procedure has to be turned around.
Requires much more insight into how the algorithm is structured and what data has
to be calculated exactly when. Which sub-problems are dependencies to sub-problems
above them. With these questions answered one has to transform the algorithm such
that each sub-problem result is only calculated once and stored into an
appropriate container. By accessing the finally calculated value in this container
we receive the solution to the problem.

A noticeable difference between the top-down and bottom-up implementation is that
now there is no more recursion. The value is calculated in an iterative manner by
solving the problems in the direction from the smallest sub-problem towards the
original problem.

> In implementations the container used to store the values to the sub-problems is
usually called **dp**, to signal to the reader of the code that a **dynamic
programming** solution is what they are observing.

### 3. (Optional) Reconstruct the optimal solution

Dynamic programming identifies the optimal value, but it doesn't necessarily
provide the solution itself. The solution most commonly is the chain of choices
made while calculating the optimal values. These could be characters chosen in the
**Longest Common Subsequence** problem, optimal piece sizes in the **Rod Cutting
Problem** or any other piece of information that describes an optimal solution.

## Acknowledgements

The information on this page heavily relies on chapter *15 Dynamic Programming*
from **Introduction to Algorithms, Third Edition by Thomas H. Cormen, Charles E.
Leiserson, Ronald L. Rivest, Clifford Stein**. For detailed explanations, examples
and the mathematical background please refer to the source.
