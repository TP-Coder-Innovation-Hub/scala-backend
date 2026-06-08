# Control Flow `[Entry]`

Scala control flow is expression-oriented. `if/else`, `match`, and `for` all return values. This is a fundamental difference from most languages where control flow is statement-based.

## if/else — An Expression

`if/else` returns a value. Use it like a ternary operator, but readable:

```scala
val status = if score >= 60 then "pass" else "fail"
```

Multi-branch:

```scala
val grade =
  if score >= 90 then "A"
  else if score >= 80 then "B"
  else if score >= 70 then "C"
  else if score >= 60 then "D"
  else "F"
```

Because it's an expression, you assign the result directly to a `val`. No mutable variable needed.

## Pattern Matching — Scala's Power Tool

`match` / `case` is Scala's pattern matching. It is more powerful than a switch statement:

```scala
val description = score match
  case s if s >= 90 => "excellent"
  case s if s >= 70 => "good"
  case s if s >= 60 => "passing"
  case _            => "failing"
```

Key features:

- **Exhaustiveness checking** — If you match on a `sealed trait`, the compiler warns you if you miss a case
- **Guards** — `case x if condition =>` adds extra conditions
- **Wildcard** — `case _ =>` matches anything (like `default`)
- **Destructuring** — Extract fields from case classes

```scala
// Destructuring case classes
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape

def area(shape: Shape): Double = shape match
  case Circle(r)        => math.Pi * r * r
  case Rectangle(w, h)  => w * h
```

The compiler checks that every `Shape` is handled. If you add `Triangle` later, the compiler warns you to update `area`.

## for-Comprehensions — Not Loops

Scala's `for` is not a loop. It is a comprehension — syntactic sugar for `map`, `flatMap`, and `filter`:

```scala
// Imperative-looking, but it's actually map/flatMap
val result = for
  x <- List(1, 2, 3)
  y <- List(10, 20)
yield x + y
// result: List(11, 21, 12, 22, 13, 23)
```

With `yield`, it produces a new collection. Without `yield`, it runs for side effects:

```scala
for
  line <- lines
  if line.nonEmpty  // filter (guard)
do println(line)
```

for-comprehensions work on anything with `map` and `flatMap` — `List`, `Option`, `Either`, `IO`, `ZIO`, `Future`. This uniformity is critical: the same syntax that iterates over lists also sequences async operations. You will see this everywhere in effect systems.

## while Loops — Rarely Needed

```scala
var i = 0
while i < 10 do
  println(i)
  i += 1
```

`while` exists but is rarely used in functional Scala. It requires `var` and returns `Unit` — two signs that there is a better way.

## Expressions Over Statements

The theme: prefer expressions (return values) over statements (perform actions). This eliminates mutable variables, makes code testable, and lets the compiler verify more.

Next: [Functions](04-functions.md)
