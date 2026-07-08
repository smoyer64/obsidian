# REST method extraction

The `gateway` code currently extracts the JSON-RPC method from a `named-jsonrpc` or `positioned-jsonrpc` request. Let's write a specification for the corresponding middleware that matches the request path for a REST method. The supported methods (and their prices) are now included in the `registry.Service` type.

## Requirements

The following requirements apply to the creation of the middleware specific to one of the `registry.Service`s that the `gateway` supports:

- The `rest.NewMiddleware` method should create a new `handler.Middleware` that contains a trie that can be walked to "decode" each request's incoming path.
- The `rest.NewMiddleware` method should error if any of the service's paths can't be added to the trie.  When this happens, a WARN-level log message should be emitted with a message and accompanying attributes that are sufficient to easily debug the problem.  This should also be compatible with a future `doctor` that provides a list of fatal and non-fatal problems and suggests their solutions.

The following requirements apply to the operation of the resulting middleware during `http.Request` handling:

* An incoming request path should be compared to the paths represented by the trie segment-by-segment.
* If the incoming path can't be matched, the middleware should return a `400 Bad Request` instead of forwarding the request to the upstream provider.

## Implementation notes

* A "path trie" that can be walked with zero-allocations is provided in the @internal/rest package.  Since we're not extracting the path parameter names and positions, this structure is:
	* Less expensive to construct when the `router` is reloaded, which is important when large numbers of services need to be processed.
	* Consumes less memory for the trie itself
	* Is often faster than the radix trie used by projects like 

## Extraneous

Running the existing tests and benchmarks can be accomplished using the following shell commands:
```
# Run tests
go test -v ./rest

# Run the allocation benchmark
go test -bench=BenchmarkRouter_BlackBoxMatch -benchmem ./rest
```

**Feel free to grill me to resolve any ambiguities or omissions above**