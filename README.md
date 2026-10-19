# sdk-conformance-specs

Language-neutral test vectors that every weft SDK (Go, Python, Ruby) must pass. They pin the byte-level contracts the SDKs implement separately: the log line header, `traceparent`, the weftd socket frame, the reserved keys in heartbeat and runtime records, service-name validation, and gRPC trace metadata.

## Files

| File | Contract |
| --- | --- |
| `header.json` | line header v1 encoding: `input` fields plus `body` to the exact `expect` line (no trailing newline) |
| `traceparent.json` | `parse` (header to `{trace_id, span_id}` or `null`; an all-zero span id parses as `""`) and `format` |
| `socket_frame.json` | `{app, line, trace_id}` to `expect_frame` (with its trailing newline), or `expect_error` with nothing written |
| `reserved_keys.json` | heartbeat and runtime records built with caller extras: `expect` values, `present` keys, `absent_values` the caller must not win, `type_pattern`, and the `ts_pattern` every `ts` must match |
| `service_names.json` | `valid` and `invalid` service names |
| `grpc_metadata.json` | `outgoing` (ctx ids plus existing metadata to metadata pairs, order-insensitive) and `incoming` (metadata pairs to ids, or `null`) |

Control bytes are JSON `\u001e` / `\u001f` escapes. An input field that is absent means empty (`""`, or `0` for `dur_ns`). A header value containing `\x1e`, `\x1f` or `\n` is written empty.

## Runner contract

Each SDK vendors a copy of the files under `testdata/conformance/` and has a test that:

1. loads every `*.json` file, and fails on a file it has no runner for;
2. fails if a file it expects is missing;
3. checks each case against its own implementation.

Adding a vector file therefore breaks every SDK until it implements it. An SDK bumps its vendored copy deliberately, with `VERSION` recording the commit it was synced from.

## Vendoring

```
make conformance
```

copies `../sdk-conformance-specs/*.json` into `testdata/conformance/` and writes the source commit to `testdata/conformance/VERSION`.

## Adding a vector

Add a case to the existing file (or a new file with a runner in every SDK), run each SDK's suite against a synced copy, and commit here first. A vector that disagrees with an SDK is a bug in that SDK or a decision to make here; resolve it before syncing. Keep vectors free of anything specific to a product or deployment.
