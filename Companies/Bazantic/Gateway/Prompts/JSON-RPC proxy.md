---
tags:
  - bazantic
  - gateway
---

# JSON-RPC Proxy

The JSON-RPC proxy is a specialization of `proxy.Handler`which should be created in @internal/proxy/jsonrpc.go.  It's purpose is to inspect both the HTTP request entity body and HTTP response entity body to add observability signals and ultimately to properly handle JSON-RPC errors.

## Requirements

- The JSON-RPC proxy must retrieve the `Signals`from the HTTP request context
- The JSON-RPC proxy must decode the JSON-RPC request from the HTTP request entity body and:
	- Must set `handler.Signals.Operation` to the value of the JSON-RPC `method` field.
	- Should sanity check that the `jsonrpc` field is set to "2.0".
- The JSON-RPC proxy must decode the JSON-RPC response from the HTTP response entity body and:
	- Must copy the value of the `error.code`field (if it exists) to `handler.Signals.Code`.
	- Must copy the value of the `error.message` field (if it exists) to `handler.Signals.Reason
- The JSON-RPC proxy must delegate the actual HTTP proxy function to `proxy.Handler.

## Implementation Notes

- The JSON-RPC proxy should be created using the `NewJSONRPCHandler` function, which should have the same signature as the `NewHandler` function.
- The JSON-RPC proxy should have the type `handler.JSONRPCHandler`.
- The JSON-RPC proxy should embed an `proxy.Handler` which can be constructed using the passed arguments.

**Write a spec that fully describes the behavior of this feature - feel free to grill me to resolve omissions and/or ambiguities.**