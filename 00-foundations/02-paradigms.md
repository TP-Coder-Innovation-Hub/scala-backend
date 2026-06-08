# Programming Paradigms ``

A paradigm is a set of rules about how to structure code. The two dominant paradigms are Object-Oriented Programming (OOP) and Functional Programming (FP). Scala supports both. For backend services, FP is the primary approach.

## Object-Oriented Programming

OOP organizes code around objects — data structures that own both state and behavior. Key ideas: encapsulation, inheritance, polymorphism.

OOP works well for GUI frameworks, game entities, and modeling physical systems. It struggles with concurrency (shared mutable state), testability (hidden dependencies), and reasoning about program behavior (objects can change at any time).

## Functional Programming

FP organizes code around pure functions and immutable data. Key ideas:

- **Purity** — A function's output depends only on its inputs. Same input, same output. Every time. No exceptions.
- **Immutability** — Data never changes after creation. You create new data from old data.
- **Composition** — Small functions combine into larger ones. Like Lego blocks that snap together.
- **Referential transparency** — You can replace a function call with its result and nothing changes.

Why this matters for backend:

```scala
// Pure function — testable, predictable, composable
def calculateTax(income: BigDecimal, rate: BigDecimal): BigDecimal =
  income * rate

// Impure function — depends on external state, hard to test, unpredictable
var currentRate: BigDecimal = ???
def calculateTax(income: BigDecimal): BigDecimal =
  income * currentRate  // What is currentRate? Who changed it? When?
```

The pure version has no surprises. The compiler can reason about it. Tests are trivial. You can compose it with other functions without fear.

## Why FP for Scala Backend

Scala's type system combined with FP gives you:

1. **Compile-time correctness** — Types encode business rules. The compiler enforces them.
2. **Effect safety** — Side effects (database calls, HTTP requests) are encoded in types, not hidden.
3. **Testability** — Pure functions are trivially testable. Impure code is isolated and injectable.
4. **Concurrency safety** — Immutable data needs no locks. Pure functions have no race conditions.

OOP exists in Scala and is useful for structuring modules and interfaces. But the workhorse pattern for backend services is functional: pure functions, immutable data, encoded effects.

## The Mental Shift

If you come from OOP, the shift is:

- From *objects that do things* to *functions that transform data*
- From *mutable state* to *immutable values*
- From *exceptions for errors* to *types that represent errors*
- From *hidden side effects* to *explicit effect types*

This roadmap teaches Scala backend through the FP lens. That is where Scala's strengths are.

Next: [Sequential, Decision, Iteration](03-sequential-decision-iteration.md)
