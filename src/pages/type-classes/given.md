## The Mechanics of Contextual Abstraction

In section we'll go through Scala's language features that implement contextual abstraction. Once we have a firm understanding of the mechanics of contextual abstraction, we'll move on to their use.

The language features for contextual abstraction have changed their names from Scala 2 to Scala 3. In the table below I show the Scala 3 features, and their Scala 2 equivalents. If you use Scala 2 you'll find that you can largely translate the code to Scala 2 just by changing the names as appropriate.

+------------------+---------------------+
| Scala 3          | Scala 2             |
+==================+=====================+
| given instance   | implicit value      |
+------------------+---------------------+
| using clause     | implicit parameter  |
+------------------+---------------------+

Let's now explain how these language features work.


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

However, it's more common to use them with a using clause. When we call a method that has a using clause, and we do not explicitly supply values for the parameters in the using clause, the compiler will look for given instances of the required type. If it finds a given instance it will automatically use it to complete the method call.

For example, we defined `double` above with a using clause with an `Int` parameter. The given instance we just defined, `theMagicNumber`, also has type `Int`. So if we call `double` without providing any values for the using clause the compiler will provide the value `theMagicNumber` for us.

```scala mdoc
double
```

The same given instance will be used for multiple parameters in a using clause with the same type, as in `add` defined above.

```scala mdoc
add
```


### Given Scope and Imports

Given instances are usually not explicitly passed to using clauses.
Their whole reason for existence is to get the compiler to do some work for us.
To ensure our code can be understood, we need to be very clear about which given instances are candidates to be supplied to a using clauses.
The **given scope** is all the places that the compiler will look for given instances.

The first rule we should know about the given scope is that it starts at the **call site**, where the method with a using clause is called, not at the **definition site** where the method is defined.
This means the following code does not compile, because the given instance is not in scope at the call site, even though it is in scope at the definition site.

```scala mdoc:reset:fail
object A {
  given a: Int = 1
  def whichInt(using int: Int): Int = int
}

A.whichInt
```

The second rule, which we have been relying on in all our examples so far, is that the given scope includes the **lexical scope** at the call site.
This is the scope where values are usually found.
This means the following code works, as `a` is defined in a scope that includes the call site.

```scala mdoc:silent
object A {
  given a: Int = 1
  
  object B {
    C.whichInt 
  }
  
  object C {
    def whichInt(using int: Int): Int = int
  }
}
```

However, if there are multiple given instances in the same scope the compiler will not arbitrarily choose one.
Instead it fails with an error telling us the choice is ambiguous.

```scala
object A {
  given a: Int = 1
  given b: Int = 2
    
  def whichInt(using int: Int): Int = int
    
  whichInt
}
// error:
// Ambiguous given instances: both given instance a in object A and
// given instance b in object A match type Int of parameter int of 
// method whichInt in object A
```

We can import given instances from other scopes, just like we can import normal values, but we must explicitly say we want to import given instances. The following code does not work because we have not explicitly imported the given instances.

```scala mdoc:reset:fail
object A {
  given a: Int = 1

  def whichInt(using int: Int): Int = int
}
object B {
  import A.*
    
  whichInt
}
```

It works when we do explicitly import them using `import A.given`.

```scala mdoc:reset:silent
object A {
  given a: Int = 1

  def whichInt(using int: Int): Int = int
}
object B {
  import A.{given, *}
    
  whichInt
}
```

One final wrinkle. The given scope includes the companion objects of any type involved in the type of the using clause. 
This is best illustrated with an example.
We'll start by defining a type `Sound` that represents the sound made by its type variable `A`, and a method `soundOf` to access that sound.

```scala mdoc:reset
trait Sound[A] {
  def sound: String
}

def soundOf[A](using s: Sound[A]): String =
  s.sound
```

Now we'll define some given instances. Notice that they are defined on the relevant companion objects.

```scala mdoc:silent
trait Cat
object Cat {
  given catSound: Sound[Cat] =
    new Sound[Cat]{
      def sound: String = "meow"
    }
}

trait Dog
object Dog {
  given dogSound: Sound[Dog] = 
    new Sound[Dog]{
      def sound: String = "woof"
    }
}
```

When we call `soundOf` we don't have to bring the instances into scope. They are automatically in the given scope by virtue of being defined on the companion objects of the types we use (`Cat` and `Dog`). If we had defined these instances on the `Sound` companion object they would also be in the given scope.

```scala mdoc
soundOf[Cat]
soundOf[Dog]
```

We should almost always be defining given instances on companion objects. This simple organization scheme means that users do not have to explicitly import them but can easily find them if they wish to inspect the implementation.


#### Given Instance Priority

Notice that given instance selection is based entirely on types. We don't even pass any values to `soundOf`! This means given instances are easiest to use when there is only one instance for each type. In this case we can just put the instances on a relevant companion object and everything works out.

However, this is not always possible (though it's often an indication of a bad design if it is not). For cases where we need multiple instances for a type, we can use the instance priority rules to select between them. We'll look at the three most important rules below.

The first rule is that explicitly passing an instance takes priority over everything else.

```scala mdoc:reset:silent
given a: Int = 1
def whichInt(using int: Int): Int = int
```
```scala mdoc
whichInt(using 2)
```
   
The second rule is that instances in the lexical scope take priority over instances in a companion object

```scala mdoc:reset:silent
trait Sound[A] {
  def sound: String
}
trait Cat
object Cat {
  given catSound: Sound[Cat] =
    new Sound[Cat]{
      def sound: String = "meow"
    }
}

def soundOf[A](using s: Sound[A]): String =
  s.sound
```
```scala mdoc
given purr: Sound[Cat]  =
  new Sound[Cat]{
    def sound: String = "purr"
  }

soundOf[Cat]
```
   
The final rule is that instances in a closer lexical scope take preference over those further away.

```scala mdoc
{
  given growl: Sound[Cat] =
   new Sound[Cat]{
     def sound: String = "growl"
   }
   
  {
    given mew: Sound[Cat] =
     new Sound[Cat]{
       def sound: String = "mew"
     }
     
    soundOf[Cat]
  }
}
```
   
There are a few more details to given instances. 


### The Role of Contextual Abstraction



However we'll now turn to type classes, which will give us the opportunity to introduce these additional features in a context where they are useful.