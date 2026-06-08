# Akka HTTP ``

Akka HTTP is built on the Akka actor system. It uses a directive-based DSL for routing where you compose small building blocks (directives) into complete route trees.

## When Akka HTTP vs http4s

- **Akka HTTP** — You already use Akka (actors, streams, cluster). You want the Akka ecosystem integration. You prefer the directives DSL.
- **http4s** — You want pure FP. You use Cats Effect. You want maximum type safety and testability.

Both are production-ready. Choose based on your stack.

## Setup

```scala
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-http"            % "10.6.3",
  "com.typesafe.akka" %% "akka-http-spray-json"  % "10.6.3",
  "com.typesafe.akka" %% "akka-actor-typed"      % "2.8.5",
  "com.typesafe.akka" %% "akka-stream"           % "2.8.5"
)
```

## CRUD API — Step by Step

### Domain and JSON format

```scala
import spray.json.DefaultJsonProtocol.*
import spray.json.RootJsonFormat

case class User(id: Long, name: String, email: String)

object UserJson:
  implicit val userFormat: RootJsonFormat[User] = jsonFormat3(User.apply)
```

### Repository

```scala
import scala.collection.concurrent.TrieMap
import scala.concurrent.Future

class UserRepository:
  private val store = TrieMap.empty[Long, User]

  def findAll(): Future[List[User]] =
    Future.successful(store.values.toList)

  def find(id: Long): Future[Option[User]] =
    Future.successful(store.get(id))

  def save(user: User): Future[Unit] =
    Future.successful(store.put(user.id, user))

  def delete(id: Long): Future[Boolean] =
    Future.successful(store.remove(id).isDefined)
```

### Routes with Directives

```scala
import akka.http.scaladsl.server.Directives.*
import akka.http.scaladsl.server.Route
import akka.http.scaladsl.model.StatusCodes
import UserJson.*

def routes(repo: UserRepository): Route =
  pathPrefix("users")(
    concat(
      // GET /users
      get {
        onSuccess(repo.findAll()) { users =>
          complete(users)
        }
      },
      // POST /users
      post {
        entity(as[User]) { user =>
          onSuccess(repo.save(user)) {
            complete(StatusCodes.Created -> user)
          }
        }
      },
      // Path-based routes
      path(LongNumber) { id =>
        concat(
          // GET /users/:id
          get {
            onSuccess(repo.find(id)) {
              case Some(user) => complete(user)
              case None       => complete(StatusCodes.NotFound -> s"User $id not found")
            }
          },
          // DELETE /users/:id
          delete {
            onSuccess(repo.delete(id)) { deleted =>
              if deleted then complete(StatusCodes.NoContent)
              else complete(StatusCodes.NotFound -> s"User $id not found")
            }
          }
        )
      }
    )
  )
```

Step by step:
1. `pathPrefix("users")` — matches paths starting with /users
2. `concat(...)` — combines multiple directives (try each in order)
3. `get`, `post`, `delete` — match HTTP method
4. `entity(as[User])` — extracts and deserializes JSON body using spray-json
5. `onSuccess(...)` — unwraps the Future, handles the async result
6. `path(LongNumber)` — extracts a Long from the path
7. `complete(...)` — sends the response (auto-serialized to JSON)

### Start the server

```scala
import akka.actor.typed.ActorSystem
import akka.actor.typed.scaladsl.Behaviors
import akka.http.scaladsl.Http
import scala.concurrent.ExecutionContextExecutor

object Main:
  def main(args: Array[String]): Unit =
    implicit val system: ActorSystem[?] = ActorSystem(Behaviors.empty, "my-system")
    implicit val ec: ExecutionContextExecutor = system.executionContext

    val repo = new UserRepository
    val binding = Http().newServerAt("0.0.0.0", 8080).bind(routes(repo))

    println(s"Server at http://localhost:8080/")
```

## Directives — The Composable Building Blocks

Directives are small route fragments that compose. Think of them as Lego blocks:

```scala
// Authentication directive
def authenticated: Directive1[String] =
  headerValueByName("Authorization").flatMap { token =>
    if token == "Bearer secret" then provide(token)
    else complete(StatusCodes.Unauthorized -> "Invalid token")
  }

// Use it: wrap any route with authentication
def protectedRoutes(repo: UserRepository): Route =
  authenticated { userId =>
    routes(repo)  // All routes now require authentication
  }
```

Common directives:
- `path`, `pathPrefix` — URL matching
- `get`, `post`, `put`, `delete` — method filtering
- `entity(as[T])` — request body extraction
- `parameters` — query parameter extraction
- `headerValueByName` — header extraction
- `onSuccess` — Future unwrapping
- `complete` — response building

## Comparison with http4s

| Aspect | http4s | Akka HTTP |
|--------|--------|-----------|
| Effect system | Cats Effect IO | Scala Future / Akka Streams |
| Routing style | Pattern matching on requests | Directive DSL |
| JSON | circe | spray-json / Jackson |
| Streaming | fs2 | Akka Streams |
| Actor integration | None (by design) | Built-in |
| Learning curve | Steeper (FP concepts) | Moderate (directive model) |

Next: [JSON Codecs](03-json-codecs.md)
