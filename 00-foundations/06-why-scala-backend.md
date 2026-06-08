# Why Scala Backend? `[Entry]`

You can build backend services in many languages. Here is why Scala deserves your attention, and when it is the wrong choice.

## Scala vs Java

Java is the safe default for JVM backend. Billions of lines in production. Massive talent pool.

Scala advantages over Java:
- **Expressiveness** — Less boilerplate. Pattern matching, case classes, for-comprehensions, type inference. A Scala service is often 2-3x shorter than the equivalent Java.
- **Type safety** — ADTs, type classes, higher-kinded types. Java has improved (sealed classes, records, pattern matching in recent versions), but Scala's type system remains more powerful.
- **FP support** — First-class functions, immutable-by-default, effect systems. Java has Streams and Optional but they are bolted on, not foundational.

Scala disadvantages:
- **Steeper learning curve** — More concepts, more abstraction power, more ways to write clever code
- **Slower compilation** — Scala compiler does more work
- **Smaller talent pool** — Fewer developers, harder to hire

## Scala vs Kotlin

Kotlin is the modern JVM language. Android's default. Pragmatic, accessible, Java-compatible.

Scala advantages over Kotlin:
- **FP power** — Kotlin has limited FP support. No higher-kinded types. No effect system ecosystem. Scala's Cats / ZIO ecosystem has no Kotlin equivalent.
- **Type system** — More expressive types enable more compiler-checked correctness.

Kotlin advantages:
- **Easier to learn** — Feels like a better Java. Onboarding is fast.
- **Google backing** — First-class Android support, Jetpack Compose, Coroutines.
- **Faster builds** — Simpler language = faster compilation.

Choose Kotlin for teams that want Java-but-better. Choose Scala when you want the type system to enforce business rules.

## Scala vs Go

Go is the cloud-native default. Simple, fast builds, great concurrency.

Scala advantages:
- **Safety** — Go's type system is minimal. No algebraic data types, no pattern matching exhaustiveness, no effect tracking. Scala catches entire categories of bugs at compile time that Go catches at runtime.
- **Expressiveness** — Go is intentionally simple. Scala embraces abstraction power.

Go advantages:
- **Speed** — Faster builds, faster startup, lower memory usage.
- **Simplicity** — Any developer can read Go code. Scala requires more training.
- **Cloud-native ecosystem** — Kubernetes, Docker, and cloud tools are written in Go.

## Who Uses Scala Backend

- **Stripe** — Payment processing, type-safe APIs
- **Morgan Stanley** — Financial systems, correctness-critical
- **Airbnb** — Data infrastructure, backend services
- **The Guardian** — Content platform, migrated from Java
- **Netflix** — Backend services, big data pipelines
- **Twitter** (historically) — Built much of the Scala ecosystem
- **HMRC** (UK tax authority) — Government digital services

## When Scala Backend Shines

- **Complex business domains** — When the domain has many states, transitions, and rules, Scala's type system encodes them
- **Type-driven design** — When you want the compiler to prove correctness, not just check syntax
- **High correctness requirements** — Financial systems, healthcare, regulated industries where bugs are expensive
- **Team stability** — When you can invest in training and the team will stay long enough to see returns

## When to Look Elsewhere

- **Startup MVP** — Use Go, Node, or Python for speed to market
- **Large team, high turnover** — The learning curve will eat productivity
- **Simple CRUD** — Scala's power is overkill for basic REST APIs over a database
- **Startup time matters** — Serverless cold starts favor Go or Node (though GraalVM native image helps)

Next: [First Code](../01-first-code/01-setup.md)
