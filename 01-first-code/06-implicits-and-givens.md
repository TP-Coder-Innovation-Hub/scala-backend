# Implicits and Givens ``

Implicits (Scala 2) and given/using (Scala 3) are Scala's mechanism for type-class pattern and dependency injection. They are the plumbing behind Cats, http4s, circe, and most of the Scala ecosystem.

## The Problem They Solve

Consider JSON serialization. You want `User` to serialize to JSON, but you don't want to add a `toJson` method to the `User` class (that couples data to serialization).

The type-class pattern: define serialization externally.

```scala
trait JsonEncoder[A]:
  def encode(value: A): String

case class User(id: Int, name: String)
```

How do you provide a `JsonEncoder[User]` without modifying `User`? Implicits/givens.

## Scala 2: Implicit Parameters

```scala
// Define an instance
implicit val userEncoder: JsonEncoder[User] = new JsonEncoder[User]:
  def encode(user: User): String = s"""{"id":${user.id},"name":"${user.name}"}"""

// Use it as an implicit parameter
def toJson[A](value: A)(implicit encoder: JsonEncoder[A]): String =
  encoder.encode(value)

toJson(User(1, "Alice"))  // Compiler finds userEncoder automatically
```

The compiler searches for an `implicit JsonEncoder[User]` in scope, finds `userEncoder`, and passes it. You never specify it at the call site.

## Scala 3: given and using

Scala 3 replaces `implicit` with clearer syntax:

```scala
// Define a given instance
given JsonEncoder[User] with
  def encode(user: User): String = s"""{"id":${user.id},"name":"${user.name}"}"""

// Use it with `using`
def toJson[A](value: A)(using encoder: JsonEncoder[A]): String =
  encoder.encode(value)

toJson(User(1, "Alice"))  // Compiler finds the given automatically
```

`given` defines the instance. `using` declares the dependency. The semantics are the same — the compiler finds and provides the instance.

## Why This Matters

This pattern powers the entire Scala ecosystem:

- **circe** — `Encoder[User]` and `Decoder[User]` are type-class instances. The compiler finds them when you call `.asJson` or `.as[User]`.
- **Cats** — `Functor[F]`, `Monad[F]`, `Monoid[A]` are type classes. Given instances provide behavior for specific types.
- **http4s** — `EntityDecoder[F, User]` tells http4s how to decode an HTTP entity into a `User`. Provided as a given.
- **Doobie** — `Get[A]` and `Put[A]` tell Doobie how to read/write database columns. Given instances for basic types, you write them for custom types.

## Context Bounds — Shorthand

When you only need the implicit/given for calling methods, use a context bound:

```scala
// Scala 2
def printJson[A: JsonEncoder](value: A): String =
  implicitly[JsonEncoder[A]].encode(value)

// Scala 3
def printJson[A: JsonEncoder](value: A): String =
  summon[JsonEncoder[A]].encode(value)
```

`A: JsonEncoder` means "A has a JsonEncoder instance available." `summon` retrieves it.

## Extension Methods

Scala 3 also uses givens for extension methods:

```scala
extension (user: User)
  def toJson(using enc: JsonEncoder[User]): String = enc.encode(user)

User(1, "Alice").toJson  // Looks like a method on User, but it's external
```

This is how libraries like circe add `.asJson` to any type without modifying it.

## Don't Fear Implicits

Implicits have a reputation for being confusing. In practice, you mostly consume them (library provides given instances, you use them), not define them. The key insight: they are dependency injection at the type level, resolved at compile time, not runtime.

Next: [Effect Systems](../02-effect-systems/01-why-effects.md)
