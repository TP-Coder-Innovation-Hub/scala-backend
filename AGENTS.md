# AGENTS.md

Instructions for AI assistants working with this repository.

## Repository Purpose

This is a **Scala backend roadmap** — educational content teaching type-safe, functional backend development. It is NOT about data engineering, Spark, Flink, or the Play framework.

## Stack

- Scala 3 (primary), Scala 2 (migration notes only)
- Cats Effect 3 or ZIO 2 for effect systems
- http4s or Akka HTTP for HTTP services
- Doobie or Slick for database access
- circe for JSON
- fs2 for streaming
- munit / weaver for testing

## Content Conventions

- **200-500 words per file.** Short, concise, direct.
- **Code examples must compile** against Scala 3 unless noted otherwise.
- Use ``, ``, `` level badges.
- No emojis in content.
- Every file is a self-contained lesson. Files reference other files by path.
- Use step-by-step explanations with code blocks.
- Prefer showing the "why" before the "how."

## Effect Systems (Module 02)

This module is the conceptual core of the roadmap. When editing or extending:

- Always explain WHY effects matter before showing HOW to use them
- Distinguish between side effects (uncontrolled) and encoded effects (typed, composable)
- Show the mental model shift: "a value that describes a computation" vs "a computation that runs"
- Compare with unchecked alternatives (exceptions, nulls, callbacks) to motivate the pattern

## When Adding Content

- Follow the existing directory numbering scheme (XX-topic/NN-name.md)
- Keep files self-contained but cross-reference related modules
- Match the tone: direct, no fluff, code-first with explanation
- Add level badges to new files
- Test any code examples mentally for Scala 3 compatibility

## File References

- `README.md` — Navigation and objectives
- `AGENTS.md` — This file
- `00-foundations/` — Programming and Scala fundamentals
- `01-first-code/` — Setup, syntax, core language features
- `02-effect-systems/` — Cats Effect, ZIO, typed error handling
- `03-http-apis/` — http4s, Akka HTTP, JSON, middleware
- `04-database/` — Doobie, Slick, migrations
- `05-concurrency/` — Fibers, streaming, supervision
- `06-production/` — Testing, observability, deployment
- `07-capstone/` — Hands-on project placeholder
