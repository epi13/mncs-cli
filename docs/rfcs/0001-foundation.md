# RFC 0001: CLI foundation

Status: Implemented (2026-09-26: descriptor-driven parser, router, renderers, exit mapping, and native suites landed)

## Principles

- Command definitions are typed data, not opaque parser callbacks.
- Help and completion derive from the same schema used for parsing.
- Parse failures are structured and diagnostic-first.
- Non-interactive/pipeline behavior remains correct even when rich terminal output exists.
- Environment, filesystem, stdin/stdout/stderr and process actions are explicit effects.
- Common CLI programs should require very little boilerplate.

## Pressure objectives

Named/default arguments, enums/sum types, derive-like schema metadata, reflection or compile-time generation, strings/Unicode, iterators, closures, error propagation, environment/process APIs, terminal capability handling and test-friendly I/O abstraction.
