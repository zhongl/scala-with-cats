# Contextual Abstraction {#sec:type-classes}

All but the simplest programs depends on the **context** in which they run. For example, the platform on which the code runs determines available resources, which in turn may determine how a program works. For example, it might change how work is scheduled depending on the availability of processor cores, or use different rendering algorithms depending on the capabilities of the available graphics hardware. Other forms of context can include configuration read from files or environment variables, or (and we'll see at lot of this later) values created at compile-time, such as serialization formats, in response to the type of some method parameters.

Scala is one of the few languages that provides features for **contextual abstraction**, known as **implicits** in Scala 2 or **given values** in Scala 3. In Scala these features are intimately related to types; types are used to select between different available given values and drive construction of given values at compile-time.

Most programmers are less confident with the features for contextual abstraction than with other parts of the language, so this chapter will start by reviewing the abstractions formely known as implicits: given values, using clauses, and extension methods. We will then look at one of their major uses, **type classes**[^type-class-defn]. Type classes are the core of Cats, which we will be exploring in the next part of this book.

Type classes are a programming pattern originating in Haskell.
They allow us to extend existing libraries with new functionality,
without using traditional inheritance,
and without altering the original library source code.

<!--
Type classes work well with another programming pattern: *algebraic data types*.
These are closed systems of types that we use to represent data or concepts.
Because the systems are closed (and therefore cannot be extended by other users),
we can process them using pattern matching
and the compiler will check the exhaustiveness of our case clauses.

There are two other patterns we need to cover in this chapter.
*Value classes* provide a way to wrap up
generic data types like `Strings` and `Ints`
and give them specific meanings in a given context.
The extra type information is useful when type classes.
*Type aliases* are another pattern that
provide aliases for large, complex types.
-->

In this chapter we will refresh our memory of type classes
from Underscore's [Essential Scala][link-essential-scala] book,
and take a first look at the Cats codebase.
We will look at two example type classes---`Show` and `Eq`---using
them to identify patterns that lay the foundations for the rest of the book.

We'll finish by tying type classes back into algebraic data types,
pattern matching, value classes, and type aliases,
presenting a structured approach to functional programming in Scala.

[^type-class-defn]: The word "class" doesn't strictly mean `class` in the Scala or Java sense.
