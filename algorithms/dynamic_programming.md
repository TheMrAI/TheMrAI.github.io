# Dynamic programming

## Name

The origin of the term dynamic programming is an interesting one. Two legends are
in circulation. One from the author [Richard Bellman](https://en.wikipedia.org/wiki/Richard_E._Bellman) 
stating that it designed the mathematical background of
his work from his actual employer. Choosing dynamic as it has no negative
connotations and to refer to the time varying, multistage nature of the problems.
Programming was chosen because it didn't contain the word mathematics and it
sounded better than planning.
The other legend refers to an attempt to one up the previously existing method,
named linear programming.

## Applicability

Dynamic programming is not an algorithm that solves a problem. It is a design
technique that solves a specific set of problems. When properly applied it can
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
<u>optimal solutions</u> to it's subproblems then we say the problem exhibits an
optimal substructure.

There are 3 steps used while testing for optimal substructure:

1. Show that the solution to the problem consists of making a choice after which
the problem space separates into one or more independent subproblems, where each
exhibits the same properties as the original problem.
2. Assume that for a given problem there exists an optimal choice.
3. Show using the "cut-and-paste" method that given an optimal solution to a
problem the resulting subproblem solutions have to be optimal as well.

> Independent optimal solutions: The optimal solutions to the subproblems B and C
forming the optimal solution to problem A do not use up resources from on one
another i.e. the solution for B does note use solutions from C and vice-versa.

> Cut-and-paste: Suppose that each subproblem solution is non-optimal. By cutting
them out and substituting an optimal solutions, demonstrate that you get a better
solution to the original problem, which contradicts your original assumption of
already having an optimal solution to your original problem.

TODO: clarifying image

### Overlapping subproblems

Subproblems are considered overlapping when the solution for them share a common
set of subproblems among them.

The implication of this requirement is that the set of distinct subproblems
is small compared to the total set of subproblems that would need to be visited
while applying a brute force approach in solving the problem.

TODO: clarifying image, a tree of problems marked and color coded, to show that the
same problems appear again and again

## Design steps

Developing an algorithm using dynamic programming is not a simple task. Especially
as dynamic programming itself isn't a way of designing per say, more of a way of
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
subproblems having the same basic format on a lower level. Following this
approach the above example can have the below recursive implementation:

```text
RECURSIVE_SOLUTION(X):
if (trivial_state_has_been_reached):
    return appropriate_value
result = 0
for each_possible_choice_while_dividing_X_into_subproblems_1_2_...:
    result = max_or_min(result, RECURSIVE_SOLUTION(subproblem_1) 
    + RECURSIVE_SOLUTION(...) + constant)
return result
```

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
historically this step was called memoization, based on the latin word [memorandum](https://en.wikipedia.org/wiki/Memoization#Etymology).

With memoization the above algorithm would look something similar to:

```text
TOP_DOWN_SOLUTION(X, memo):
if (trivial_state_has_been_reached):
    return appropriate_value
result = 0
for each_possible_choice_while_dividing_X_into_subproblems_1_2_...:
    if (memo has value for subproblem_1):
        subproblem_1_value = memo[subproblem_1]
    else:
        subproblem_1_value = RECURSIVE_SOLUTION(subproblem_1, memo)
        memo[subproblem_1] = subproblem_1_value
    if (memo has value for subproblem_...):
        subproblem_..._value = memo[subproblem_...]
    else:
        subproblem_..._value = RECURSIVE_SOLUTION(subproblem_..., memo)
        memo[subproblem_...] = subproblem_..._value
    result = max_or_min(result, subproblem_1_value + subproblem_2_value +
    + subproblem_..._value + constant)
    memo[X] = result
return result
```

The algorithm stays the same, except that now previously calculated values are
looked up prior to re-calculating them.

#### Bottom-up

The bottom-up approach is a bit different as the procedure has to be turned around.
Requires much more insight into how the algorithm is structured and what data has
to be calculated exactly when. Which sub-problems are dependencies to sub-problems
above them. With these questions answered one has to transform the algorithm such
that each sub-problem result is only calculated once and stored into an
appropriate container. By accessing the finally calculated value in this container
we receive the solution to the problem.

It is hard to convey how this would look like on a general example as the one
above, but the following could be used as an approximation:

```text
BOTTOM_UP_SOLUTION(X):
let dp[1..n, 1..m]
// fill up dp[x][y] with values for the trivial problems
for i = 1 to n:
    dp[i][1] = 0
for j = 1 to m:
    dp[1][m] = 0
// fill up dp systematically
for i = 2 to n:
    for j = 2 to m:
        dp[n][m] = min_or_max(dp[n][m], dp[n-1][m] + dp[n][m-1] + constant)
return dp[n][m]
```

Above we see the top-down approach where the **memo** map has been changed to be
a matrix called **dp**. Then instead of recursively defining the values from the
top down, they get iteratively calculated from the bottom up. Each value gets
calculated when all necessary previous values are available.

The nature of the container isn't important. What should be a noticeable difference
between the top-down and bottom-up implementation is that now there is no more
recursion. The value is calculated in an iterative manner by solving the problems
in the direction from the smallest sub-problem towards the original problem.

> In implementations the container used to store the values is usually called
**dp**, to signal to the reader of the code that a **dynamic programming** solution
is what they are observing.

### 3. (Optional) Reconstruct the optimal solution

## Acknowledgements

The information on this page heavily relies on chapter *15 Dynamic Programming*
from **Introduction to Algorithms, Third Edition by Thomas H. Cormen, Charles E.
Leiserson, Ronald L. Rivest, Clifford Stein**. For detailed explanations, examples
and the mathematical background please refer to the source.
