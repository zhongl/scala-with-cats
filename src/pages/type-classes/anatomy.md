## Anatomy of a Type Class

Let's now look at how type classes are implemented.
There are three important components to a type class:
the type class itself, which defines an interface,
type class instances, which implement the type class for particular types,
and the methods that use type classes.
The table below shows the language features that correspond to each component.

+---------------------+------------------+
| Type Class Concept  | Language Feature |
+=====================+==================+
| Type class          | trait            |
+---------------------+------------------+
| Type class instance | given instance   |
+---------------------+------------------+
| Type class use      | using clause     |
+---------------------+------------------+

Let's see how this works in detail.


### The Type Class

A type class is an interface or API
that represents some functionality we want implemented.
In Scala a type class is represented by a trait with at least one type parameter.
For example, we can represent generic "serialize to JSON" behaviour
as follows:

```scala mdoc:silent:reset-object
// Define a very simple JSON AST
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json
final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
case object JsNull extends Json

// The "serialize to JSON" behaviour is encoded in this trait
trait JsonWriter[A] {
  def write(value: A): Json
}
```

`JsonWriter` is our type class in this example,
with `Json` and its subtypes providing supporting code.
When we come to implement instances of `JsonWriter`,
the type parameter `A` will be the concrete type of data we are writing.


### Type Class Instances

The instances of a type class
provide implementations of the type class for specific types we care about,
which can include types from the Scala standard library
and types from our domain model.

In Scala we create type class instances by defining
given instances implementing the type class.

```scala mdoc:silent
object JsonWriterInstances {
  given stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json =
        JsString(value)
    }
  
  final case class Person(name: String, email: String)
  
  given JsonWriter[Person] with
    def write(value: Person): Json =
      JsObject(Map(
        "name" -> JsString(value.name),
        "email" -> JsString(value.email)
      ))
  
  // etc...
}
```

In this example we define two type class instances of `JsonWriter`, one for `String` and one for `Person`.
The definition for `String` uses the syntax we saw in the previous section.
The definition for `Person` uses two bits of syntax that are new in Scala 3.
Firstly, writing `given JsonWriter[Person]` creates an anonymous given instance. 
We declare just the type and don't need to name the instance.
This is fine because we don't usually need to refer to given instances by name.
The second bit of syntax is the use of `with` to implement a trait directly without having to 
write out `new JsonWriter[Person]` and so on.
Finally, in a real implementation we'd usually want to define the instances on a companion object: the instance for `String` on the `JsonWriter` companion object (because we cannot define it on the `String` companion object) and the instance for `Person` on the `Person` companion object. 
I haven't done this here because I would need to redeclare `JsonWriter`, as a type and it's companion object must be declared at the same time.



### Type Class Use

A type class use is any functionality 
that requires a type class instance to work.
In Scala this means any method 
that accepts instances of the type class as part of a using clause.

We're going to look at two patterns of type class usage, 
which we call **interface objects** and **interface syntax**.
You'll find these in Cats and other libraries.

#### Interface Objects

The simplest way of creating an interface that uses a type class
is to place methods in a singleton object:

```scala mdoc:silent
object Json {
  def toJson[A](value: A)(using w: JsonWriter[A]): Json =
    w.write(value)
}
```

To use this object, we import any type class instances we care about
and call the relevant method:

```scala mdoc:silent
import JsonWriterInstances.{*, given}
```

```scala mdoc
Json.toJson(Person("Dave", "dave@example.com"))
```

The compiler spots that we've called the `toJson` method
without providing the given instances.
It tries to fix this by searching for given instances
of the relevant types and inserting them at the call site.


#### Interface Syntax

We can alternatively use ***extension methods** to
extend existing types with interface methods[^pimping].
This is sometimes referred to as as **"syntax"** for the type class, 
which is the term used by Cats.
Scala 2 has an equivalent for extension methods kwown as **implicit classes**.

[^pimping]: You may occasionally see extension methods
referred to as "type enrichment" or "pimping".
These are older terms that we don't use anymore.

Here's an example defining an extension method that adds a `toJson` method to
any type for which we have a `JsonWriter` instance.

```scala mdoc:silent
object JsonSyntax {
  extension [A](value: A) {
    def toJson(using w: JsonWriter[A]): Json =
      w.write(value)
  }
}
```

We use interface syntax by importing it
alongside the instances for the types we need:

```scala mdoc:silent
import JsonWriterInstances.given
import JsonSyntax.*
```

```scala mdoc
Person("Dave", "dave@example.com").toJson
```

In Scala 3 we can define extension methods directly on a type class trait.
Since we're defining `toJson` as just calling `write` on `JsonWriter`,
we can instead define `toJson` directly on `JsonWriter` and avoid creating an separate extension method.
(If you're wondering why we did not do this originally, we didn't want to introduce too many concepts at once.) 

```scala mdoc:invisible:reset-object
// Define a very simple JSON AST
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json
final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
case object JsNull extends Json
```

```scala mdoc:silent
trait JsonWriter[A] {
  extension (value: A) def toJson: Json
}
```

```scala mdoc:invisible
object JsonWriterInstances {
  given stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      extension (value: String) 
        def toJson: Json = JsString(value)
    }
  
  final case class Person(name: String, email: String)
  
  given JsonWriter[Person] with
    extension (value: Person) 
      def toJson: Json =
        JsObject(Map(
          "name" -> JsString(value.name),
          "email" -> JsString(value.email)
        ))
  
  // etc...
}
```

Now any type that has a `JsonWriter` method automatically gets a `toJson` extension.

```scala mdoc
import JsonWriterInstances.{*, given}

Person("Dave", "dave@example.com").toJson
```


#### The `summon` Method

The Scala standard library provides
a generic type class interface called `summon`.
Its definition is very simple:

```scala
def summon[A](using value: A): A =
  value
```

We can use `summon` to summon any value in the given scope.
We provide the type we want and `summon` does the rest:

```scala mdoc
import JsonWriterInstances.given

summon[JsonWriter[String]]
```

Most type classes in Cats provide other means to summon instances.
However, `summon` is a good fallback for debugging purposes.
We can insert a call to `summon` within the general flow of our code
to ensure the compiler can find an instance of a type class
and ensure that there are no ambiguity errors.
