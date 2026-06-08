# Streaming with fs2 ``

Not all data fits in memory. Log files, database result sets, HTTP request bodies, real-time events — these are streams. Processing them as in-memory lists will exhaust your heap.

fs2 (Functional Streams for Scala) provides pure functional streaming. Streams are values you compose. Backpressure is built in.

## Core Concepts

A stream has three parts:

> 🖼️ **[IMAGE_PLACEHOLDER]** — fs2 streaming source pipe sink backpressure pipeline

1. **Source** — Produces values (read file, listen to socket, generate sequence)
2. **Pipe** — Transforms values (filter, map, batch, parse)
3. **Sink** — Consumes values (write to file, send to database, print)

```scala
import fs2.Stream
import cats.effect.IO

// Source: a stream of integers
val numbers: Stream[IO, Int] = Stream.emits(1 to 1000)

// Pipe: transform
val doubled: Stream[IO, Int] = numbers.map(_ * 2)

// Pipe: filter
val evens: Stream[IO, Int] = doubled.filter(_ % 2 == 0)

// Sink: consume
val program: IO[Unit] = evens.evalMap(n => IO.println(n)).compile.drain
```

## Through — Composing Pipes

`through` chains pipes:

```scala
val pipeline: Stream[IO, Unit] = Stream.emits(1 to 1000)
  .through(_.map(_ * 2))          // double
  .through(_.filter(_ % 3 == 0))  // keep multiples of 3
  .through(_.take(10))            // first 10
  .through(_.evalMap(n => IO.println(n)))  // print each
```

## File Streaming

Read a file line by line without loading it all into memory:

```scala
import fs2.io.file.{Files, Path}
import cats.effect.IO

// Read file as a stream of bytes, split into lines, process each
val processFile: Stream[IO, Unit] = 
  Files[IO].readUtf8Lines(Path("huge-log.txt"))
    .filter(_.contains("ERROR"))     // keep error lines
    .through(_.take(100))            // first 100 errors
    .evalMap(line => IO.println(line))

// Run
processFile.compile.drain
```

Even if the file is 50GB, this uses constant memory. Lines are read one at a time, filtered, and discarded.

## Log Processing Pipeline — Complete Example

```scala
import fs2.{Stream, text}
import fs2.io.file.{Files, Path}
import cats.effect.*
import java.time.Instant

case class LogEntry(timestamp: Instant, level: String, message: String)

object LogProcessor:

  def parseLine(line: String): Option[LogEntry] = line.split(" ", 3) match
    case Array(ts, level, msg) =>
      Some(LogEntry(Instant.parse(ts), level, msg))
    case _ => None

  def process(inputPath: String, outputPath: String): Stream[IO, Unit] =
    Files[IO].readUtf8Lines(Path(inputPath))
      .map(parseLine)                    // parse each line
      .collect { case Some(e) => e }     // keep valid entries
      .filter(_.level == "ERROR")        // keep errors
      .groupWithin(100, 1.second)        // batch: up to 100 or 1 second
      .evalMap { batch =>
        // Write batch to database or output file
        val lines = batch.map(e => s"${e.timestamp} ${e.level} ${e.message}")
        IO.println(s"Processing batch of ${batch.size} errors")
      }
```

Step by step:
1. `readUtf8Lines` — stream lines from file (constant memory)
2. `map(parseLine)` — parse each line to `Option[LogEntry]`
3. `collect { case Some(e) => e }` — keep valid entries, discard parse failures
4. `filter(_.level == "ERROR")` — keep only error-level entries
5. `groupWithin(100, 1.second)` — batch into chunks of 100 or 1-second windows (whichever comes first)
6. `evalMap` — process each batch (write to DB, send alerts, etc.)

## Backpressure

Backpressure is automatic. If the consumer is slower than the producer, the producer is slowed down. No unbounded buffering. No out-of-memory crashes.

> 🖼️ **[IMAGE_PLACEHOLDER]** — fs2 backpressure automatic producer consumer flow control

```scala
// Fast producer (every 10ms) → slow consumer (every 100ms)
// fs2 automatically slows the producer to match
val stream = Stream.awakeEvery[IO](10.milliseconds)
  .evalMap { _ =>
    IO.println("Produced") *> IO.sleep(100.milliseconds) *> IO.println("Consumed")
  }
  .take(10)
```

## Streaming from HTTP (http4s)

http4s request/response bodies are fs2 streams:

```scala
// Stream request body
val processBody: IO[Unit] =
  req.bodyStream    // Stream[IO, Byte]
    .through(text.utf8.decode)   // Stream[IO, String]
    .through(text.lines)         // Stream[IO, String]
    .evalMap(line => IO.println(line))
    .compile
    .drain
```

## Why fs2

- **Constant memory** — Process terabytes of data with bounded heap
- **Backpressure** — No manual throttling, no buffer overflow
- **Composition** — Streams are values. Combine, split, merge, filter as needed
- **Integration** — Works natively with http4s (request/response bodies), Doobie (streaming query results), and Cats Effect

Next: [Supervision](03-supervision.md)
