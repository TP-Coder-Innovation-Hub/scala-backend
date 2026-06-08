# Setup ``

Get your machine ready to write and run Scala. Three things: JDK, build tool, editor.

## 1. Install JDK

Scala runs on the JVM. You need JDK 17 or later (JDK 21 recommended).

```bash
# macOS (Homebrew)
brew install openjdk@21

# Linux (sdkman — recommended for managing JDK versions)
curl -s "https://get.sdkman.io" | bash
sdk install java 21.0.2-tem
```

Verify:

```bash
java -version
# openjdk version "21.0.2" or similar
```

## 2. Install Build Tool

You have two options:

### Option A: sbt (Standard)

sbt is the most widely used Scala build tool. Most production projects use it.

```bash
# macOS
brew install sbt

# Linux (sdkman)
sdk install sbt
```

Create a project:

```bash
mkdir my-project && cd my-project
```

Create `build.sbt`:

```scala
val scala3Version = "3.4.2"

lazy val root = project
  .in(file("."))
  .settings(
    name := "my-project",
    version := "0.1.0",
    scalaVersion := scala3Version
  )
```

Create the directory structure:

```bash
mkdir -p src/main/scala
```

Create `src/main/scala/Main.scala`:

```scala
@main def hello(): Unit =
  println("Hello, Scala!")
```

Run:

```bash
sbt run
# Hello, Scala!
```

### Option B: Scala CLI (Lightweight)

Scala CLI is faster for experimentation and scripts. Good for learning.

```bash
# macOS
brew install Virtuslab/brew/scala-cli

# Linux
curl -fL https://github.com/Virtuslab/scala-cli/releases/latest/download/scala-cli-x86_64-pc-linux.gz | gzip -d > scala-cli && chmod +x scala-cli
```

Create `hello.scala`:

```scala
@main def hello(): Unit =
  println("Hello, Scala!")
```

Run:

```bash
scala-cli run hello.scala
# Hello, Scala!
```

**For this roadmap, use sbt for full projects and Scala CLI for quick experiments.**

## 3. Install Editor

### VS Code + Metals (Recommended)

Metals is the Scala language server. It provides type information, go-to-definition, completions, and error highlighting.

```bash
# Install VS Code extension
code --install-extension scalameta.metals
```

Open your project in VS Code. Metals will detect `build.sbt` and index the project. Wait for indexing to complete (first time takes a minute).

### IntelliJ IDEA + Scala Plugin

IntelliJ with the Scala plugin has the best Scala support. Free with Community Edition.

1. Install IntelliJ IDEA Community Edition
2. Go to Plugins, search "Scala", install
3. Open your project (import from existing sources, choose sbt)

## Verify Everything Works

```bash
sbt new scala/scala3.g8
# Follow prompts, navigate to created project
cd scala3-project
sbt run
# Hello, World!
```

You should see "Hello, World!". If you do, your setup is complete.

Next: [Variables and Types](02-variables-and-types.md)
