# Doobie — Pure Functional JDBC `[Mid]`

Doobie is a pure functional database library. A database interaction is a value of type `ConnectionIO[A]` — a description of a computation that interacts with the database. You compose these descriptions, then hand them to a transactor that executes them.

## Setup

```scala
libraryDependencies ++= Seq(
  "org.tpolecat" %% "doobie-core"   % "1.0.0-RC5",
  "org.tpolecat" %% "doobie-hikari"  % "1.0.0-RC5",
  "org.postgresql" %  "postgresql"   % "42.7.3"
)
```

## Transactor — The Database Connection

A `Transactor` manages connections. You create one, pass it to Doobie programs:

```scala
import cats.effect.*
import doobie.*
import doobie.hikari.*

def transactor[F[_]: Async]: Resource[F, Transactor[F]] =
  for
    hikari <- HikariTransactor.newHikariTransactor[F](
      driverClassName = "org.postgresql.Driver",
      url             = "jdbc:postgresql://localhost:5432/mydb",
      user            = "postgres",
      pass            = "password"
    )
  yield hikari
```

## ConnectionIO — Describing Database Interactions

```scala
import doobie.*
import doobie.implicits.*

case class User(id: Long, name: String, email: String)
```

### Query (read)

```scala
def findAll: ConnectionIO[List[User]] =
  sql"SELECT id, name, email FROM users"
    .query[User]
    .to[List]

def find(id: Long): ConnectionIO[Option[User]] =
  sql"SELECT id, name, email FROM users WHERE id = $id"
    .query[User]
    .option
```

### Update (write)

```scala
def save(user: User): ConnectionIO[Int] =
  sql"INSERT INTO users (id, name, email) VALUES (${user.id}, ${user.name}, ${user.email})"
    .update
    .run

def delete(id: Long): ConnectionIO[Int] =
  sql"DELETE FROM users WHERE id = $id"
    .update
    .run
```

### Compose with for-comprehensions

```scala
def createAndFind(user: User): ConnectionIO[Option[User]] = for
  _    <- save(user)
  found <- find(user.id)
yield found
```

`ConnectionIO[A]` is a value. It does not touch the database. For-comprehensions build a composite description. The transactor runs it.

## Running Programs

```scala
import cats.effect.*

object Main extends IOApp.Simple:

  val run: IO[Unit] =
    transactor[IO].use { xa =>

      // .transact(xa) converts ConnectionIO to IO and executes it
      val program: IO[List[User]] = findAll.transact(xa)

      program.flatMap { users =>
        IO.println(users)
      }
    }
```

Step by step:
1. `transactor[IO]` — acquire a connection pool (Resource)
2. `.use { xa => ... }` — run with the transactor, release when done
3. `findAll` — a `ConnectionIO[List[User]]`
4. `.transact(xa)` — execute it, get `IO[List[User]]`
5. `IO.println` — print the results

## Parameterized Queries

```scala
// Safe interpolation — $variable becomes a prepared statement parameter
def findByName(name: String): ConnectionIO[List[User]] =
  sql"SELECT id, name, email FROM users WHERE name = $name"
    .query[User]
    .to[List]

// Fragments for dynamic queries
import doobie.Fragments

def search(name: Option[String], minId: Option[Long]): ConnectionIO[List[User]] = {
  val filters = List(
    name.map(n => fr"name = $n"),
    minId.map(id => fr"id > $id")
  ).flatten

  val where = if filters.isEmpty then Fragment.empty else Fragments.whereAnd(filters: _*)

  (fr"SELECT id, name, email FROM users" ++ where)
    .query[User]
    .to[List]
}
```

Doobie uses prepared statements automatically. The `sql"..."` interpolator creates parameterized queries. SQL injection is impossible by construction.

## Type Mappings

Doobie needs `Get[A]` and `Put[A]` instances for types used in queries. Built-in for basic types (Int, Long, String, BigDecimal, etc.). For custom types:

```scala
import doobie.*
import java.time.Instant

given Get[Instant] = Get[Long].map(Instant.ofEpochMilli)
given Put[Instant] = Put[Long].contramap(_.toEpochMilli)
```

## Why Doobie

- **Pure** — `ConnectionIO[A]` is a value. Testable, composable, no side effects until transact.
- **Type-safe** — The compiler checks column types match Scala types (via `Get`/`Put`).
- **SQL-first** — You write real SQL. No ORM abstraction leak. Full control over queries.
- **Composable** — For-comprehensions build complex transactions from simple queries.

Next: [Slick](02-slick.md)
