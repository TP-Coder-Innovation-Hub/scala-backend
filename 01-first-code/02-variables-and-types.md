# Variables and Types ``

Scala has two kinds of variables: `val` (immutable) and `var` (mutable). Use `val` by default. Every variable has a type, but you often don't need to write it — the compiler figures it out.

## val vs var

```scala
val name: String = "Alice"   // Immutable. Cannot be reassigned.
var count: Int = 0           // Mutable. Can be reassigned.
```

Prefer `val`. Always. In functional Scala, `var` is a code smell. If you reach for `var`, there is almost always a better way.

```scala
val name = "Alice"   // Type inferred as String
val age = 30         // Type inferred as Int
val pi = 3.14        // Type inferred as Double
```

The compiler infers types from the right-hand side. You can omit the type annotation in most cases. Add explicit types for public APIs and when the inference picks something too generic.

## Basic Types

Scala's basic types map to JVM primitives but are objects:

| Scala Type | JVM Type | Description |
|-----------|----------|-------------|
| `Int` | `int` | 32-bit integer |
| `Long` | `long` | 64-bit integer |
| `Double` | `double` | 64-bit floating point |
| `Float` | `float` | 32-bit floating point |
| `Boolean` | `boolean` | `true` or `false` |
| `String` | `String` | Text (Java String, enriched) |
| `Char` | `char` | Single character |
| `Unit` | `void` | No meaningful value (like void) |
| `Nothing` | N/A | Bottom type — no values at all |
| `Any` | `Object` | Top type — everything is an Any |

```scala
val x: Int = 42
val y: Long = 1000000000L
val flag: Boolean = true
val letter: Char = 'A'
val nothing: Unit = ()
```

## String Interpolation

```scala
val name = "Alice"
val age = 30

// s-interpolation: plug in values
val greeting = s"Hello, $name. You are $age years old."

// f-interpolation: formatted (like printf)
val formatted = f"$name is $age%.1f years old"

// raw-interpolation: no escape processing
val path = raw"C:\Users\new\file.txt"  // backslashes preserved
```

`s"..."` is the most common. Use it whenever you build strings from variables.

## Type Safety in Action

The compiler stops you from mixing types:

```scala
val x: Int = 42
val y: String = "hello"

// val bad: String = x  // compilation error: found Int, required String
val ok: String = x.toString  // explicit conversion
val alsoOk: String = s"$x"   // string interpolation converts
```

## Why This Matters

In backend services, every variable represents domain data. Using `val` means data doesn't change unexpectedly. Using types means the compiler catches mismatches — you never accidentally pass an age where a name is expected.

Consider modeling a user:

```scala
val userId: Int = 1001
val userName: String = "alice"
val email: String = "alice@example.com"
val isActive: Boolean = true
```

Each field has a distinct type. The compiler won't let you swap them. This is trivial here but critical in a service with hundreds of fields across dozens of types.

Next: [Control Flow](03-control-flow.md)
