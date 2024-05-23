## What Type Classes Are

We've have now seen the mechanics of type classes: they are a specific arrangement of trait, given instances, and using clauses. This is a very craft-level explanation. Let's now raise the level of the explanation which two different views of type classes.

The first view goes back Chapter [@sec:codata], where we looked at codata. The type class itself---the trait---is an instance codata and it has the usual advantages of codata (we can easily add implementations) and disadvantages (we cannot easily changes the interface). Given instances and using clauses add the ability to chose the codata implementation based on the type of the context parameter and the instances in the given scope, and to compose instances from smaller components.

Raising the level of abstraction again, we can say that type classes allow us to implement functionality (the type class instance) separately from the type to which it applies, so that the implementation only needs to be defined at the point of the use---the call site---not at the point of declaration.
