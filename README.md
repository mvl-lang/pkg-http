# pkg-http

HTTP server toolkit for [MVL](https://github.com/LAB271/mvl_language).

Pure MVL — no `bridge.rs`, no `extern` blocks. Fully verified against all 11 compiler requirements. Built on `std.net` TCP primitives.

## Install

```bash
mvl add github.com/mvl-lang/pkg-http v0.2.0
mvl install
```

## Usage

```mvl
use pkg.http.{Method, Request, Response, Router, MatchedRoute,
              parse_request, serialize_response, error_response,
              new_router, route, dispatch, ok, not_found}
use std.net.{tcp_listen, tcp_accept, tcp_read_request, tcp_write,
             tcp_close_stream, tcp_close_listener}

fn setup_router() -> Router {
    let r: Router = new_router();
    let r: Router = route(r, Method::Get, "/hello", "hello");
    route(r, Method::Get, "/users/{id}", "get_user")
}

partial fn handle(req: Request, matched: MatchedRoute) -> Response {
    match matched.name {
        "hello"    => ok("{\"message\": \"Hello, world!\"}"),
        "get_user" => ok("{\"id\": 1}"),
        _          => not_found(),
    }
}
```

### Static File Serving

```mvl
use pkg.http.{serve_dir, Request, Response, Router, MatchedRoute,
              new_router, route, dispatch, ok, not_found}

fn setup_router() -> Router {
    let r: Router = new_router();
    route(r, Method::Get, "/api/health", "health")
}

partial fn handle(req: Request, matched: MatchedRoute) -> Response ! FileRead {
    match matched.name {
        "health" => ok("{\"status\": \"ok\"}"),
        _        => not_found(),
    }
}

// In your dispatcher, fall through to serve_dir for unmatched routes:
partial fn dispatcher(req: Request) -> Response ! FileRead {
    let r: Router = setup_router();
    match dispatch(r, req) {
        Ok(matched) => handle(req, matched),
        Err(_)      => if req.path.starts_with("/docs") {
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

`serve_dir` splits the request path on `/` and rejects any segment that is `..`. This prevents directory traversal attacks before any filesystem access occurs.

```
GET /docs/../etc/passwd  → 403 Forbidden  (blocked by has_traversal check)
GET /docs/../../secret   → 403 Forbidden
GET /docs/..             → 403 Forbidden
```

### IFC Integration

File contents read from disk are returned as `Tainted[String]` by `std.io.read_file`. Before including them in the HTTP response, `serve_dir` detaints with an explicit audit tag:

```mvl
let content: String = relabel trust(raw, "HTTP-STATIC-FILE");
```

This is safe because the path has already been validated — `has_traversal` confirms it stays within the served directory. The audit tag `"HTTP-STATIC-FILE"` makes this trust boundary visible in security reviews.

### Effect Requirement

`serve_dir` requires `! FileRead` — the compiler enforces that only functions declaring this effect can serve static files. Pure handler functions cannot accidentally access the filesystem.

## Modules

### `http.mvl` — Core

Types, parsing, routing, and response building.

| Category | Items |
|----------|-------|
| **Types** | `Status`, `Method`, `Request`, `Response`, `HttpError`, `Router`, `MatchedRoute`, `Route` |
| **Parsing** | `parse_request`, `parse_method`, `query_first`, `query_all`, `header` |
| **Response** | `respond`, `ok`, `created`, `not_found`, `error_response`, `serialize_response` |
| **Routing** | `new_router`, `route`, `dispatch` |
| **Server** | `ConnectionHandler` (actor), `Dispatcher` (fn type) |
| **Static Files** | `serve_dir`, `mime_type` |

### `rest.mvl` — JSON REST Helpers

Convenience functions for JSON APIs.

| Category | Items |
|----------|-------|
| **Responses** | `json_ok`, `json_created`, `json_no_content`, `json_error`, `json_ok_str`, `json_created_str` |
| **Params** | `param_string`, `param_int` |
| **Body** | `body_json`, `body_obj` |
| **JSON** | `json_field_string`, `json_field_int`, `json_get_object`, `json_escape`, `json_str` |
| **Errors** | `http_not_found`, `http_bad_request`, `http_unauthorized`, `http_internal_error` |

### `testing.mvl` — Test Helpers

Build mock requests for unit testing handlers.

## Key Design Decisions

- **IFC-aware**: `parse_request` returns `Tainted[String]` for user-controlled fields (query params). Handlers must explicitly `relabel trust(...)` before using them.
- **Pure routing**: `new_router`, `route`, and `dispatch` are pure functions with no effects. Side effects only happen at the network boundary.
- **Pattern matching**: Routes support `{param}` placeholders (e.g. `/users/{id}`). Matched parameters are available via `param_string` and `param_int`.

## Architecture

```
pkg-http/
├── mvl.toml              # package manifest (no [native] — pure MVL)
└── src/
    ├── http.mvl              # core types, parsing, routing
    ├── rest.mvl              # JSON REST helpers
    ├── testing.mvl           # test request builders
    ├── http_test.mvl         # core tests
    ├── http_server_test.mvl  # server integration tests
    └── testing_test.mvl      # testing helper tests
```

## Effects

Parsing and routing functions are **pure** (no effects). Server actors require `! Net + Console`.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
