# Supervision ``

Fibers fail. Network calls timeout. Database connections drop. Supervision is how you manage fiber lifecycles — what happens when things go wrong, and how to clean up.

> 🖼️ **[IMAGE_PLACEHOLDER]** — fiber supervision structured concurrency all or nothing cancellation

## The Problem

When a fiber fails, resources leak if not cleaned up:

```scala
val leaky = for
  conn <- IO(database.getConnection())  // acquired resource
  result <- IO(conn.query("SELECT ...")) // what if this throws?
  _ <- IO(conn.close())                  // never reached if query fails
yield result
```

The connection is never closed. In production, this exhausts your connection pool.

## Cats Effect: Resource and Fiber Supervision

### Resource — Guaranteed Cleanup

`Resource` ensures cleanup runs regardless of success or failure:

```scala
import cats.effect.{IO, Resource}

def withConnection[A](use: Connection => IO[A]): IO[A] =
  Resource.make(IO(database.getConnection()))(conn => IO(conn.close()))
    .use(use)
```

`Resource.make(acquire)(release).use(f)` guarantees: acquire, then run f, then release — even if f fails.

### Supervised Fibers

When you start multiple fibers, one failing shouldn't leave others running:

```scala
import cats.effect.{IO, IOApp, Outcome}

val supervised = for
  fiber1 <- fetchApi("service-a").start
  fiber2 <- fetchApi("service-b").start
  fiber3 <- fetchApi("service-c").start
  // If fiber2 fails, fiber1 and fiber3 keep running — leaked!
yield ()
```

Use `IO.race` or `IO.parSequence` for structured concurrency:

```scala
// All must succeed, or all are canceled
val allOrNothing: IO[List[String]] = 
  IO.parSequence(List(
    fetchApi("service-a"),
    fetchApi("service-b"),
    fetchApi("service-c")
  ))
```

If any one fails, all others are canceled. No leaked fibers.

### Supervisor Special

For long-running background tasks that should restart on failure:

```scala
import cats.effect.{IO, Supervisor}

val program = Supervisor[IO].use { supervisor =>
  // Background task that auto-restarts on failure
  def backgroundFetch: IO[Unit] =
    fetchApi("health-check").handleErrorWith { err =>
      IO.println(s"Health check failed: $err") *> 
      IO.sleep(5.seconds) *> 
      backgroundFetch  // retry
    }

  for
    _ <- supervisor.supervise(backgroundFetch)  // runs in background
    _ <- mainApplication                        // foreground work
  yield ()
}
```

`Supervisor` manages background fibers. When the `Supervisor` scope ends, all supervised fibers are canceled.

## ZIO: Supervision Built In

ZIO has supervision as a first-class concept:

```scala
import zio.*

// ZIO tracks all child fibers automatically
val program = for
  _ <- fetchApi("a").fork  // forked fiber is tracked
  _ <- fetchApi("b").fork  // tracked
  _ <- fetchApi("c").fork  // tracked
yield ()
// When `program` completes, ZIO can supervise/cancel all forked fibers

// Supervise with a policy
val supervised = program.supervised(Supervisor.fromFibers { fibers =>
  ZIO.debug(s"Active fibers: ${fibers.size}")
})
```

## Graceful Shutdown

When your service receives a shutdown signal (SIGTERM in Kubernetes), you need to:

1. Stop accepting new requests
2. Finish in-flight requests
3. Release resources (database connections, file handles)
4. Exit

```scala
import cats.effect.{IO, IOApp, ExitCode}
import cats.effect.std.Dispatcher
import com.comcast.ip4s.*
import org.http4s.ember.server.EmberServerBuilder

object Main extends IOApp:

  def run(args: List[String]): IO[ExitCode] =
    transactor[IO].use { xa =>
      EmberServerBuilder.default[IO]
        .withHost(host"0.0.0.0")
        .withPort(port"8080")
        .withHttpApp(routes(xa).orNotFound)
        .withShutdownTimeout(30.seconds)  // give in-flight requests 30s
        .build
        .use { server =>
          IO.println(s"Server running on ${server.address}") *>
          IO.never  // block forever until interrupted
        }
    }.as(ExitCode.Success)
```

Step by step:
1. `transactor.use` — acquires database pool, releases on shutdown
2. `.build.use` — starts HTTP server, stops on shutdown
3. `IO.never` — runs until the process receives SIGTERM
4. On shutdown: server stops accepting requests, waits up to 30s for in-flight requests, then transactor releases the pool

## Supervision Rules

1. **Use `Resource` for every scarce resource** — database connections, file handles, HTTP clients. Never acquire without a release plan.

2. **Use structured concurrency** — `parSequence`, `parTraverse`, `race`. Avoid `.start` unless you have a supervision plan.

3. **Handle errors at boundaries** — Don't let errors propagate to the top. Handle them where you have context.

4. **Test shutdown behavior** — Start your app, send SIGTERM, verify resources are released.

5. **Never leak fibers** — Every `.start` should have a corresponding `.join` or cancellation.

Next: [Production](../06-production/01-testing.md)
