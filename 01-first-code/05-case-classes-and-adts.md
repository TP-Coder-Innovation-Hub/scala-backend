# Case Classes and ADTs ``

Case classes and sealed traits are the workhorse of Scala backend. Together they form Algebraic Data Types (ADTs) — the mechanism for modeling your domain so the compiler enforces correctness.

## Case Classes — Immutable Data

```scala
case class User(id: Int, name: String, email: String)
```

A `case class` gives you:

- **Immutability** — Fields are `val` by default. You cannot modify them.
- **Equality** — Two case classes with the same fields are equal: `User(1, "Alice", "a@a.com") == User(1, "Alice", "a@a.com")` is `true`.
- **Copying** — `user.copy(name = "Bob")` creates a new instance with one field changed.
- **Pattern matching** — You can destructure case classes in `match`.
- **Serialization** — Automatic `toString`, `hashCode`, and support for JSON libraries.

```scala
val user = User(1, "Alice", "alice@example.com")

user.copy(email = "new@example.com")  // New instance, original unchanged
user.name                              // "Alice"
user match
  case User(_, name, _) => println(s"Found $name")
```

Case classes are how you represent domain data in Scala backend. A user, an order, a payment, a configuration — all case classes.

## Sealed Traits — Sum Types

A `sealed trait` can only be extended in the same file. The compiler knows every subtype:

```scala
sealed trait PaymentMethod
case class CreditCard(number: String, expiry: String) extends PaymentMethod
case class BankTransfer(account: String, routing: String) extends PaymentMethod
case object Cash extends PaymentMethod
```

`PaymentMethod` is a sum type — a value is one of CreditCard, BankTransfer, or Cash. Nothing else. The compiler knows this and checks exhaustiveness:

```scala
def process(method: PaymentMethod): String = method match
  case CreditCard(number, _)  => s"Charge card $number"
  case BankTransfer(acc, _)   => s"Transfer to $acc"
  case Cash                   => "Cash payment"
  // Forget a case? Compiler warning: "match may not be exhaustive"
```

## ADTs = Case Classes + Sealed Traits

An Algebraic Data Type combines:

- **Product types** (case classes) — "this AND that" — `User` has id AND name AND email
- **Sum types** (sealed traits) — "this OR that" — `PaymentMethod` is CreditCard OR BankTransfer OR Cash

Together, they let you model domains precisely:

```scala
sealed trait OrderStatus
case object Draft extends OrderStatus
case class Submitted(at: java.time.Instant) extends OrderStatus
case class Paid(transactionId: String, at: java.time.Instant) extends OrderStatus
case class Shipped(trackingNumber: String) extends OrderStatus
case class Cancelled(reason: String, at: java.time.Instant) extends OrderStatus
```

Every possible state is encoded. The compiler ensures you handle every state. If someone adds `Returned`, the compiler flags every match that doesn't handle it.

## Why ADTs Are Central to Scala Backend

1. **Domain modeling** — Your business rules become types. Invalid states become unrepresentable.
2. **Exhaustiveness** — The compiler forces you to handle every case. No forgotten error paths.
3. **Refactoring safety** — Add a new case, and the compiler lists every place that needs updating.
4. **Serialization** — Case classes serialize to JSON/databases automatically with circe/Doobie.
5. **Pattern matching** — Clean, readable logic that handles each case explicitly.

```scala
// Invalid state is unrepresentable
case class Order(
  id: Int,
  items: List[Item],
  status: OrderStatus,
  createdAt: java.time.Instant
)
```

You cannot create an order without a status. You cannot have a status that doesn't exist. The type system enforces your business rules.

Next: [Implicits and Givens](06-implicits-and-givens.md)
