## The Mechanics of Contextual Abstraction

In section we'll go through Scala's language features that implement contextual abstraction. Once we have a firm understanding of the mechanics of contextual abstraction, we'll move on to their use.

The language features for contextual abstraction have changed from Scala 2 to Scala 3. In the table below I show the Scala 3 and the Scala 2 equivalents of the main features.

+------------------+---------------------+
| Scala 3          | Scala 2             |
+==================+=====================+
| given instance   | implicit value      |
+------------------+---------------------+
| using clause     | implicit parameter  |
+------------------+---------------------+

Let's now explain how they work.


### Using Clauses

We'll start with **using clauses**. A using clause is a method parameter list that starts with the `using` keyword.

```scala mdoc:silent
def double(using x: Int) = x + x
```

The `using` keyword applies to all parameters in the list, so in `add` below both `x` and `y` are in the using clause.

```scala mdoc:silent
def add(using x: Int, y: Int) = x + y
```

We can have normal parameter lists, and multiple using clauses, in the same method.

```scala mdoc:silent
def addAll(x: Int)(using y: Int)(using z: Int): Int =
  x + y + z
```

We cannot pass parameters to a using clause in the normal way. We must proceed the parameters with the `using` keyword as shown below.

```scala mdoc
double(using 1)
add(using 1, 2)
addAll(1)(using 2)(using 3)
```

However this is not the typical way to pass parameters. We usually use given instances instead, so let's turn to them.


### Given Instances

A given instance is a value that is defined with the `given` keyword. Here's a simple example.

```scala mdoc:silent
given theMagicNumber: Int = 3
```

We can use a given instance like a normal value.

```scala mdoc:silent
theMagicNumber * 2
```

However, it's more common to use them with a using clause. When we call a method that has a using clause, and we do not explicitly supply values for the parameters in the using clause, the compiler will look for given instances of the required type.

For example, we defined `double` above with a using clause with an `Int` parameter. The given instance we just defined, `theMagicNumber`, also have type `Int`. So if we call `double` without providing any values for the using clause the compiler will use the value `theMagicNumber` for us.

```scala mdoc
double
```

The same value will be used if there are multiple parameters in a using clause with the same type, as in `add` defined above.

```scala mdoc
add
```

However, if we define multiple given instances of the same type we get an error. The compiler will not arbitrarily choose one for us.

```scala mdoc:reset:invisible
def double(using x: Int) = x + x
given theMagicNumber: Int = 3
```

```scala mdoc:silent
given ohNo: Int = 4
```

```scala
double
// Ambiguous given instances: both given instance theMagicNumber and
// given instance ohNo match type Int of parameter x of method add
```

There are a few more details to given instances. However we'll now turn to type classes, which will give us the opportunity to introduce these additional features in a context where they are useful.
