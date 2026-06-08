# Slick — Functional-Relational Mapping `[Mid]`

Slick is a database library that provides a type-safe, LINQ-like query DSL. Instead of writing SQL strings, you write Scala code that the compiler type-checks, then Slick translates to SQL.

## Setup

```scala
libraryDependencies ++= Seq(
  "com.typesafe.slick" %% "slick"           % "3.5.1",
  "com.typesafe.slick" %% "slick-hikaricp"   % "3.5.1",
  "org.postgresql"      % "postgresql"       % "42.7.3"
)
```

## Table Definitions

Define your database schema as Scala classes:

```scala
import slick.jdbc.PostgresProfile.api.*

class Users(tag: Tag) extends Table[(Long, String, String)](tag, "users"):
  def id    = column[Long]("id", O.PrimaryKey)
  def name  = column[String]("name")
  def email = column[String]("email")
  def *     = (id, name, email)  // default projection

val users = TableQuery[Users]
```

For case class mapping:

```scala
case class User(id: Long, name: String, email: String)

class Users(tag: Tag) extends Table[User](tag, "users"):
  def id    = column[Long]("id", O.PrimaryKey)
  def name  = column[String]("name")
  def email = column[String]("email")
  def *     = (id, name, email).mapTo[User]

val users = TableQuery[Users]
```

## Queries — Type-Safe SQL

```scala
// SELECT * FROM users
val allUsers = users.result

// SELECT * FROM users WHERE id = ?
def findUser(id: Long) = users.filter(_.id === id).result.headOption

// INSERT
def insertUser(user: User) = users += user

// UPDATE
def updateName(id: Long, newName: String) =
  users.filter(_.id === id).map(_.name).update(newName)

// DELETE
def deleteUser(id: Long) = users.filter(_.id === id).delete
```

The compiler type-checks these queries. If you write `users.filter(_.age === 42)` and `age` doesn't exist as a column, it doesn't compile.

## Running Queries

```scala
import slick.jdbc.PostgresProfile.api.*

val db = Database.forConfig("mydb")

// db.run executes a DBIO action
val action: DBIO[Seq[User]] = users.result

import scala.concurrent.Future
val result: Future[Seq[User]] = db.run(action)

result.onComplete { users =>
  users.foreach(println)
}
```

`DBIO[A]` is a database action (similar to Doobie's `ConnectionIO[A]`). Compose with for-comprehensions:

```scala
val program: DBIO[Option[User]] = for
  _     <- users += User(1, "Alice", "alice@example.com")
  found <- users.filter(_.id === 1L).result.headOption
yield found

db.run(program.transactionally)  // wraps in a transaction
```

## Joins

```scala
case class Order(id: Long, userId: Long, total: BigDecimal)

class Orders(tag: Tag) extends Table[Order](tag, "orders"):
  def id     = column[Long]("id", O.PrimaryKey)
  def userId = column[Long]("user_id")
  def total  = column[BigDecimal]("total")
  def *      = (id, userId, total).mapTo[Order]
  def user   = foreignKey("fk_user", userId, users)(_.id)

val orders = TableQuery[Orders]

// Join: users with their orders
val query = for
  order <- orders
  user  <- order.user
yield (user.name, order.total)

db.run(query.result)
```

Foreign keys become navigation methods. Joins look like for-comprehensions over collections. Slick generates the SQL.

## Slick vs Doobie

| Aspect | Slick | Doobie |
|--------|-------|--------|
| Query style | Scala DSL (LINQ-like) | Raw SQL strings |
| Type safety | DSL is compile-time checked | SQL checked at runtime (column types) |
| Effect system | Scala Future | Cats Effect IO |
| Learning curve | DSL to learn | SQL you already know |
| Control | Slick generates SQL | You write exact SQL |
| FP alignment | Functional but Future-based | Pure FP, Cats Effect native |

**Choose Slick when:**
- You want compile-time query checking
- Your team prefers a DSL over raw SQL
- Queries are standard (no exotic SQL features)

**Choose Doobie when:**
- You want full SQL control (complex queries, DB-specific features)
- You use Cats Effect throughout
- You prefer SQL-as-code over a query DSL

Next: [Migrations](03-migrations.md)
