---
tags:
  - bazantic
  - gateway
---

# Bazantic Universal API (BUA) Client

The Bazantic Universal API provides an OpenAPI specification that describes what end-points are available

## Requirements

1. A BUA client must be generated from the OpenAPI specification
2. The client must allow the caller to authenticate

## Implementation Notes
1. Add the `oapi-codegen` tool to the Go tools in `go.mod`
2. Add a `go:generate` directive to create `bua.Client`
3. The generated client will authenticate to the BUA using a `Bearer` token in the `Authorization` header.  The `NewClient`constructor must allow the value needed for this header to be optionally passed.
4. The OpenAPI specification is available at https://bazantic-registry.vercel.app/openapi.json
5. The OpenAPI specification is served with an ETag which should be copied to @internale/bua/data/openapi.etag.
6. If the OpenAPI specification needs to be downloaded prior to code generation, it should be put in @internal/bua/data/openapi.json

**Grill me to resolve omissions and ambiguities**