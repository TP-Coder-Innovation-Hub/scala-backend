# Observability ``

You cannot operate what you cannot see. Observability is logging, metrics, and tracing — the three pillars that tell you what your service is doing in production.

## Logging

### Using log4cats (Cats Effect)

log4cats provides a pure functional logging interface:

```scala
libraryDependencies += "org.typelevel" %% "log4cats-slf4j" % "2.7.0"
```

```scala
import cats.effect.IO
import org.typelevel.log4cats.Logger
import org.typelevel.log4cats.slf4j.Slf4jLogger

object UserService:
  given Logger[IO] = Slf4jLogger.getLogger[IO]

  def findUser(id: Long): IO[Option[User]] =
    for
      _    <- Logger[IO].info(s"Looking up user $id")
      user <- repo.find(id)
      _    <- Logger[IO].info(s"User lookup result: ${user.map(_.name).getOrElse("not found")}")
    yield user
```

Logger is an effect — `Logger[IO].info(msg)` returns `IO[Unit]`. No side effects escape. Logger can be injected (via given/using) and swapped in tests.

### Structured Logging

Log structured data (JSON) for machine parsing:

```scala
import org.typelevel.log4cats.structlog.StructuredLogger

def handleRequest(req: Request): IO[Response] =
  val log = StructuredLogger[IO]
  for
    _      <- log.info(Map("method" -> req.method.name, "path" -> req.uri.path.toString))("Request received")
    result <- routes(req)
    _      <- log.info(Map("status" -> result.status.code.toString))("Request completed")
  yield result
```

Structured logs are searchable in log aggregation systems (ELK, CloudWatch Logs Insights, Datadog).

## Metrics

### Prometheus with http4s

```scala
libraryDependencies ++= Seq(
  "org.http4s" %% "http4s-prometheus-metrics" % "0.23.25"
)
```

```scala
import org.http4s.server.middleware.Metrics
import org.http4s.metrics.prometheus.Prometheus

for
  collector <- Prometheus.metricsOps[IO]("app")
  meteredRoutes = Metrics[IO](collector)(routes(repo))
  exposed = Router("/" -> meteredRoutes, "/metrics" -> Prometheus.exportService)
yield exposed
```

This gives you:

- Request count by method/path/status
- Request duration histograms (latency percentiles)
- Active request gauge

Scrape `/metrics` with Prometheus. Visualize in Grafana.

### Custom Metrics

```scala
import io.prometheus.client.Counter

val requestCounter = Counter.build()
  .name("api_requests_total")
  .help("Total API requests")
  .labelNames("method", "path", "status")
  .register()

def recordRequest(method: String, path: String, status: Int): IO[Unit] =
  IO(requestCounter.labels(method, path, status.toString).inc())
```

## Tracing

Tracing follows a request across service boundaries. When Service A calls Service B which calls Service C, a trace links all three calls:

```scala
// With natchez (Cats Effect tracing)
libraryDependencies += "org.tpolecat" %% "natchez-http4s" % "0.5.0"

import natchez.Trace
import natchez.Span

def getUser(id: Long)(using Trace[IO]): IO[User] =
  Trace[IO].span("getUser") {
    for
      _    <- Trace[IO].put("user.id" -> id)
      user <- repo.find(id)
      _    <- Trace[IO].put("user.found" -> user.isDefined)
    yield user
  }
```

Traces export to Jaeger, Zipkin, Honeycomb, Datadog. Each span records timing and metadata. You see exactly where time is spent.

## Observing Functional Programs

Functional programs are sometimes harder to observe because the description and execution are separate. Guidelines:

1. **Log at boundaries** — Log when entering/exiting effect boundaries (HTTP handlers, database calls, external API calls). Don't log inside pure functions.

2. **Use effect-safe logging** — log4cats returns `IO[Unit]`. Never use `println` or a side-effecting logger inside `IO`.

3. **Trace effect chains** — Use natchez spans to trace the execution of for-comprehensions. Each step gets a span.

4. **Measure at the edges** — HTTP middleware measures request latency. Database middleware measures query time. Don't instrument every function.

5. **Correlate with request IDs** — Add a request ID to all logs and traces for a request. Makes debugging possible.

```scala
// Request ID middleware
def requestIdMiddleware(routes: HttpRoutes[IO]): HttpRoutes[IO] = Kleisli { req =>
  val id = req.headers.get[RequestId].map(_.value).getOrElse(java.util.UUID.randomUUID.toString)
  for
    _     <- Logger[IO].info(Map("requestId" -> id))("Request started")
    resp  <- routes(req.putHeaders(Header.Raw(CIString("X-Request-Id"), id)))
    _     <- Logger[IO].info(Map("requestId" -> id, "status" -> resp.status.code.toString))("Request completed")
  yield resp
}
```

Next: [Deployment](03-deployment.md)
