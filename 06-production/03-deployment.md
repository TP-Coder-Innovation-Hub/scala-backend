# Deployment ``

Scala runs on the JVM. Deployment means packaging your application into a format the target environment can run — usually a Docker container.

## Docker with sbt-native-packager

sbt-native-packager generates Docker images from your sbt project:

```scala
// project/plugins.sbt
addSbtPlugin("com.github.sbt" % "sbt-native-packager" % "1.10.4")
```

```scala
// build.sbt
enablePlugins(JavaAppPackaging, DockerPlugin)

Docker / packageName := "my-app"
Docker / dockerBaseImage := "eclipse-temurin:21-jre"
Docker / dockerExposedPorts := Seq(8080)
Docker / dockerUsername := Some("my-org")
```

Build and publish:

```bash
sbt Docker/publishLocal     # Build image locally
sbt Docker/publish           # Push to registry
```

The generated Dockerfile:
- Uses a slim JRE base image
- Copies your fat JAR
- Sets the entrypoint to `java -jar app.jar`
- Exposes configured ports

## Manual Dockerfile

For more control:

```dockerfile
FROM eclipse-temurin:21-jre AS runtime

WORKDIR /app
COPY target/scala-3.4.2/my-app.jar /app/app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

Build and run:

```bash
sbt package                           # Create the JAR
docker build -t my-app:latest .       # Build image
docker run -p 8080:8080 my-app:latest  # Run container
```

## GraalVM Native Image

JVM startup is slow (1-3 seconds). GraalVM native image compiles your Scala app to a standalone native binary:

- Startup in milliseconds
- Lower memory usage (no JVM overhead)
- Smaller container image

```scala
// project/plugins.sbt
addSbtPlugin("com.github.sbt" % "sbt-native-packager" % "1.10.4")
addSbtPlugin("org.scalameta"  % "sbt-native-image"     % "0.3.3")
```

```scala
// build.sbt
enablePlugins(NativeImagePlugin)
nativeImageOptions ++= Seq(
  "--no-fallback",
  "-H:+ReportExceptionStackTraces",
  "--initialize-at-build-time"
)
```

Build:

```bash
sbt nativeImage  # Produces a native binary in target/
```

Caveats:
- Not all libraries support native image (reflection-based libraries may fail)
- Build requires GraalVM installed
- Compile time is long (minutes, not seconds)
- Dynamic class loading and runtime reflection don't work without explicit configuration

Use native image for serverless functions, CLI tools, and services where startup time matters. Stick with JVM for long-running services where startup is negligible.

## sbt Docker Plugin

Alternative to sbt-native-packager for Docker builds:

```scala
// project/plugins.sbt
addSbtPlugin("se.marcuslonnberg" % "sbt-docker" % "1.10.0")
```

```scala
// build.sbt
enablePlugins(DockerPlugin)

dockerSettings := DockerSettings(
  baseImage = "eclipse-temurin:21-jre",
  exposedPorts = Seq(8080),
  repository = "my-org/my-app"
)
```

## Production Configuration

Externalize configuration. Never hardcode values:

```scala
// Pureconfig for typesafe config
libraryDependencies += "com.github.pureconfig" %% "pureconfig" % "0.17.7"
```

```hocon
# application.conf
app {
  http {
    host = "0.0.0.0"
    host = ${?HTTP_HOST}
    port = 8080
    port = ${?HTTP_PORT}
  }
  database {
    url = "jdbc:postgresql://localhost:5432/mydb"
    url = ${?DATABASE_URL}
    user = "postgres"
    user = ${?DATABASE_USER}
    password = "password"
    password = ${?DATABASE_PASSWORD}
  }
}
```

```scala
import pureconfig.ConfigSource
import pureconfig.generic.derivation.default.*

case class HttpConfig(host: String, port: Int)
case class DatabaseConfig(url: String, user: String, password: String)
case class AppConfig(http: HttpConfig, database: DatabaseConfig) derives ConfigReader

val config: IO[AppConfig] = IO.blocking(ConfigSource.default.loadOrThrow[AppConfig])
```

Environment variables override defaults (`${?ENV_VAR}`). In Kubernetes, inject config via ConfigMaps and Secrets.

## Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-org/my-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

JVM tuning for containers — set heap size based on container limits:

```dockerfile
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "/app/app.jar"]
```

`-XX:MaxRAMPercentage=75.0` tells the JVM to use 75% of container memory for heap. The rest is for off-heap, metaspace, and native memory.

Next: [Workshop](../07-workshop/README.md)
