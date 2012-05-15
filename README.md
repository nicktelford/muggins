Muggins
=======
*An experiment in to better metadata.*

Muggins is an experiment, inspired by [this tweet](https://twitter.com/#!/viktorklang/status/202018613316694016) from Viktor Klang, and the subsequent conversation.

The goal is to investigate code metadata, with the aim of providing more detailed, more useful and more accurate metadata to tooling such as code-generators, documentation generators, compilers and static analysis tools.

Documentation
-------------

Documentation is probably the earliest forms of metadata. Traditionally used to describe a unit of code in a form that's easier for humans to understand.

Modern languages typically use a documentation generator to transform this inline metadata in to documentation, formatted for readability and often cross-referenced and indexed for search. Scala's approach to this is [ScalaDoc](https://wiki.scala-lang.org/display/SW/Scaladoc), which is largely inspired by the well known [JavaDoc](http://www.oracle.com/technetwork/java/javase/documentation/index-jsp-135444.html).

Traditionally, such tools define the documentation using a standard block-comment with some additional formatting rules. The language compilers see the documentation as merely a block-comment and skip it entirely, which leads to some common problems:

  * Code samples are not checked by the compiler/interpreter (see Viktor's Tweet above)
  * 

It would be better if instead, the documentation was compiled and introspectable.

Annotations
-----------

As I noted above, documentation is merely a form of metadata; it describes _what_ the code does and _how_ to use it. However, there are other questions we might like to answer about our code, such as "what's the algorithmic complexity of this function?" or "". What we need is a simple and generic way to define code metadata.

Enter annotations.

Annotations provide two important functions:

  * Static (compile-time) metadata, e.g. [@tailrec](http://www.scala-lang.org/api/current/scala/annotation/tailrec.html)
  * Run-time metadata, e.g. [Jersey/JAX-RS](http://jersey.java.net/nonav/documentation/latest/jax-rs.html) REST bindings

In muggins, we'll look mostly at Static metadata, which is useful for introspection and providing meta-data to static analysis tools and the compiler.

Muggins
-------

Muggins is formed of two parts:

  * annotations - a library of annotations for defining code metadata.
  * docs - a compiler plugin that generates documentation using muggins annotations.

Examples
--------

Here we define an entity as a case class, with annotated documentation:

```scala
@description("a short, public message from a user")
@example[examples.Tweet]
case class Tweet(

  @description("message content")
  text: String, 

  @description("UNIX timestamp the tweet was sent at")
  timestamp: Int, 

  @description("author that published the tweet")
  author: User
)
```

Importantly, the `@example` annotation is providing a reference to a type, in this case in the package `examples`. This ensures that all examples are statically checked by the compiler, _*preventing examples from becoming obsolete or incorrect*_.

We can take advantage of Scala's expressive function literals to provide inline examples, if we so desire:

```scala
trait Collection {

  @description("number of elements in this collection")
  @complexity("O(n)", "n" -> "number of elements in this collection")
  @example({
    def check(c: Collection, size: Int): Boolean = c.size == size
  })
  def size: Int
}
```

You'll notice that we added some metadata describing the algorithmic complexity of this method. This could be used by documentation generators to describe the performance characteristics of the method, or static analysis tools to verify the performance characteristics of the method.

Introspectable
--------------

Because these annotations are introspectable during compilation, the metadata they contain is available for more than simply documentation. In each of the above examples we use the `@description` annotation to describe each code unit. While traditionally used for documentation, this information could be used by anything that would want some generic metadata for a code unit; for example, compile-time errors could provide better labels. The possibilities are endless.

Notation
--------

In my adventures I've noticed that Scala annotations do not have an infix notation, unlike method calls (`obj1 meth obj2`) and type constructors (`[String X Int]`). This strikes me as an inconsistency that I'd love to discuss with someone. I believe that an infix notation for unary annotations may be helpful in improving metadata readability.

```scala
@describe "a short, public message from a user
@example[examples.Tweet]
case class Tweet(

  @description "message content"
  text: String,

  @description "UNIX timestamp the tweet was sent at"
  timestamp: Int,

  @description "author that published the tweet"
  author: User
)
```

This is an issue I plan to raise in scala-debate once I've conducted more research.

Vs. ScalaDoc
------------------------

Muggins is not meant to replace ScalaDoc, although it does experiment with providing all the metadata that would normally be defined with ScalaDoc. The majority of Scala users have experienced the JavaDoc-style of code documentation for long enough that they would find annotations verbose and difficult to read. For this reason, muggins is unlikely to ever be a viable replacement.

But it sure is an interesting experiment!

