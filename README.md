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
