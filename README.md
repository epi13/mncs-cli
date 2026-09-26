# mncs-cli

The canonical MNCS-native command-line interface layer: typed command
descriptors, argument parsing, routing, human/machine formatting, and
exit-code mapping over the persistent MNCS system.

`mncs-cli` is a **projection, not an authority**. It owns parsing,
validation at the interface boundary, routing, formatting, and exit
codes. It does not own execution policy, test semantics, diagnosis,
persistence, lineage, planning, or language semantics — those belong to
Forge, mncs-test, Debug, Doctor, Store, and the other canonical owners.
Commands that name those subsystems return **delegation records**; the
CLI reports where a request routes and refuses to pretend it executed.

## What the CLI owns

- command parsing against declarative descriptors (`src/cli/parse.mncs`)
- argument validation with structured errors (`ParseErrorKind`)
- identity resolution to canonical spellings (command/option/owner words)
- command routing to handlers or delegation records (`src/cli/app.mncs`)
- human formatting: help text, command index (`src/cli/render.mncs`)
- machine formatting: compact JSON descriptors, results, delegations
- exit-code mapping with PASS / FAIL / UNKNOWN preserved (`src/cli/outcome.mncs`)
- stdout/stderr separation: data to stdout, diagnostics to stderr

## What the CLI delegates

| Capability | Owner | CLI exposure |
|---|---|---|
| workflow execution | Forge | `forge` delegation record, never executes |
| test semantics | mncs-test | `test` delegation record, never runs tests |
| diagnosis | Debug | `debug` delegation record, never diagnoses |
| repository health | Doctor | `doctor` delegation record, never checks |
| command vocabulary | `mncs.cli.schema` descriptors | shared by parsing, help, schema, completion |

## Command surface

```
mncs help [--command NAME]     human help for one command, or the index
mncs schema --command NAME     machine-readable JSON descriptor
mncs version                   machine-readable CLI identity
mncs verdict --outcome WORD    project pass|fail|unknown to JSON plus exit code
mncs forge --action A [--target T]    delegate to Forge (exit 2, UNKNOWN)
mncs test --action A [--target T]     delegate to mncs-test (exit 2, UNKNOWN)
mncs debug --action A [--target T]    delegate to Debug (exit 2, UNKNOWN)
mncs doctor --action A [--target T]   delegate to Doctor (exit 2, UNKNOWN)
mncs commands                  list command names
```

One canonical tree lives in `mncs.cli.app.command_table`; help, JSON
schema, and the parser all consume the same descriptors, so they cannot
drift apart. There are no legacy aliases and no parallel command trees.

## Identity model

Commands, options, owners, and outcome words are bounded byte spellings
matched exactly (`token.name_matches`). Human selectors (`--command
verdict`) resolve to canonical descriptor identities (stable numeric
command codes 1–9); similar names never collapse. Values (targets,
actions) are carried opaquely up to the 64-byte token window; entries
wider than the window report `TruncatedToken` instead of matching a
prefix.

## Human / machine interfaces

- Human output (help, index, diagnostics) is concise data-forward text.
- Machine output is compact JSON with a stable shape per command
  (`schema`, `version`, `verdict`, delegation records).
- Structured output is the contract; table alignment may change freely.
- stdout carries data; stderr carries diagnostics. Usage errors emit an
  empty (zero-filled) stdout window and the diagnostic on stderr.
- Transport is a uniform 1024-byte window per stream; the significant
  prefix is the renderer's tracked length (see `P-CLI-001`).

## PASS / FAIL / UNKNOWN and exit codes

| Result | Exit | Meaning |
|---|---|---|
| PASS | 0 | command completed |
| FAIL | 1 | semantic failure, preserved exactly |
| UNKNOWN | 2 | incomplete/delegated, never folded into FAIL |
| invalid invocation | 64 | usage error, no semantic outcome |
| unavailable service | 69 | canonical dependency missing |
| internal CLI defect | 70 | bug in the CLI layer |

`verdict --outcome unknown` exits 2 with outcome UNKNOWN preserved;
delegation commands exit 2 for the same reason. Usage errors carry
`has_outcome = false`, so no caller can read them as PASS, FAIL, or
UNKNOWN.

## Agents

Prefer bounded machine queries: `schema --command NAME` for capability
discovery, `version` for identity, `verdict` for the outcome contract.
Parse stdout JSON; never scrape help text. Read commands never trigger
execution — and delegated commands never execute at all.

## Adding a command

1. Add the descriptor to `command_table` (name, summary, options,
   delegation metadata) with a stable code.
2. Add a handler branch in `run_command`; keep subsystem semantics out.
3. Add native tests in `tests/native/` covering parsing, routing, both
   output sides, and the exit mapping.
4. Help, JSON schema, and validation follow from the descriptor.

## What stays host-owned

argv acquisition, terminal I/O, process exit, signal handling, shell
completion integration, and terminal capability detection remain host
responsibilities. The MNCS side owns everything from the token window
inward. Subsystem invocation (Forge/Test/Debug/Doctor) stays host-routed
until those subsystems offer callable in-language APIs (`P-CLI-007`).

## Verification

```
python3 scripts/run_tests.py            # both native suites via `mncs test`
python3 scripts/run_tests.py --suites cliparse
python3 scripts/run_tests.py --suites cliapp
```

51 native assertions cover token classification, descriptor lookup,
all seven parse error kinds, renderer primitives (including overflow
poisoning), the full exit-code table, UNKNOWN preservation through
`verdict` and delegation, stdout/stderr separation, and byte-exact
output contracts for `version` and `verdict`.

## Pressures

Genuine language/tooling pressures found while building this layer are
recorded in `docs/LANGUAGE_PRESSURES.md` with minimal reproducers.
Doctor currently reports `DOC102: unknown profile 0.18` for every
`.mncs` file here; the toolchain compiles and runs profile 0.18 (as in
sibling campaigns), so this is Doctor's stale registry, not a source
defect.
