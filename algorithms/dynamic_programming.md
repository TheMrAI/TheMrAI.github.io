# Dynamic programming

## Name

The origin of the term dynamic programming is an interesting one. Two legends are
in circulation. One from the author [Richard Bellman](https://en.wikipedia.org/wiki/Richard_E._Bellman) stating that it designed the mathematical background of 
his work from his actual employer. Choosing dynamic as it has no negative 
connotations and to refer to the time warrying, multistage nature of the problems. 
Programming was choosen because it didn't contain the word mathematics and it 
sounded better than planning.
The other legend refers to an attempt to one up the previously existing method, 
named linear programming. 

## Applicability

Dynamic programming is not an algorithm that solves a problem. It is a design
technique that solves a specific set of problems. When properly applied it can
transform algorithms from exponential 𝒪(xⁿ) to polynomial 𝒪(n<sup>x</sup>) time
complexity for a comparably small, but not negligible space requirement of 𝒪(n<sup>x</sup>).

It is most commonly used for optimization problems. In these problems the goal is
to either maximize of minimaze the value function in question. The result is
one of the possiblle optimal solutions in a space where more than one can exist.

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
forming the optimal solution to problem A do not depend on one another i.e. the 
solution for B does note use up resources from C and vice-versa.

> Cut-and-paste: Suppose that each subproblem solution is non-optimal. By cutting
them out and substituting an optimal solutions, demonstrate that you get a better
solution to the original problem, which contradicts your original assumption of
already having an optimal solution to your original problem.

TODO: clarifying image

### Overlapping subproblems


## Design steps

## Top-down / Bottom-up

## Memoization

## Acknowledgements

The information on this page heavily rellies on chapter *15 Dynamic Programming*
from **Introduction to Algorithms, Third Edition by Thomas H. Cormen, Charles E.
Leiserson, Ronald L. Rivest, Clifford Stein**. For detailed explanations, examples
and the mathematical background please refer to the source.