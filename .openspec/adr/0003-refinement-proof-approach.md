# ADR-0003: Refinement Proof Approach — Computed Values, Not Constant Wrappers

**Status:** Accepted
**Date:** 2026-06-18
**Context:** MVL's `where` constraints on function parameters create refinement proof obligations at call sites, visible in `mvl prove`. The question is what constitutes a meaningful proof vs proof theater.

## Decision

**Refinement proofs must be over computed or user-supplied values where the constraint asserts real correctness. Wrapping literal constants in a validation function to produce trivial proof counts is explicitly rejected.**

### What is meaningful

```mvl
// byte_value proves that 2-byte UTF-8 codepoint arithmetic stays in [0, 255]
fn byte_value(n: Int where n >= 0 && n <= 255) -> Byte { from_int(n) }

// Called with a computed value guarded by an explicit branch condition:
if cp >= 0 && cp <= 255 {
    out = out.concat(String::from_bytes([byte_value(cp)]))
}
// → mvl prove reports: utf8_from_bytes → byte_value(n) (L1:trivial)
// The proof is meaningful: cp is the result of (b0-192)*64 + (b1-128),
// and the branch condition is the evidence for the constraint.
```

The `validate_port_range(user_port)` pattern in `examples/crud_api` is the canonical example: a user-supplied value that genuinely might be out of range.

### What is not meaningful

```mvl
// REJECTED: wrapping a known constant adds zero information
fn http_status_valid(code: Int where code >= 100 && code < 600) -> Int { code }
match s {
    Status::Http200Ok => http_status_valid(200),  // trivially true, adds nothing
    ...
}
```

The constant `200` satisfies `>= 100 && < 600` — any reader can see this. The prover calling it "L1:trivial" is accurate: it is trivially true AND trivially uninteresting.

### Static return type annotations are fine

```mvl
// Acceptable: declares the invariant for callers without manufacturing proof theater
pub total fn status_code(s: val Status) -> Int where self >= 100 && self < 600 {
    match s { Status::Http200Ok => 200, ... }
}
```

The return-type refinement documents the contract. Callers that need to prove the constraint can pattern-match on the result; they don't need a wrapper call.

## Target state

`mvl prove` output for pkg-http should show only proofs over computed values. The current baseline:

```
1 proven (L1:1), 0 runtime, 0 failed
  utf8_from_bytes → byte_value(n) — `self >= 0 && self <= 255`  (L1:trivial)
```

Future proofs should follow the same pattern: a helper with a `where` constraint called with a value whose bounds are established by branch conditions or type information, not by inspecting a literal.

## Consequences

- `mvl prove` proof count is a quality signal only when proofs are over non-trivial values.
- New `where`-constrained helpers should be introduced only when there is a genuine arithmetic invariant to document.
- Code reviewers should question any `where`-constrained function called exclusively with literals.

## Connected to

- MVL Req 10 (Refinements): `mvl assurance` REQ10 counts proven call sites
- ADR-0002: `total fn` annotation policy — `byte_value` is `total fn`
- Issue #6: std.strings hex utilities — when `hex_char_value` lands with a refined return type `Option[Int where self >= 0 && self <= 15]`, the `hex_digit` mirror can be replaced and its call sites become provable
