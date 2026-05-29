# Contributing to pkg-http

Thank you for your interest in contributing to the MVL HTTP package.

## Getting Started

1. Fork this repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Make your changes
4. Run the type checker: `mvl check src/http.mvl`
5. Run tests: `mvl test src/http_test.mvl`
6. Commit with a conventional message: `git commit -m "feat: add ..."`
7. Push and open a pull request

## Development Setup

You need the [MVL compiler](https://github.com/LAB271/mvl_language) installed:

```bash
# Build from source
git clone https://github.com/LAB271/mvl_language.git
cd mvl_language
cargo build
```

## Code Style

- Follow the MVL syntax conventions (see the [MVL cheat sheet](https://github.com/LAB271/mvl_language/blob/main/CLAUDE.md))
- All public functions must have doc comments (`///`)
- All functions must declare their effects
- Use `total fn` where possible; `partial fn` only when unavoidable
- This package is pure MVL — no `bridge.rs` or `extern` blocks

## Testing

Tests live in `src/http_test.mvl`, `src/http_server_test.mvl`, and `src/testing_test.mvl`.

```bash
mvl check src/http.mvl           # type-check
mvl test src/http_test.mvl       # run tests
```

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
