## Conclusions

In this chapter we took a first look at type classes.
We saw the components that make up a type class:

- A `trait`, which is the type class

- Type class instances, which are given instances.

- Type class usage, which uses using clauses.

We saw that type classes can be composed from components using type class composition.
This is one form of metaprogramming in Scala, 
where we can get the compiler to do work for us based on our program's types.

We can view type classes are marrying codata with tools to select and compose implementations based on type. 
Additionally, we can view type classes as shifting implementation from the definition site to the call site.


[Implicit parameters: dynamic scoping with static types][implicits] introduces implicit parameters, which provided inspiration for Scala 2's implicit parameters and evolved into Scala 3's given instances and using clauses. It's interesting to see the differences between implicit parameters as proposed here and how they are implemented in Scala.

[@10.1145/325694.325708]

[implicits]: https://dl.acm.org/doi/abs/10.1145/325694.325708
