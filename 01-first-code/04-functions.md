# Functions ``

Functions are the building blocks of functional programming. In Scala, functions are first-class citizens — they are values you can pass, store, and compose.

## Defining Functions

```scala
def add(a: Int, b: Int): Int = a + b
```

- `def` — keyword to define a function
- `add` — name
- `(a: Int, b: Int)` — parameters with types
- `: Int` — return type
- `= a + b` — body (an expression)

For multi-line bodies:

```scala
def greet(name: String): String =
  val greeting = s"Hello, $name"
  greeting.toUpperCase
```

The last expression in the body is the return value. No `return` keyword needed.

## Anonymous Functions (Lambdas)

```scala
val double = (x: Int) => x * 2

double(5)  // 10
```

Shorter syntax with underscores (when each parameter is used once):

```scala
val double: Int => Int = _ * 2
val add: (Int, Int) => Int = _ + _
```

## Higher-Order Functions

Functions that take functions as parameters or return functions:

```scala
def applyTwice(f: Int => Int, x: Int): Int = f(f(x))

applyTwice(_ * 2, 3)  // 12 (3 * 2 = 6, 6 * 2 = 12)
```

This is the core of functional composition. You build small functions and combine them:

```scala
val numbers = List(1, 2, 3, 4, 5)

numbers
  .map(_ * 2)          // List(2, 4, 6, 8, 10)
  .filter(_ > 5)       // List(6, 8, 10)
  .reduce(_ + _)        // 24
```

`map`, `filter`, `reduce`, `foldLeft`, `flatMap` — these are higher-order functions you will use daily. They replace loops with composable transformations.

## Functions vs Methods

A `def` is a method (belongs to a class/object). A `val` with a function type is a function value:

```scala
// Method
def square(x: Int): Int = x * x

// Function value (can be passed around)
val squareFn: Int => Int = x => x * x

// Convert method to function (eta expansion)
val squareRef = square _
```

In practice, Scala automatically converts methods to functions when needed (e.g., passing a method to a higher-order function).

## Closures

A closure captures variables from its enclosing scope:

```scala
def makeMultiplier(factor: Int): Int => Int =
  (x: Int) => x * factor  // captures `factor`

val triple = makeMultiplier(3)
triple(10)  // 30
```

The function `(x: Int) => x * factor` closes over `factor`. It remembers the value even after `makeMultiplier` returns.

## Currying

A curried function takes multiple parameter lists instead of one:

```scala
def add(a: Int)(b: Int): Int = a + b

val add5 = add(5) _  // Partial application: Int => Int
add5(3)  // 8
```

Currying enables partial application — fix some arguments, get a new function. This pattern is used heavily in Scala libraries for type-class resolution and DSL construction.

## Why This Matters

In Scala backend, you compose functions into pipelines:

```scala
val processRequest: Request => Either[Error, Response] =
  parse andThen validate andThen authorize andThen handle
```

Each step is a function. The pipeline is a function. Testable, composable, type-checked by the compiler.

Next: [Case Classes and ADTs](05-case-classes-and-adts.md)
