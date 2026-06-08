# Error Handling `[Mid]`

Exceptions are unchecked in Scala (and Java). The compiler does not track them. You don't know a method throws until it crashes at runtime. In a backend service, this means unhandled errors become 500 responses.

Typed errors fix this. The type signature says what can go wrong. The compiler ensures you handle it.

## Option — Presence or Absence

`Option[A]` represents a value that may not exist:

```scala
def findUser(id: Int): Option[User] =
  if id > 0 then Some(User(id, "Alice")) else None

findUser(1) match
  case Some(user) => println(s"Found: $user")
  case None       => println("Not found")
```

Use `Option` when absence is expected and not an error (e.g., looking up a key in a map).

## Either — Success or Typed Error

`Either[E, A]` is a value that is either `Left(error)` or `Right(success)`:

```scala
sealed trait AppError
case class NotFound(id: Int) extends AppError
case class InvalidInput(msg: String) extends AppError

def getUser(id: Int): Either[AppError, User] =
  if id <= 0 then Left(InvalidInput("id must be positive"))
  else if id > 1000 then Left(NotFound(id))
  else Right(User(id, "Alice"))

// For-comprehension chains Either values
def handleRequest(id: Int): Either[AppError, String] = for
  user    <- getUser(id)
  profile <- getProfile(user.id)
yield profile.name
```

The type `Either[AppError, String]` tells you: this computation either fails with `AppError` or succeeds with `String`. The compiler does not let you ignore the error case.

## Validated — Accumulating Errors

`Either` fails fast — the first error stops computation. `Validated` accumulates all errors:

```scala
import cats.data.Validated
import cats.data.Validated.{Valid, Invalid}
import cats.data.NonEmptyChain

case class Registration(name: String, email: String, age: Int)

def validateName(name: String): Validated[NonEmptyChain[String], String] =
  if name.isEmpty then Invalid(NonEmptyChain("Name cannot be empty"))
  else Valid(name)

def validateEmail(email: String): Validated[NonEmptyChain[String], String] =
  if !email.contains("@") then Invalid(NonEmptyChain("Invalid email"))
  else Valid(email)

def validateAge(age: Int): Validated[NonEmptyChain[String], Int] =
  if age < 18 then Invalid(NonEmptyChain("Must be 18 or older"))
  else Valid(age)

// Combine — accumulates all errors
def validate(name: String, email: String, age: Int): Validated[NonEmptyChain[String], Registration] =
  (
    validateName(name),
    validateEmail(email),
    validateAge(age)
  ).mapN(Registration.apply)
```

Step by step:
1. Each `validateX` returns `Validated` with potentially multiple errors wrapped in `NonEmptyChain`
2. `.mapN` combines three validations. If all pass, builds a `Registration`. If any fail, accumulates all error messages.
3. Use this for input validation where you want to report all problems at once.

```scala
validate("", "bad-email", 15)
// Invalid(Chain("Name cannot be empty", "Invalid email", "Must be 18 or older"))

validate("Alice", "alice@example.com", 25)
// Valid(Registration("Alice", "alice@example.com", 25))
```

## With Effect Types

Combine typed errors with effect types:

```scala
// Cats Effect
def getUser(id: Int): IO[Either[AppError, User]] = ???

// Better: use IO.raiseError / IO.fromEither
def getUser(id: Int): IO[User] =
  if id <= 0 then IO.raiseError(InvalidInput("id must be positive"))
  else IO.pure(User(id, "Alice"))

// ZIO: typed errors in the signature
def getUser(id: Int): ZIO[Any, AppError, User] =
  if id <= 0 then ZIO.fail(InvalidInput("id must be positive"))
  else ZIO.succeed(User(id, "Alice"))
```

## Why Typed Errors Beat Exceptions

| | Exceptions | Typed Errors |
|---|-----------|--------------|
| Visibility | Hidden in implementation | Visible in type signature |
| Compiler enforcement | None | Must handle or propagate |
| Composability | Break referential transparency | Compose with for-comprehensions |
| Documentation | Read the docs/source | Read the type |
| Testing | Hope you catch every throw path | Compiler lists unhandled cases |

In a backend service, typed errors mean:
- Every API endpoint's error responses are visible in the type
- The compiler catches forgotten error paths
- Input validation returns all errors at once, not one at a time
- Tests can assert on specific error types

Next: [HTTP APIs](../03-http-apis/01-http4s.md)
