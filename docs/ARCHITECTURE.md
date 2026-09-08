# Architecture

## Layers

1. **Command schema** — names, nesting, arguments, options, defaults, validation and metadata.
2. **Parsing** — argv/token interpretation and typed conversion.
3. **Execution** — command dispatch, structured context, effects and exit/error mapping.
4. **Presentation** — help, completion, prompts, progress, tables, color and terminal capabilities.
5. **Configuration** — environment, files and precedence rules.
6. **Testing/tooling** — synthetic argv/stdin/terminal, snapshots and schema inspection.

## First milestones

1. Typed command/subcommand parser.
2. Generated help and structured errors.
3. Environment/configuration integration.
4. Completion and shell-safe output.
5. Interactive prompts/progress and basic terminal UI.
