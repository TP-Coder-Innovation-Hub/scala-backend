# http4s — Pure Functional HTTP `[Mid]`

http4s is an HTTP server and client library built on Cats Effect. Routes are functions. Middleware is function composition. Everything is a value.

## Core Concept: Routes as Functions

In http4s, a route is a function from `Request` to `Option[Response]` — more precisely, a `Kleisli[F, Request, Response]`, which is a wrapper around `Request => F[Response]`:

```scala
type HttpRoutes[F[_]] = Kleisli[[A] =>> OptionT[F, A], Request, Response]
// Simplified: Request => F[Option[Response]]
```

A request comes in. Your route either handles it (returns `Some(Response)`) or passes it on (`None`). This is function composition, not a framework callback.

## Setup

```scala
// build.sbt
libraryDependencies ++= Seq(
  "org.http4s" %% "http4s-ember-server" % "0.23.25",
  "org.http4s" %% "http4s-circe"        % "0.23.25",
  "org.http4s" %% "http4s-dsl"          % "0.23.25"
)
```

## CRUD API — Step by Step

### Define the domain

```scala
import io.circe.generic.auto.*
import org.http4s.circe.CirceEntityCodec.*

case class User(id: Long, name: String, email: String)
```

### In-memory store

```scala
import cats.effect.ref.Ref
import cats.effect.IO

class UserRepository(store: Ref[IO, Map[Long, User]]):
  def findAll: IO[List[User]] = store.get.map(_.values.toList)
  def find(id: Long): IO[Option[User]] = store.get.map(_.get(id))
  def save(user: User): IO[Unit] = store.update(_.updated(user.id, user))
  def delete(id: Long): IO[Boolean] = store.modify { m =>
    (m - id, m.contains(id))
  }
```

### Define routes

```scala
import org.http4s.*
import org.http4s.dsl.io.*
import org.http4s.circe.CirceEntityCodec.*
import io.circe.generic.auto.*

def routes(repo: UserRepository): HttpRoutes[IO] = HttpRoutes.of[IO]:

  // GET /users
  case GET -> Root / "users" =>
    for
      users <- repo.findAll
      resp  <- Ok(users)
    yield resp

  // GET /users/:id
  case GET -> Root / "users" / LongVar(id) =>
    for
      user <- repo.find(id)
      resp <- user match
        case Some(u) => Ok(u)
        case None    => NotFound(s"User $id not found")
    yield resp

  // POST /users
  case req @ POST -> Root / "users" =>
    for
      user <- req.as[User]
      _    <- repo.save(user)
      resp <- Created(user)
    yield resp

  // DELETE /users/:id
  case DELETE -> Root / "users" / LongVar(id) =>
    for
      deleted <- repo.delete(id)
      resp    <- if deleted then NoContent() else NotFound(s"User $id not found")
    yield resp
```

Step by step:
1. `HttpRoutes.of[IO]` — pattern match on HTTP method + path
2. `GET -> Root / "users"` — matches GET /users
3. `LongVar(id)` — extracts a Long path variable
4. `req.as[User]` — decodes JSON body to `User` (circe does this, needs `Encoder[User]` and `Decoder[User]`)
5. `Ok(users)`, `Created(user)`, `NotFound(msg)` — helpers that build responses with appropriate status codes

### Wire it all together

```scala
import cats.effect.*
import com.comcast.ip4s.*
import org.http4s.ember.server.EmberServerBuilder

object Main extends IOApp.Simple:

  val run: IO[Unit] = for
    store <- Ref[IO].of(Map.empty[Long, User])
    repo   = UserRepository(store)
    _     <- EmberServerBuilder.default[IO]
      .withHost(host"0.0.0.0")
      .withPort(port"8080")
      .withHttpApp(routes(repo).orNotFound)
      .build
      .useForever
  yield ()
```

`routes(repo).orNotFound` converts `HttpRoutes[F]` (returns `Option[Response]`) into `HttpApp[F]` (returns `Response` — 404 if no route matches).

## Why http4s

- **Testable** — Routes are functions. Pass a `Request`, get a `Response`. No running server needed.
- **Composable** — Middleware wraps routes. Routes combine with `<+>` (try first route, fallback to second).
- **Type-safe** — circe codecs ensure JSON matches your types. Path extractors ensure URL parameters are valid.
- **Effect-safe** — Everything returns `IO`. Side effects are explicit.

Next: [Akka HTTP](02-akka-http.md)
