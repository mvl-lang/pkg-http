# ADR-0001: Test File Mirror Pattern — Workaround for Issue #96

**Status:** Accepted (workaround — superseded when #96 is resolved)
**Date:** 2026-06-18
**Context:** MVL does not currently support module-level imports in test files; test files cannot `use` the same package they live in. This means `http_test.mvl` cannot write `use pkg.http.{Status, percent_decode, ...}` — it must redeclare those types and helper functions locally. This is tracked as issue #96 ("inter-package import in test context").

## Decision

Each source file `foo.mvl` has a mirror test file `foo_test.mvl`. The test file:

1. **Re-declares all types** used by its tests verbatim (same variants and field names as the source).
2. **Re-declares private helper functions** that are not exported but are called directly in tests (`hex_digit`, `percent_decode`, `utf8_from_bytes`, `flush_bytes`).
3. **Does NOT re-declare public functions** that are tested via the normal call path.

The mirror copies must stay in sync with the source definitions. Divergence is a latent bug.

## Keeping mirrors in sync

Rules enforced by code review:
- Any change to a re-declared type or helper in `foo.mvl` must also update `foo_test.mvl`.
- Re-declared helpers carry no `total` keyword (they are not verified separately).
- Refinement constraints (`where n >= 0 && n <= 255`) ARE mirrored so that proof obligations exist in both files.

The `make coverage` report indirectly surfaces divergence: if the source and mirror implement different branch conditions, coverage percentages will differ between the two coverage groups.

## When #96 is resolved

Replace all re-declarations in `*_test.mvl` with a single `use pkg.http.{...}` import. The type re-declaration blocks and helper copies can then be deleted. No behaviour changes.

## Consequences

- All `*_test.mvl` files contain a boilerplate section that must be kept manually.
- Changes to types or private helpers require a two-file diff.
- Coverage tool reports both the source and the mirror separately; the mirror is noise.
- `make check` does not catch divergence — only test failures do.

## Connected to

- Issue #96: Module-level import in test context
- ADR-0002: Explicit `total fn` annotation — mirror functions intentionally omit `total`
- Issue #6: std.strings hex utilities (when added, `hex_digit` mirror can be deleted)
