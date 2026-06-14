# Scala Backend Roadmap

> A focused, opinionated path from zero to production-ready Scala backend developer.
> Built by [TP-Coder-Innovation-Hub](https://github.com/TP-Coder-Innovation-Hub).

This roadmap teaches **type-safe, functional backend services** in Scala. Not data engineering. Not Spark. Not Play framework. We build HTTP APIs with **http4s / Akka HTTP**, manage effects with **Cats Effect / ZIO**, and talk to databases with **Doobie / Slick**.

The core thesis: **types catch your bugs at compile time**. Scala's type system is powerful enough to encode business rules, error paths, and effect boundaries so the compiler enforces correctness before your code ever runs.

---

## Objectives

By the end of this roadmap you will be able to:

- Write Scala programs using functional programming principles (purity, immutability, composition)
- Use effect systems (Cats Effect, ZIO) to build composable, testable, side-effect-safe programs
- Build HTTP APIs with http4s or Akka HTTP, with typed JSON codecs
- Interact with databases using Doobie or Slick
- Handle concurrency with fibers and streaming (fs2)
- Test, observe, and deploy Scala backend services

---

## Roadmap

| # | Module | Focus |
|---|--------|-------|
| 00 | [Foundations](00-foundations/) | Programming concepts, Scala history, why Scala for backend |
| 01 | [First Code](01-first-code/) | Setup, syntax, types, functions, case classes, implicits |
| 02 | [Effect Systems](02-effect-systems/) | **The core.** Why effects matter, Cats Effect, ZIO, typed errors |
| 03 | [HTTP APIs](03-http-apis/) | http4s, Akka HTTP, JSON codecs, middleware |
| 04 | [Database](04-database/) | Doobie, Slick, migrations |
| 05 | [Concurrency](05-concurrency/) | Fibers, fs2 streaming, supervision |
| 06 | [Production](06-production/) | Testing, observability, deployment |
| 07 | [Workshop](07-workshop/) | Hands-on project (coming soon) |

---

## How to Use This Roadmap

1. Go in order. Each module builds on the last.
2. Type every code example. Reading is not doing.
3. Module 02 (Effect Systems) is the conceptual core. Re-read it if needed.
4. Use [AGENTS.md](AGENTS.md) if you're working with AI assistants on this material.

## Stack

- **Language:** Scala 3 (with Scala 2 notes where relevant)
- **Build:** sbt or Scala CLI
- **Effects:** Cats Effect 3 or ZIO 2
- **HTTP:** http4s or Akka HTTP
- **Database:** Doobie or Slick
- **Streaming:** fs2
- **Testing:** munit, weaver
- **JSON:** circe

## Level Badges

- `` — No prior Scala experience needed
- `` — Comfortable with basics, ready for real patterns
- `` — Production-grade thinking, architecture decisions
