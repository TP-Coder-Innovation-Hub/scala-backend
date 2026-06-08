# Testing `[Mid]`

Testing functional Scala is different from testing imperative code. Pure functions are trivially testable. Effectful code is tested by asserting on IO values. No mocking framework needed — just pass different inputs.

## munit — The Test Framework

munit is a lightweight test framework with first-class Cats Effect support:

```scala
libraryDependencies += "org.scalameta" %% "munit" % "1.0.0" % Test
```

### Basic Tests

```scala
import munit.FunSuite

class MathSuite extends FunSuite:
  test("addition works"):
    assertEquals(1 + 1, 2)

  test("list map transforms elements"):
    assertEquals(List(1, 2, 3).map(_ * 2), List(2, 4, 6))
```

### Testing IO Values — Cats Effect

```scala
libraryDependencies += "org.typelevel" %% "munit-cats-effect" % "2.0.0" % Test
```

```scala
import munit.CatsEffectSuite
import cats.effect.IO

class UserRepoSuite extends CatsEffectSuite:

  test("save and find a user"):
    for
      store <- IO.ref(Map.empty[Long, User])
      repo   = new UserRepository(store)
      user   = User(1, "Alice", "alice@example.com")
      _     <- repo.save(user)
      found <- repo.find(1)
    yield assertEquals(found, Some(user))

  test("find returns None for missing user"):
    for
      store <- IO.ref(Map.empty[Long, User])
      repo   = new UserRepository(store)
      found <- repo.find(999)
    yield assertEquals(found, None)
```

No mock framework. No test database. The repository takes a `Ref[IO, Map]` — in tests, you create an empty one. In production, you create one from a database connection. The code is the same.

### Testing HTTP Routes (http4s)

```scala
import munit.CatsEffectSuite
import org.http4s.*
import org.http4s.implicits.*
import io.circe.generic.auto.*
import org.http4s.circe.CirceEntityCodec.*

class UserRoutesSuite extends CatsEffectSuite:

  def app: HttpApp[IO] =
    val store = Ref.unsafe[IO, Map[Long, User]](Map(1L -> User(1, "Alice", "a@a.com")))
    val repo = UserRepository(store)
    routes(repo).orNotFound

  test("GET /users returns all users"):
    for
      response <- app.run(Request(method = GET, uri = uri"/users"))
      users    <- response.as[List[User]]
    yield assertEquals(users, List(User(1, "Alice", "a@a.com")))

  test("GET /users/1 returns the user"):
    for
      response <- app.run(Request(method = GET, uri = uri"/users/1"))
      user     <- response.as[User]
    yield assertEquals(user, User(1, "Alice", "a@a.com"))

  test("GET /users/999 returns 404"):
    for
      response <- app.run(Request(method = GET, uri = uri"/users/999"))
    yield assertEquals(response.status, Status.NotFound)

  test("POST /users creates a user"):
    val user = User(2, "Bob", "bob@example.com")
    for
      response <- app.run(Request(method = POST, uri = uri"/users").withEntity(user))
      created  <- response.as[User]
    yield assertEquals(response.status, Status.Created)
```

No running server. `app.run(request)` passes a request directly to the route function. Fast. Deterministic. No port conflicts.

## weaver — Alternative Test Framework

weaver is another Cats Effect-native test framework with a different API:

```scala
libraryDependencies += "com.disneystreaming" %% "weaver-cats" % "0.8.4" % Test
```

```scala
import weaver.IOSuite
import cats.effect.IO

object UserRepoSuite extends IOSuite:
  type Res = UserRepository

  def sharedResource: Resource[IO, UserRepository] =
    Resource.eval(IO.ref(Map.empty[Long, User])).map(new UserRepository(_))

  test("save and retrieve")(repo => for
    user   = User(1, "Alice", "a@a.com")
    _     <- repo.save(user)
    found <- repo.find(1)
  yield expect(found == Some(user)))
```

weaver uses `expect` instead of `assertEquals` and provides shared resource setup.

## Property-Based Testing with scalacheck

Generate random inputs to test properties that hold for all values:

```scala
libraryDependencies += "org.scalacheck" %% "scalacheck" % "1.18.0" % Test
```

```scala
import munit.ScalaCheckSuite
import org.scalacheck.Prop.*

class PropertySuite extends ScalaCheckSuite:
  property("list reverse is involutory"):
    forAll { (list: List[Int]) =>
      assertEquals(list.reverse.reverse, list)
    }

  property("addition is commutative"):
    forAll { (a: Int, b: Int) =>
      assertEquals(a + b, b + a)
    }
```

scalacheck generates hundreds of random inputs. Properties that pass for all random inputs are likely correct. Edge cases are discovered automatically.

## The Functional Testing Advantage

- **No mocks needed** — Pass real implementations with in-memory stores
- **No test database** — Use `Ref` for state, `IO` for effects
- **No running server** — Call route functions directly
- **Deterministic** — Pure functions, no hidden state
- **Fast** — In-memory, no I/O, no network

Next: [Observability](02-observability.md)
