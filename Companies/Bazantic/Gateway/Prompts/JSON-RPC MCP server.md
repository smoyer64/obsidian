
# JSON-RPC MCP server

The `gateway` already generates an MCP server for services that include an OpenAPI specification.  This document provides the requirements for a "parallel" feature to generate an MCP server for services that include an OpenRPC specification.  It's important to note that, for a JSON-RPC service, it's also important to differentiate between whether the parameters are "named" or "positional".

## Requirements
- The `mcp.NewOpenRPCMiddleware` function should have name, version and doc parameters to mirror `mcp.NewMiddleware, but the document should be a parsed OpenRPC specification.

## Implementation notes

- There are tools that should help parse an OpenRPC specification and to help build a converter that can convert an MCP server's named parameters to positional parameters (for `positioned-jsonrpc`) available in the github.com/open-rpc/meta-schema Go package.
- A more comprehensive description of the OpenRPC schema as well as a full parser in the github.com/open-rpc/meta-schema Go package.