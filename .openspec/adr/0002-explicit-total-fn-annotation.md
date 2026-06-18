# ADR-0002: Explicit `total fn` Annotation Policy

**Status:** Accepted
**Date:** 2026-06-18
**Context:** MVL infers totality for functions that have no unbounded loops and no calls to `partial fn`. These show up as `total*` (implicit) in `mvl assurance`. The question is: should pkg-http rely on inference or annotate explicitly?

## Decision

**All functions that are total must carry the explicit `total fn` keyword.** No implicit totality (`total*`) is permitted in source files.

Rationale:
- `mvl assurance` reports `52 total fn (52 explicit, 0 implicit)` — this is the target state.
- Explicit annotation is a contract: the author claims termination and exhaustiveness, and the checker verifies it. Implicit totality is a silent default that can be broken by adding a call to a `partial fn` without realising the impact.
- `total fn` on a function that calls `partial fn` is a compile error — this is the intended safety net.
- `partial fn` is already explicit; totality should be equally explicit.

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

- `accept_loop`, `serve` — infinite TCP accept loop
- `json_ok`, `json_created`, `json_error` — call `std.json.encode` which is `partial fn`
- `body_json`, `body_obj` — call `std.json.decode` which is `partial fn`
- `parse_response`, `parse_rest`, `expect_*` — in `testing.mvl`, call the above

## Consequences

- `make assurance` will report `0 implicit total` — use this as the gate.
- Any new function added without a totality keyword will appear as `total*` and fail the policy check.
- Reviewers should reject PRs that introduce implicit totality.

## Connected to

- MVL Req 3 (Totality): `mvl assurance` REQ3 verifies this
- MVL Req 8 (Termination): `total fn` fns must have provable termination
- ADR-0001: Mirror functions in test files intentionally omit `total` (they are not verified separately by the assurance tool)
