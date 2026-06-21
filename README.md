# pkg-http

Raw HTTP/1.0 server toolkit for [MVL](https://github.com/mvl-lang/mvl).

Pure MVL — no `bridge.rs`, no `extern` blocks. Built on `std.net` TCP primitives.

**Scope:** raw HTTP protocol, routing, and static file serving. Typed JSON request/response helpers and the REST client live in [pkg-rest](https://github.com/mvl-lang/pkg-rest) — see [ADR-0001](.openspec/adr/0001-package-scope.md).

## Install

```bash
mvl add github.com/mvl-lang/pkg-http v0.4.0
mvl install
```

## Usage

```mvl
use pkg.http.{HttpMethod, Request, Response, Router, MatchedRoute,
              parse_request, serialize_response, error_response,
              new_router, route, dispatch, ok, not_found}

total fn setup_router() -> Router {
    let r: Router = new_router();
    let r: Router = route(r, HttpMethod::Get, "/hello", "hello");
    route(r, HttpMethod::Get, "/users/{id}", "get_user")
}

partial fn handle(req: Request, matched: MatchedRoute) -> Response {
    match matched.name {
        "hello"    => ok("{\"message\": \"Hello, world!\"}"),
        "get_user" => ok("{\"id\": 1}"),
        _          => not_found(),
    }
}
```

For typed JSON responses, request body parsing, and REST helpers:

```mvl
use pkg.rest.json.{json_ok, json_error, body_json, param_int, http_not_found}
```

See [pkg-rest](https://github.com/mvl-lang/pkg-rest) for the full API.

## Static File Serving

```mvl
use pkg.http.{HttpMethod, serve_dir, Request, Response, Router, MatchedRoute,
              new_router, route, dispatch, ok, not_found}

total fn setup_router() -> Router {
    let r: Router = new_router();
    route(r, HttpMethod::Get, "/api/health", "health")
}

partial fn dispatcher(req: Request) -> Response ! FileRead {
    let r: Router = setup_router();
    match dispatch(r, req) {
        Ok(matched) => match matched.name {
            "health" => ok("{\"status\": \"ok\"}"),
            _        => not_found(),
        },
        Err(_) => if req.path.starts_with("/docs") {
            serve_dir("/docs", "./swagger-ui", req)
        } else {
            not_found()
        },
    }
}
```

`serve_dir("/docs", "./swagger-ui", req)` maps URL paths to filesystem paths:

| Request | File served |
|---------|-------------|
| `GET /docs/index.html` | `./swagger-ui/index.html` |
| `GET /docs/` | `./swagger-ui/index.html` (auto-index) |
| `GET /docs/css/style.css` | `./swagger-ui/css/style.css` |
| `GET /docs/missing.txt` | 404 |
| `GET /docs/../etc/passwd` | 403 (path traversal blocked) |

## Security

### Path Traversal Protection

`serve_dir` rejects any path segment that is `..`. Directory traversal is blocked before any filesystem access occurs.

### IFC Integration

`parse_request` returns `Tainted[String]` for user-controlled fields (query, headers, body). Handlers must explicitly `relabel trust(...)` with an audit tag before using them.

`serve_dir` detaints file contents with the audit tag `"HTTP-STATIC-FILE"` — visible in security reviews.

### Effect Requirement

`serve_dir` requires `! FileRead`. Pure handler functions cannot accidentally access the filesystem.

## Modules

### `http.mvl` — Core

Types, parsing, routing, and response building.

| Category | Items |
|----------|-------|
| **Types** | `HttpStatus`, `HttpMethod`, `Request`, `Response`, `HttpError`, `Router`, `MatchedRoute`, `Route` |
| **Parsing** | `parse_request`, `parse_method`, `query_first`, `query_all`, `header` |
| **Response** | `respond`, `ok`, `created`, `not_found`, `error_response`, `serialize_response` |
| **Routing** | `new_router`, `route`, `dispatch` |
| **Server** | `ConnectionHandler` (actor), `Dispatcher` (fn type), `serve` |
| **Static Files** | `serve_dir`, `mime_type` |

### `testing.mvl` — Test Helpers

Build mock requests and parse responses for unit testing handlers.

## Key Design Decisions

- **Raw HTTP only.** JSON helpers, REST client, and OpenAPI live in pkg-rest. See [ADR-0001](.openspec/adr/0001-package-scope.md).
- **IFC-aware.** All user-controlled input is `Tainted[String]`. Detaint via `relabel trust(...)` with an audit tag.
- **Pure routing.** `new_router`, `route`, and `dispatch` are pure. Side effects only at the network boundary.
- **Pattern matching routes.** `{param}` placeholders (e.g. `/users/{id}`). Matched params via `query_first` or pkg-rest's `param_int`.
- **Explicit totality.** Every function carries `total fn` or `partial fn`. See [ADR-0002](.openspec/adr/0002-explicit-total-fn-annotation.md).
- **Refinements over computed values.** See [ADR-0003](.openspec/adr/0003-refinement-proof-approach.md).

## Architecture

```
pkg-http/
├── mvl.toml              # package manifest (pure MVL, no [native])
├── .openspec/adr/        # architectural decisions
└── src/
    ├── http.mvl                # core types, parsing, routing, server
    ├── testing.mvl             # test request builders
    ├── http_server_test.mvl    # server integration tests
    └── testing_test.mvl        # testing helper tests
```

```
pkg-rest         ← typed REST: JSON helpers, REST client, OpenAPI
   ↓ depends on
pkg-http         ← raw HTTP protocol, routing, static files (this package)
   ↓ depends on
std.net          ← TCP primitives
```

## Effects

Parsing and routing functions are **pure** (no effects). Server actors require `! Net + Console`. Static file serving requires `! FileRead`.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache License 2.0 — see [LICENSE](LICENSE).
