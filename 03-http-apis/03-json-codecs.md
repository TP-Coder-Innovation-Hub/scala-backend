# JSON Codecs `[Mid]`

Every backend service serializes and deserializes JSON. In Scala, this is done through type classes — `Encoder[A]` (convert A to JSON) and `Decoder[A]` (convert JSON to A). The compiler ensures your types match.

## circe — The Standard

circe is the most widely used JSON library in the Scala FP ecosystem. It integrates with Cats and http4s.

```scala
libraryDependencies ++= Seq(
  "io.circe" %% "circe-core"    % "0.14.7",
  "io.circe" %% "circe-generic"  % "0.14.7",
  "io.circe" %% "circe-parser"   % "0.14.7"
)
```

### Automatic Derivation

```scala
import io.circe.*
import io.circe.generic.auto.*
import io.circe.parser.*
import io.circe.syntax.*

case class User(id: Int, name: String, email: String)

// Encode to JSON
val user = User(1, "Alice", "alice@example.com")
val json: Json = user.asJson
val jsonString: String = json.noSpaces
// {"id":1,"name":"Alice","email":"alice@example.com"}

// Decode from JSON
val decoded: Either[Error, User] = decode[User]("""{"id":1,"name":"Alice","email":"alice@example.com"}""")
// Right(User(1,Alice,alice@example.com))

// Decode with missing field — fails
val bad = decode[User]("""{"id":1,"name":"Alice"}""")
// Left(DecodingFailure(Attempt to decode value on failed cursor, List(DownField(email))))
```

Automatic derivation (`import io.circe.generic.auto.*`) generates codecs at compile time using reflection. Convenient, but adds compile time and provides less control.

### Semi-Automatic Derivation (Recommended for Production)

Explicitly derive codecs per type. Faster compilation, explicit control:

```scala
import io.circe.*
import io.circe.generic.semiauto.*

case class User(id: Int, name: String, email: String)

object User:
  given encoder: Encoder[User] = deriveEncoder[User]
  given decoder: Decoder[User] = deriveDecoder[User]
```

Use this in production. You see exactly which types have codecs. Compilation is faster.

### Custom Codecs

When the JSON shape doesn't match the Scala type:

```scala
case class Money(amount: BigDecimal, currency: String)

object Money:
  given encoder: Encoder[Money] = Encoder.instance { m =>
    Json.obj(
      "amount" -> m.amount.asJson,
      "currency" -> m.currency.asJson
    )
  }

  given decoder: Decoder[Money] = Decoder.instance { cursor =>
    for
      amount   <- cursor.get[BigDecimal]("amount")
      currency <- cursor.get[String]("currency")
    yield Money(amount, currency)
  }
```

### ADT Serialization

```scala
sealed trait PaymentStatus
case class Paid(amount: BigDecimal) extends PaymentStatus
case object Pending extends PaymentStatus
case class Failed(reason: String) extends PaymentStatus

object PaymentStatus:
  given encoder: Encoder[PaymentStatus] = Encoder.instance {
    case p: Paid     => Json.obj("type" -> "paid".asJson, "amount" -> p.amount.asJson)
    case Pending     => Json.obj("type" -> "pending".asJson)
    case f: Failed   => Json.obj("type" -> "failed".asJson, "reason" -> f.reason.asJson)
  }

  given decoder: Decoder[PaymentStatus] = Decoder.instance { cursor =>
    cursor.get[String]("type").flatMap {
      case "paid"     => cursor.get[BigDecimal]("amount").map(Paid.apply)
      case "pending"  => Right(Pending)
      case "failed"   => cursor.get[String]("reason").map(Failed.apply)
      case other      => Left(DecodingFailure(s"Unknown type: $other", cursor.history))
    }
  }
```

The type field disambiguates sum types in JSON. This pattern is common when the JSON wire format doesn't match Scala's ADT structure.

## play-json

Used in the Play/Lagom ecosystem. Not our primary recommendation (prefer circe for FP stacks), but common in existing projects:

```scala
import play.api.libs.json.*

case class User(id: Int, name: String, email: String)

object User:
  implicit val format: Format[User] = Json.format[User]  // macro-based derivation

val json = Json.toJson(User(1, "Alice", "a@a.com"))
val user = json.as[User]
```

## uPickle

Lightweight, fast, no macro dependencies. Good for minimal setups:

```scala
import upickle.default.*

case class User(id: Int, name: String, email: String) derives ReadWriter

val json = write(User(1, "Alice", "a@a.com"))
val user = read[User](json)
```

## Why JSON Codecs Matter

A JSON codec is the boundary between your typed domain and the untyped outside world. When the codec is type-driven:

- **Invalid JSON is caught at decode time** — the compiler ensures every field maps to a type
- **Missing fields are caught** — `Decoder` requires all non-optional fields
- **Type mismatches are caught** — a string where an int is expected fails with a clear error
- **API contracts are enforced** — your Scala types define the API shape

Next: [Middleware](04-middleware.md)
