# ADR-0003: Refinement Proofs — Over Computed Values, Not Constant Wrappers

**Status:** Accepted
**Date:** 2026-06-21

## Context

MVL's `where` constraints on function parameters create refinement proof obligations at call sites, visible in `mvl prove`. The question is what constitutes a meaningful proof vs proof theater.

## Decision

**Refinement proofs in pkg-http must be over computed or user-supplied values where the constraint asserts real correctness.** Wrapping literal constants in a validation function to manufacture trivial proof counts is rejected.

## What is meaningful

A `where` constraint on a parameter that may legitimately be out of range:

```mvl
total fn byte_value(n: Int where n >= 0 && n <= 255) -> Byte { from_int(n) }

// Called with a computed value guarded by a branch condition:
if cp >= 0 && cp <= 255 {
    out = out.concat(String::from_bytes([byte_value(cp)]))
}
```

`mvl prove` reports the call site as `L1:trivial`, and that label is accurate — the bound is provable from the branch condition, which is exactly the evidence we want recorded.

## What is not meaningful

```mvl
// REJECTED: wrapping a known constant adds zero information
fn http_status_valid(code: Int where code >= 100 && code < 600) -> Int { code }
match s {
    HttpStatus::Http200Ok => http_status_valid(200),  // trivially true, adds nothing
    ...
}
```

The literal `200` satisfies the constraint by inspection. The prover is correct that it's trivially true, but the proof carries no information a reader didn't already have.

## Return-type refinements are fine

```mvl
pub total fn status_code(s: val HttpStatus) -> Int where self >= 100 && self < 600 {
    match s { HttpStatus::Http200Ok => 200, ... }
}
```

The return-type refinement documents the invariant for callers. Callers who need the constraint at their site get it from the type system, not from a wrapper call.

## Consequences

- `mvl prove` proof count is a quality signal only when proofs are over non-trivial values.
- New `where`-constrained helpers should be introduced only when there is a genuine arithmetic invariant to document.
- Code reviewers question any `where`-constrained function called exclusively with literals.

## Connected to

- MVL Req 10 (Refinements)
- ADR-0002: explicit totality annotation
