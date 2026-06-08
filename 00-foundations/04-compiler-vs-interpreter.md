# Compiler vs Interpreter `[Entry]`

Programs are written in human-readable text. Machines execute binary instructions. Something must translate between the two. That something is either a compiler or an interpreter.

## Compiled Languages

A compiler translates your entire program into machine code **before** it runs. You write code, run the compiler, get an executable, then run the executable.

Advantages:
- Errors caught before execution (type errors, syntax errors, missing imports)
- Optimized code (the compiler can analyze the whole program)
- No runtime translation overhead

Disadvantages:
- Slower feedback loop (compile, then run)
- Platform-specific binaries (a Linux binary won't run on Windows without recompilation)

## Interpreted Languages

An interpreter reads and executes your code **line by line** at runtime. No separate compilation step.

Advantages:
- Fast feedback (run immediately)
- Platform-independent (any machine with the interpreter can run it)

Disadvantages:
- Errors discovered at runtime (type errors, undefined variables — they crash in production)
- Slower execution (translation happens every time)

## Scala: Compiled to JVM Bytecode

Scala takes a hybrid approach:

1. **Compile** Scala source code (`.scala`) into JVM bytecode (`.class` files) using `scalac`
2. **JVM** (Java Virtual Machine) runs the bytecode
3. **JIT** (Just-In-Time compiler) further optimizes hot code paths at runtime

This means:

- **Compile-time type checking** — The Scala compiler catches type errors, missing methods, pattern match exhaustiveness violations, and many logic errors before your code runs. This is Scala's superpower for backend development.
- **Cross-platform** — Any machine with a JVM runs your code. Linux, macOS, Windows, same bytecode.
- **JVM ecosystem** — You have access to every Java library ever written. Decades of battle-tested code.
- **JIT optimization** — The JVM monitors running code and optimizes hot paths. Your code gets faster the longer it runs.

## Why Compile-Time Checking Matters for Backend

In a backend service, a runtime type error means:

- A 500 error for a user
- A crash at 3 AM
- A pager alert
- An incident report

In Scala, that error is caught at compile time — before deployment. The compiler says "this doesn't type check" and you fix it at your desk, in daylight, with coffee.

```scala
// Compiler catches this: String cannot be compared with Int
val name: String = "Alice"
val result = name > 42  // compilation error
```

This is the value proposition: **shift bugs left**. Find them at compile time, not runtime.

Next: [What Is Scala](05-what-is-scala.md)
