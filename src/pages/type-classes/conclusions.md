## Conclusions

In this chapter we took a first look at type classes.
We saw the components that make up a type class:

- A `trait`, which is the type class

- Type class instances, which are given instances.

- Type class usage, which uses using clauses.

We saw that type classes can be composed from components using type class composition.
This is one form of metaprogramming in Scala, 
where we can get the compiler to do work for us based on our program's types.

We can view type classes as marrying codata with tools to select and compose implementations based on type. 
Additionally, we can view type classes as shifting implementation from the definition site to the call site.


Type classes were introduced in [@10.1007/3-540-19027-9_9] and [@10.1145/75277.75283]. The concept of implicit parameters didn't come until later, in [@10.1145/325694.325708]. The original motivation for implicit parameters was to decompose type classes into smaller orthogonal language features, but they were shown to be useful for several other tasks. Scala's implementation differs in several ways from the original presentation, but they similarly have proven useful for solving problems beyond type classes. Our focus here

Type classes in Scala [@10.1145/1932682.1869489]

[Implicit parameters: dynamic scoping with static types][implicits] introduces implicit parameters, which provided inspiration for Scala 2's implicit parameters and evolved into Scala 3's given instances and using clauses. It's interesting to see the differences between implicit parameters as proposed here and how they are implemented in Scala.



[implicits]: https://dl.acm.org/doi/abs/10.1145/325694.325708
