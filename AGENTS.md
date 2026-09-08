# Agent and contributor contract

- Prefer `mncs-language` for implementation and examples.
- Optimize for clear application code and excellent diagnostics.
- Keep command schemas machine-inspectable and reusable for help/completion/testing.
- Treat stdout, stderr, exit status and terminal behavior as explicit contracts.
- Record language/compiler/runtime friction in `docs/LANGUAGE_PRESSURES.md` with minimal examples.
- Avoid hidden process-global state where explicit context can work.
- Test Unix pipelines, non-interactive execution and terminal interaction separately.
