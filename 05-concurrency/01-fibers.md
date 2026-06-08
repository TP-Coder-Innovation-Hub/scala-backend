# Fibers `[Mid]`

Fibers are lightweight threads — green threads managed by the runtime, not the OS. You can run hundreds of thousands of fibers concurrently without the memory and context-switching overhead of OS threads.

## Why Fibers Over Threads

An OS thread costs ~1MB of stack memory. 10,000 threads = 10GB of RAM. A fiber costs ~a few KB. 1,000,000 fibers is feasible.

More importantly, fibers are managed by the effect runtime (Cats Effect or ZIO), which schedules them efficiently on a small pool of OS threads (typically matching your CPU cores).

## Cats Effect Fibers

### Basic Concurrency

```scala
import cats.effect.*
import scala.concurrent.duration.*

def fetchApi(name: String): IO[String] =
  IO.println(s"Fetching from $name...") *> 
  IO.sleep(500.millis) *>                  // simulate network delay
  IO.pure(s"response from $name")

// Sequential (slow): ~1.5 seconds
val sequential: IO[List[String]] = for
  r1 <- fetchApi("service-a")
  r2 <- fetchApi("service-b")
  r3 <- fetchApi("service-c")
yield List(r1, r2, r3)

// Concurrent (fast): ~0.5 seconds
val concurrent: IO[List[String]] = for
  results <- IO.parSequence(List(
    fetchApi("service-a"),
    fetchApi("service-b"),
    fetchApi("service-c")
  ))
yield results
```

Step by step:
1. `IO.sleep(500.millis)` — simulates an API call delay
2. `sequential` — runs one at a time. 3 calls * 500ms = 1500ms total
3. `IO.parSequence` — runs all IOs concurrently. 500ms total
4. Results are collected in order regardless of completion order

### Parallel Operations

```scala
import cats.effect.*
import cats.implicits.*

// parMapN — run N operations in parallel, combine results
val parResult: IO[(String, String, String)] = 
  (fetchApi("a"), fetchApi("b"), fetchApi("c")).parMapN { (a, b, c) =>
    (a, b, c)
  }

// parTraverse — map + parallel execution
def fetchUsers(ids: List[Int]): IO[List[User]] =
  ids.parTraverse(id => fetchUser(id))
```

### Start and Join — Manual Fiber Control

```scala
val program: IO[String] = for
  fiber <- fetchApi("service-a").start  // start fiber, get Fiber[IO, String]
  _     <- IO.println("Doing other work while waiting...")
  result <- fiber.join                   // wait for fiber to complete
yield result
```

`.start` launches a fiber in the background. `.join` waits for it. Between start and join, you can do other work.

### Race — First Wins

```scala
// Returns whichever completes first. Cancels the loser.
val fastest: IO[String] = IO.race(
  fetchApi("primary"),
  fetchApi("fallback")
).map(_.merge)  // Either[String, String] => String
```

Use `race` for timeouts or fallback patterns:

```scala
val withTimeout: IO[String] = IO.race(
  fetchApi("slow-service"),
  IO.sleep(5.seconds) *> IO.raiseError(new Exception("Timeout"))
).flatMap {
  case Left(result) => IO.pure(result)
  case Right(_)     => IO.raiseError(new Exception("Service timed out"))
}
```

## ZIO Fibers

```scala
import zio.*
import scala.concurrent.duration.*

def fetchApi(name: String): Task[String] =
  ZIO.debug(s"Fetching from $name...") *>
  ZIO.sleep(500.millis) *>
  ZIO.succeed(s"response from $name")

// Parallel
val concurrent: Task[List[String]] =
  ZIO.foreachPar(List("a", "b", "c"))(fetchApi)

// Race
val fastest: Task[String] = ZIO.race(
  fetchApi("primary"),
  fetchApi("fallback")
)

// Fork and join
val forked: Task[String] = for
  fiber  <- fetchApi("a").fork
  _      <- ZIO.debug("Doing other work...")
  result <- fiber.join
yield result
```

ZIO's fiber operations mirror Cats Effect's: `fork` = `start`, `foreachPar` = `parTraverse`, `race` = `race`.

## Key Principle: Fibers Don't Share Mutable State

Fibers are safe because functional Scala doesn't use mutable shared state. Each fiber operates on immutable data. Communication happens through message passing or effect composition, not locks.

```scala
// This is safe — no shared mutation
val safe = for
  r1 <- fetchApi("a").start
  r2 <- fetchApi("b").start
  v1 <- r1.join
  v2 <- r2.join
yield combine(v1, v2)

// This would be unsafe — but Scala discourages it
// var sharedCounter = 0  // Don't do this
```

Next: [Streaming](02-streaming.md)
