# MNCS language pressure ledger

For each finding: the CLI workload, current behavior, desired behavior,
reproducer, owner, workaround, and closure test. All findings below were
hit while building the canonical CLI slice (`src/cli`, profile 0.18)
and are evidenced by compiler diagnostics or runtime probes from this
campaign. Standing target list from the bootstrap (declarative schemas,
typed parsing, structured errors, explicit effects, deterministic
command tests) is substantially realized; the entries below are what
remains genuinely missing or frictional.

## P-CLI-001: no length-preserving exact-to-view projection (workaround in place)

- Workload: projecting a rendered `OutBuf` (`[byte; 1024]` + live
  length) into the `ApplicationExit.stdout` transport view.
- Current: `finish` coerces the full 1024-byte window; the coerced view
  reads the whole window and the significant prefix is the separately
  tracked `len`. Hosts must slice `[..len]`.
- Desired: coerce an exact array to a view whose runtime length is the
  live prefix.
- Reproducer: `render.finish` in `src/cli/render.mncs`; probe showed a
  coerced `[byte; 1024]` reads length 1024 regardless of content.
- Owner: mncs-language (read-only this campaign).
- Workaround: `OutBuf.len` is the contract; native tests assert prefix
  content plus a zero byte at `len`.
- Closure: a projection primitive with a test showing `len == live`.
- Blocking: no (documented transport contract).

## P-CLI-002: no string or byte-string literals (workaround in place)

- Workload: spelling command names, option names, help words, JSON keys.
- Current: `"` is rejected at lexing (`MNL002: unsupported source
  character '"'`); every spelling is numeric bytes via `pack8` with a
  comment giving the intended text.
- Desired: byte-string literals projecting to bounded byte arrays.
- Reproducer: `/tmp/clispike/p4.mncs` (`let word: [byte; 4] = "help";`
  fails lexical analysis).
- Owner: mncs-language (read-only this campaign).
- Workaround: `token.pack8` + `schema.pad32`/`pad_summary`.
- Closure: `"help"` (or equivalent) accepted where `[byte; N]` or a
  bounded view is expected.
- Blocking: no (verbose but explicit).

## P-CLI-003: span copy and functional update refuse views (designed around)

- Workload: building tokens and output buffers from argv/views.
- Current: `copy_span` requires an exact destination (`MNE265`);
  `replace` requires an exact base (`MNE198`).
- Desired: either view-targeting variants or a documented rule that
  views are read-only projections (the current de-facto rule).
- Reproducer: `/tmp/clispike/copy.mncs` and `p3.mncs` probes.
- Owner: mncs-language (read-only this campaign).
- Workaround: all mutation targets exact arrays; views only at
  ingestion (`from_entry`) and transport (`finish`).
- Closure: compiler documentation or view-accepting primitives.
- Blocking: no.

## P-CLI-004: no enum equality (designed around)

- Workload: branching on `OptionKind`, comparing parse error kinds and
  outcomes in code and tests.
- Current: `==` on enums is rejected (`MNE121: comparison operands
  must have an integer type`); decisions go through `match`.
- Desired: derived equality for field-less enums, or keep `match` as
  the idiom and document it.
- Reproducer: `entry.kind == OptionKind.Value` in `src/cli/parse.mncs`.
- Owner: mncs-language (read-only this campaign).
- Workaround: `schema.option_takes_value` plus numeric projections
  (`parse.error_code`, `outcome.outcome_code`, `outcome.class_code`)
  as the stable host/test comparison vocabulary.
- Closure: decision either way; projections stay useful regardless.
- Blocking: no.

## P-CLI-005: no nested field projection (designed around)

- Workload: reading `result.exit.code`-style nested values.
- Current: `a.b.c` does not parse; intermediate lets are required.
- Desired: nested projection, or document single-level as the idiom.
- Reproducer: `parts.name.len` in `tests/native/cliparse.mncs`.
- Owner: mncs-language (read-only this campaign).
- Workaround: bind each level (`let pname: Token = parts.name;`).
- Closure: parse support or documented idiom.
- Blocking: no.

## P-CLI-006: reserved words collide with natural bindings (designed around)

- Workload: ordinary local names in folds and diagnostics.
- Current: `match` and `over` are rejected as binding names
  (`MNP050` cascade; `over` in carry position).
- Desired: a documented reserved-word list near the grammar.
- Reproducer: `let match: bool` in `src/cli/schema.mncs`;
  `let over: OutBuf` in `tests/native/cliparse.mncs`.
- Owner: mncs-language (read-only this campaign).
- Workaround: `is_match`, `extra`.
- Closure: reserved-word documentation.
- Blocking: no.

## P-CLI-007: subsystems are not callable from MNCS (architectural boundary)

- Workload: the operator-CLI vision — `forge`/`test`/`debug`/`doctor`
  commands that actually route through canonical owners.
- Current: no typed in-language invocation of Forge, mncs-test,
  Debug, or Doctor exists; the CLI returns delegation records
  (`routing: "delegate"`, Outcome Unknown, exit 2) and the host
  launcher routes them.
- Desired: callable subsystem APIs so delegation records can carry
  real execution/result identities instead of routing metadata alone.
- Reproducer: `handle_delegate` in `src/cli/app.mncs` — the honest
  boundary; anything more would be shell-orchestration in-language.
- Owner: Forge / Test / Debug / Doctor + language effects (all
  read-only this campaign).
- Workaround: delegation records with owner/action/target/route.
- Closure: an in-language `forge --action` equivalent returning a
  canonical result identity.
- Blocking: for full operator-CLI execution, yes; for the CLI
  framework and projection contract delivered here, no.

## P-CLI-008: sequence literals need typed lets (ergonomic friction)

- Workload: inline byte arrays as call arguments.
- Current: `MNE183` unless the literal sits under an exact or
  bounded-view expected type; generic parameters (`[byte; N]`) do not
  propagate inward, so callers bind a typed `let` first.
- Desired: generic-argument inference for sequence literals.
- Reproducer: first `/tmp/clispike/spike.mncs` revision.
- Owner: mncs-language (read-only this campaign).
- Workaround: typed lets before calls.
- Closure: inference or a targeted diagnostic suggestion.
- Blocking: no.

## External gaps observed (not language pressures)

- Doctor reports `DOC102: unknown profile 0.18` for all 8 `.mncs`
  files here, while the toolchain compiles and runs profile 0.18
  (same as sibling campaigns). Doctor's profile registry is stale;
  recorded here, fix belongs to mncs-doctor (read-only).
- Debug `import-test` cannot resolve out-of-tree suite sources
  (derives `target/tests/native/...` under the CLI root). Native
  results import unchanged for in-tree layouts; noted for mncs-debug
  (read-only).
