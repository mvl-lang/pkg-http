# Contributing to pkg-http

Thank you for your interest in contributing to the MVL HTTP package.

## Getting Started

1. Fork this repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Make your changes
4. Run the type checker: `make check`
5. Run tests: `make test`
6. Commit with a conventional message: `git commit -m "feat: add ..."`
7. Push and open a pull request

## Development Setup

You need the [MVL compiler](https://github.com/mvl-lang/mvl) installed:

```bash
git clone https://github.com/mvl-lang/mvl.git
cd mvl
cargo build --release
```

Add the resulting binary to your `PATH` (or use `make` targets from this repo, which auto-detect the local build).

## Code Style

- Follow the MVL syntax conventions (see the [MVL language docs](https://github.com/mvl-lang/mvl/blob/main/docs/language.md))
- All public functions must have doc comments (`///`)
- All functions must declare their effects
- Use `total fn` where possible; `partial fn` only when unavoidable (ADR-0002)
- Refinement constraints should be over computed/user-supplied values, not literal wrappers (ADR-0003)
- This package is pure MVL — no `bridge.rs` or `extern` blocks
- pkg-http scope is **raw HTTP only** — JSON helpers belong in pkg-rest (ADR-0001)

## Testing

```bash
make check        # type-check all source files
make test         # run tests
make assurance    # full assurance report (totality, effects, refinements)
make coverage     # branch coverage
```

## Architectural Decisions

See `.openspec/adr/` for the architectural decision records:

- ADR-0001: Package Scope — Raw HTTP Only
- ADR-0002: Explicit `total fn` Annotation Policy
- ADR-0003: Refinement Proofs — Over Computed Values

Read these before proposing structural changes.

## Reporting Issues

File issues at https://github.com/mvl-lang/pkg-http/issues. For transpiler or language bugs, file at https://github.com/mvl-lang/mvl/issues.

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
