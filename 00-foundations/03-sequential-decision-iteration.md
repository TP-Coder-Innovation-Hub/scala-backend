# Sequential, Decision, Iteration ``

Every program, regardless of language or paradigm, is built from three operations: sequential execution, decision, and iteration. Master these and you can build anything.

## 1. Sequential — Do This, Then That

Execute steps in order.

```scala
val name = "Alice"
val greeting = s"Hello, $name"
println(greeting)
```

Each line depends on the previous one. This is the default. Most of your code is sequential.

In functional programming, sequencing is often expressed through composition:

```scala
val result = parse(input).map(validate).flatMap(save)
```

`parse` runs, then `validate` runs on the result, then `save` runs on that result. Same sequential logic, expressed as function composition.

## 2. Decision — Do This OR That

Choose a path based on a condition.

```scala
if score >= 90 then "A"
else if score >= 80 then "B"
else if score >= 70 then "C"
else "F"
```

In Scala, `if/else` is an **expression** — it returns a value. This is different from languages where `if` is a statement (returns nothing). Scala expressions eliminate mutable variables:

```scala
// Expression-oriented: assign the result of a decision
val grade: String =
  if score >= 90 then "A"
  else "F"
```

Pattern matching extends decisions with structure:

```scala
score match
  case s if s >= 90 => "A"
  case s if s >= 80 => "B"
  case _            => "F"
```

## 3. Iteration — Repeat

Process a collection or repeat until a condition is met.

Imperative iteration (loops):
```scala
var i = 0
while i < 10 do
  println(i)
  i += 1
```

Functional iteration (higher-order functions):
```scala
(0 until 10).foreach(println)

// Or transform:
val doubled = (0 until 10).map(_ * 2)
```

In Scala, you rarely write raw loops. Instead, you use `map`, `filter`, `foldLeft`, `foreach` — functions that iterate for you. This is safer (no off-by-one errors), more readable, and composable.

## Why Scala Prefers Expressions

Scala pushes you toward expressions over statements:

- `if/else` returns a value (expression)
- `match` returns a value (expression)
- `try/catch` returns a value (expression)
- `for`/`yield` returns a value (expression — a comprehension, not a loop)

When every construct returns a value, you write fewer `var`s, have fewer side effects, and the compiler catches more mistakes.

Next: [Compiler vs Interpreter](04-compiler-vs-interpreter.md)
