# mncs-cli

Batteries-included command-line application infrastructure for MNCS.

`mncs-cli` is a language-ergonomics pressure project: polished command-line programs should be straightforward to express in `mncs-language` while retaining typed arguments, explicit effects, structured errors and machine-readable command metadata.

## Initial scope

- commands, subcommands, options and positional arguments
- generated help, usage and shell completion
- typed parsing and validation
- prompts, progress, tables and terminal capabilities
- configuration/environment integration
- structured stdout/stderr and exit semantics
- terminal UI foundations
- deterministic CLI test harnesses

## Repository layout

- `docs/ARCHITECTURE.md`
- `docs/rfcs/0001-foundation.md`
- `docs/LANGUAGE_PRESSURES.md`
- `AGENTS.md`
