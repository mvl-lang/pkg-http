# ADR-0001: Package Scope — Raw HTTP Only

**Status:** Accepted
**Date:** 2026-06-21

## Context

The HTTP ecosystem in MVL spans several concerns:

- Wire-level protocol parsing/serialization (HTTP/1.0 request lines, headers, status codes)
- Routing and dispatch
- JSON request/response helpers (typed serialization, body parsing)
- REST client (outbound calls over HTTPS)
- OpenAPI generation, schema introspection

Without clear boundaries, a single package would accrete all of these and become a kitchen sink.

## Decision

**pkg-http stays the raw HTTP layer.** It owns:

| In scope | Out of scope |
|----------|--------------|
| `HttpStatus`, `HttpMethod` enums | Typed JSON request/response builders |
| `Request`, `Response`, `HttpError` | REST client (outbound HTTPS) |
| `parse_request`, `serialize_response` | OpenAPI generation |
| `Router`, `route`, `dispatch` | Authentication, sessions, middleware |
| `serve_dir`, `mime_type` (static file serving) | Anything requiring `std.json` |
| Path/query parsing primitives | |

Typed JSON helpers (`json_ok`, `body_json`, `param_int`, etc.) live in **pkg-rest**, which depends on pkg-http for raw types.

## Layering

```
pkg-rest         ← typed REST: client + server JSON helpers, OpenAPI
   ↓ depends on
pkg-http         ← raw HTTP protocol, routing, static files
   ↓ depends on
std.net          ← TCP, UDP primitives
```

A package one layer up may depend on the layer below. Skipping is allowed (pkg-rest may use `std.net` directly), but reverse dependencies are forbidden.

## Rationale

- **Single responsibility per package.** pkg-http should answer "what is HTTP?", not "what is a JSON REST API?"
- **No `std.json` dependency in pkg-http.** Keeps the raw layer free of serialization concerns. A consumer who wants to serve HTML, plain text, or protobuf over HTTP shouldn't pay for `std.json` parser code.
- **Easier to reason about effects.** pkg-http functions are mostly `total fn` over pure data. JSON parsing is `partial fn` (decode can fail), so it lives in pkg-rest where partiality is expected.

## Consequences

- `rest.mvl` (the JSON helpers) has been removed from pkg-http and moved to pkg-rest's server module.
- pkg-http v0.4.0 is a breaking change: consumers must update imports from `pkg.http.{json_ok, body_json, ...}` to `pkg.rest.server.{json_ok, body_json, ...}`.
- New JSON or REST helpers should land in pkg-rest, not here.

## Connected to

- pkg-rest issue #3 — typed server-side routes
