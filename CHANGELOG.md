# Changelog

All notable changes to pkg-http will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.2.0] - 2026-05-29

### Added
- `serve_dir(prefix, dir_path, req)` — serve static files from a directory under a URL prefix (#999)
- `mime_type(ext)` — MIME type lookup for html, css, js, json, png, svg, ico, txt, woff2, woff (unknown → `application/octet-stream`)
- Path traversal protection: `..` segments in request paths are rejected with 403
- Auto-serve `index.html` for directory paths (empty or trailing `/`)
- IFC-safe: file contents detainted with audit tag `"HTTP-STATIC-FILE"` after path validation
- 27 new tests for file_extension, has_traversal, and mime_type

## [0.1.2] - 2026-05-29

### Added
- CHANGELOG.md tracking all releases

## [0.1.1] - 2026-05-29

### Added
- Apache License 2.0
- CONTRIBUTING.md with setup, code style, and testing instructions
- README.md with install, usage example, module reference, and architecture

## [0.1.0] - 2026-05-29

### Added
- Initial release — pure MVL HTTP server toolkit
- Core types: `Status`, `Method`, `Request`, `Response`, `HttpError`, `Router`, `MatchedRoute`, `Route`
- Request parsing: `parse_request`, `parse_method`, `query_first`, `query_all`, `header`
- Response building: `respond`, `ok`, `created`, `not_found`, `error_response`, `serialize_response`
- Pattern-matching router: `new_router`, `route`, `dispatch` with `{param}` placeholders
- JSON REST helpers: `json_ok`, `json_created`, `json_no_content`, `json_error`, `param_string`, `param_int`, `body_json`, `body_obj`
- Test helpers: mock request builders for unit testing handlers
- IFC-aware: `parse_request` returns `Tainted[String]` for user-controlled fields
- Server actor: `ConnectionHandler` with `Dispatcher` function type
- Built on `std.net` TCP primitives — no `bridge.rs`, no `extern` blocks
