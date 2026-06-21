# ADR-0002: Explicit `total fn` Annotation Policy

**Status:** Accepted
**Date:** 2026-06-21

## Context

MVL infers totality for functions that have no unbounded loops and no calls to `partial fn`. These show up as `total*` (implicit) in `mvl assurance`. The question is: should pkg-http rely on inference, or annotate explicitly?

## Decision

**Every function in pkg-http that is total must carry the explicit `total fn` keyword.** No implicit totality (`total*`) is permitted in source files.

## Rationale

- Explicit annotation is a contract: the author claims termination and exhaustiveness, the checker verifies it.
- Implicit totality is a silent default that can be broken by adding a call to a `partial fn` without realising the impact. With an explicit annotation, the compiler catches it.
- `partial fn` is already explicit; totality should be equally explicit.
- `mvl assurance` reports `N total fn (N explicit, 0 implicit)` — `0 implicit` is the gate.

## Application

| Function kind | Keyword |
|---|---|
| Pure constructors, builders | `total fn` |
| Exhaustive enum match | `total fn` |
| For/while loops with `decreases` | `total fn` |
| Calls only `total fn` callees | `total fn` |
| Calls `partial fn` or has unbounded loops | `partial fn` |
| Actor methods | no keyword (actor scheduling is partial by nature) |

Effect annotations are orthogonal: `pub total fn serve_dir(...) -> Response ! FileRead` is valid.

## What `partial fn` legitimately covers in pkg-http

- `accept_loop`, `serve` — infinite TCP accept loop (intentional non-termination)
- `parse_response`, `expect_*` in `testing.mvl` — call `parse_rest` which is partial

## Consequences

- `make assurance` reports `0 implicit total` — use this as the CI gate.
- Any new function added without a totality keyword fails the policy.
- Code reviewers reject PRs that introduce implicit totality.

## Connected to

- MVL Req 3 (Totality)
- MVL Req 8 (Termination)
