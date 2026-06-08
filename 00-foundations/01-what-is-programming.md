# What Is Programming? `[Entry]`

A program is a recipe. It takes inputs, follows steps, and produces outputs. That's it.

Consider a recipe for tea:

```
1. Boil water
2. Put tea bag in cup
3. Pour water over tea bag
4. Wait 3 minutes
5. Remove tea bag
6. Serve
```

A program is the same thing: precise instructions a machine executes. The machine has no intuition. Every step must be explicit and unambiguous.

## Three Operations

All programs reduce to three operations:

1. **Sequential** — Do this, then that. Step 1, then step 2.
2. **Decision** — If the water is boiling, proceed. Otherwise, wait.
3. **Iteration** — Stir three times. Repeat until dissolved.

Every program you will ever write, in any language, is built from these three primitives. Loops, function calls, network requests, database queries — all are combinations of sequence, decision, and iteration.

## Why This Matters

Understanding that programs are structured recipes gives you a mental model that scales. A 10-line script and a 10-million-line system both follow the same rules. The difference is how you organize the recipes.

In functional programming (which Scala favors), you compose small recipes into larger ones. Each small recipe does one thing. You combine them. The type system checks that the recipes fit together before you run anything.

## What Code Actually Does

When you write code, you are:

1. **Defining data** — What information exists (a user, an order, a temperature)
2. **Defining transformations** — How data changes (validate input, calculate total, format response)
3. **Defining effects** — What interacts with the outside world (read file, send HTTP response, write to database)

Scala's strength is making all three explicit in types. The compiler verifies your recipe before the oven ever turns on.

Next: [Paradigms](02-paradigms.md)
