# REST method extraction

The `gateway` code currently extracts the JSON-RPC method from a `named-jsonrpc` or `positioned-jsonrpc` request. Let's write a specification for the corresponding middleware that matches the request path for a REST method. The supported methods (and their prices) are now included in the `registry.Service` type.


## Requirements

The following requirements apply to the creation of the middleware specific to one of the `registry.Service`s that the `gateway` supports:

- The `rest.NewMiddleware` method should create a new `handler.Middleware` that contains a trie that can be walked to "decode" each request's incoming path.

The following requirements apply to the operation of the resulting middleware during `http.Request` handling:

* 

## Implementation notes

* An zero-allocation