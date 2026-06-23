---
tags:
  - bazantic
  - gateway
---

# JSON-RPC Proxy

The JSON-RPC proxy is a specialization of `proxy.Handler`which should be created in @internal/proxy/jsonrpc.go.  It's purpose is to inspect both the HTTP request entity body and HTTP response entity body to add observability signals and ultimately to properly handle JSON-RPC errors.

## Requirements

- The JSON-RPC proxy must retrieve the `Signals`from the HTTP request context
- The JSON-RPC proxy must 

## Implementation Notes

- The JSON-RPC proxy should be created using the 