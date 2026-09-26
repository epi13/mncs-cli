# Architecture

## Layers (realized)

1. **Command schema** (`src/cli/schema.mncs`) — `CommandSpec` /
   `OptionSpec` records: names, kinds (flag/value), required flags,
   delegation metadata (owner), bounded summaries. Constructors
   (`make_command`, `make_option`) pad explicit byte spellings into the
   fixed windows. Help, JSON schema, and the parser consume the same
   descriptor.
2. **Tokens** (`src/cli/token.mncs`) — the 64-byte token window over
   argv entries: long-flag/assignment classification, `--name=value`
   splitting, dash stripping, exact name matching, truncation flags.
   No string literals exist in MNCS, so spellings are explicit numeric
   bytes built with `pack8` (pressure `P-CLI-002`).
3. **Parsing** (`src/cli/parse.mncs`) — total fold of at most 8 argument
   tokens into `ParseResult`: per-option values, positionals, and one
   of seven structured error kinds with the offending token attached.
   Numeric `error_code` projections serve hosts and tests (no enum
   equality, `P-CLI-004`).
4. **Outcome** (`src/cli/outcome.mncs`) — `Outcome` (Pass/Fail/Unknown)
   travels beside `ExitClass` and the `ApplicationExit` transport
   value. `class_for_outcome` maps semantics to codes 0/1/2;
   usage/unavailable/internal failures carry `has_outcome = false`.
5. **Presentation** (`src/cli/render.mncs`) — `OutBuf`: a 1024-byte
   exact window plus live length and a poisoning `ok` flag. Human help
   and compact JSON renderers compose `put`/`put_chunk8`/`put_bytes`/
   `put_name`/`put_u64` primitives. `finish` coerces to the transport
   view; the significant prefix is `buf.len` (`P-CLI-001`).
6. **Application tree** (`src/cli/app.mncs`) — the reference command
   table (codes 1–9), handlers, diagnostics, and `dispatch`. Delegated
   commands (`forge`, `test`, `debug`, `doctor`) return delegation
   records with Outcome Unknown; the dispatcher never executes them.
7. **Testing** (`tests/native/`, `scripts/run_tests.py`) — `cliparse`
   (30 unit assertions) and `cliapp` (21 dispatcher assertions) through
   canonical `mncs test`; evidence envelopes carry run identity.

## Data flow

```
argv tokens -> token window -> find_command -> parse_args
  -> ok: run_command -> render OutBufs -> to_exit -> CommandExit
  -> err: parse_error_text -> usage diagnostic -> CommandExit (64)
```

`CommandExit { has_outcome, outcome, class, exit: ApplicationExit }`
keeps semantic result, process mapping, and transport separate.

## Key decisions

- Delegation over duplication: subsystem commands are descriptors with
  owner metadata, not subprocess orchestration in MNCS.
- Numeric spellings over strings: the language has no string literals;
  the cost is explicit and recorded as pressure, not worked around
  with a parallel text system.
- Exact buffers inside, views at the boundary: all mutation targets
  exact arrays (`copy_span`/`replace` refuse views, `P-CLI-003`);
  `up_to` views appear only at ingestion (`from_entry`) and transport
  (`finish`).
- `match` over equality for enums; numeric codes (`error_code`,
  `outcome_code`, `class_code`) as the stable host/test vocabulary.
- Empty transport sides are zero-filled projections, never length-zero
  literals: indexing stays total on every path.
