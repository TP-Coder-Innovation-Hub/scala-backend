# What Is Scala? `[Entry]`

Scala was created by **Martin Odersky** at EPFL (Swiss Federal Institute of Technology) and released in **2004**. Odersky previously co-designed Java generics and wrote the reference `javac` compiler. He created Scala because he believed Java was too verbose and inflexible for the problems he wanted to solve.

## Scala = SCAlable LAnguage

The name reflects the design goal: a language that scales from small scripts to large, multi-team systems. Scala achieves this by combining two paradigms:

1. **Functional Programming** — Pure functions, immutable data, type classes, pattern matching, algebraic data types
2. **Object-Oriented Programming** — Classes, traits (interfaces), inheritance, encapsulation

Every value is an object. Every function is a value. There is no artificial separation between FP and OO — they work together.

## Strong Type System

Scala's type system is its defining feature for backend work:

- **Static typing** — Types checked at compile time, not runtime
- **Type inference** — The compiler figures out types so you don't annotate everything
- **Algebraic Data Types (ADTs)** — `sealed trait` + `case class` let you model domains where the compiler checks you handle every case
- **Type classes** — Extend types without modifying them (more in [06-implicits-and-givens](../01-first-code/06-implicits-and-givens.md))
- **Higher-kinded types** — Abstract over type constructors (`F[_]`), enabling libraries like Cats and http4s
- **Opaque types** (Scala 3) — Zero-cost type wrappers that prevent mixing up domain concepts

```scala
// The compiler ensures you handle every case
sealed trait PaymentStatus
case class Paid(amount: BigDecimal) extends PaymentStatus
case object Pending extends PaymentStatus
case class Failed(reason: String) extends PaymentStatus

def handle(status: PaymentStatus): String = status match
  case Paid(amount)    => s"Received $amount"
  case Pending         => "Waiting for payment"
  case Failed(reason)  => s"Payment failed: $reason"
  // If you forget a case, the compiler warns you
```

## Scala 2 vs Scala 3

- **Scala 2** (2006–2021): Mature ecosystem, most existing libraries, `implicit` keyword
- **Scala 3** (2021–present): New syntax, `given`/`using` instead of `implicit`, union/intersection types, optional braces, better error messages. The future of the language.

This roadmap uses Scala 3 syntax but notes Scala 2 differences where relevant.

## Why Scala for Backend

The backend value proposition:

1. **Correctness at compile time** — The type system catches bugs before deployment
2. **Expressiveness** — Write in 10 lines what takes 50 in Java
3. **Concurrency model** — Effect systems (Cats Effect, ZIO) provide safe, composable concurrency
4. **JVM reliability** — Decades of production-hardened runtime, garbage collection, monitoring tools
5. **Ecosystem** — Access to all Java libraries plus a rich Scala-native ecosystem

Next: [Why Scala Backend](06-why-scala-backend.md)
