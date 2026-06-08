# Middleware ``

Middleware wraps routes to add cross-cutting concerns: authentication, logging, CORS, error handling. In functional Scala, middleware is function composition — a function that takes a route and returns a new route with added behavior.

## http4s Middleware

### Authentication

```scala
import org.http4s.*
import org.http4s.dsl.io.*
import cats.effect.IO

def authMiddleware(routes: HttpRoutes[IO]): HttpRoutes[IO] = Kleisli { req =>
  req.headers.get[Authorization] match
    case Some(Authorization(Bearer(token))) if token.value == "secret" =>
      routes(req)  // Token valid, pass to routes
    case _ =>
      OptionT.pure(Response[IO](Status.Unauthorized))  // Reject
}
```

Apply it by wrapping your routes:

```scala
val protectedRoutes = authMiddleware(routes(repo))
```

### Logging

```scala
import org.http4s.server.middleware.Logger
import cats.effect.IO

val loggedRoutes: HttpRoutes[IO] = Logger.httpRoutes[IO](
  logHeaders = true,
  logBody = true
)(routes)
```

http4s ships with built-in middleware for common needs. `Logger` logs request/response details.

### CORS

```scala
import org.http4s.server.middleware.CORS
import cats.effect.IO

val corsRoutes: HttpRoutes[IO] = CORS.policy.withAllowOriginAll
  .withAllowMethodsAll
  .withAllowHeadersAll
  .apply(routes)
```

### Error Handling

Wrap routes to catch unhandled errors and return structured responses:

```scala
import cats.effect.IO
import org.http4s.*
import io.circe.syntax.*
import io.circe.Json

def errorHandler(routes: HttpRoutes[IO]): HttpRoutes[IO] = Kleisli { req =>
  routes(req).handleErrorWith { error =>
    val status = error match
      case _: IllegalArgumentException => Status.BadRequest
      case _: NoSuchElementException  => Status.NotFound
      case _                           => Status.InternalServerError

    val body = Json.obj(
      "error" -> error.getMessage.asJson,
      "status" -> status.code.asJson
    )

    OptionT.pure(Response[IO](status).withEntity(body))
  }
}
```

### Composing Middleware

Middleware composes. Apply multiple layers:

```scala
val app = errorHandler(
  authMiddleware(
    CORS.policy.withAllowOriginAll.apply(
      Logger.httpRoutes(true, true)(
        routes(repo)
      )
    )
  )
).orNotFound
```

Order matters. Outer middleware runs first on the request, last on the response. The pattern above:
1. Request enters `errorHandler`
2. Passes to `authMiddleware`
3. Passes to `CORS`
4. Passes to `Logger`
5. Reaches `routes`
6. Response flows back out through Logger, CORS, auth, errorHandler

## Akka HTTP Middleware (Directives)

In Akka HTTP, cross-cutting concerns are directives composed into routes:

```scala
import akka.http.scaladsl.server.Directives.*
import akka.http.scaladsl.server.Route
import akka.http.scaladsl.model.StatusCodes
import akka.http.scaladsl.model.headers.*

// Authentication as a directive
def authenticated(inner: String => Route): Route =
  headerValueByName("Authorization") { auth =>
    if auth == "Bearer secret" then inner("user-123")
    else complete(StatusCodes.Unauthorized -> "Invalid token")
  }

// Logging as a directive
def logged(inner: => Route): Route =
  extractRequest { req =>
    println(s"${req.method.name} ${req.uri}")
    inner
  }

// CORS as a directive
def corsHeaders(inner: => Route): Route =
  respondWithHeaders(
    `Access-Control-Allow-Origin`.*,
    `Access-Control-Allow-Methods`(HttpMethods.GET, HttpMethods.POST, HttpMethods.DELETE)
  )(inner)

// Compose
val protectedRoutes: Route =
  corsHeaders(
    logged(
      authenticated { userId =>
        routes(repo)
      }
    )
  )
```

## The Pattern: Wrap, Don't Scatter

Without middleware, every route repeats auth, logging, error handling:

```scala
// DON'T do this
case GET -> Root / "users" =>
  if isAuthenticated(req) then
    log("GET /users")
    try repo.findAll.flatMap(Ok(_))
    catch handleError
  else Unauthorized()
```

With middleware, routes focus on business logic:

```scala
// DO this — middleware handles cross-cutting concerns
case GET -> Root / "users" =>
  repo.findAll.flatMap(Ok(_))
```

Authentication, logging, CORS, error handling — each is a separate middleware. Applied once. Composable. Testable independently.

Next: [Database](../04-database/01-doobie.md)
