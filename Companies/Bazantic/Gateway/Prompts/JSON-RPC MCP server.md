
# JSON-RPC MCP server

The `gateway` already generates an MCP server for services that include an OpenAPI specification.  This document provides the requirements for a "parallel" feature to generate an MCP server for services that include an OpenRPC specification.  It's important to note that, for a JSON-RPC service, it's also important to differentiate between whether the parameters are "named" or "positional".

## Requirements
- The `mcp.NewOpenRPCMiddleware` function should have name, version and doc parameters to mirror `mcp.NewMiddleware, but the document should be a parsed OpenRPC specification.
- Each method in an OpenRPC document should be read to set the associated MCP tool's name and description.
- Each method's parameters in an OpenRPC document should be read and dereferenced to create the associated MCP tool's input schema.
- Each method's result object in an OpenRPC document should be read and dereferenced to create the associated MCP tool's output schema.
- For services that use `positioned-jsonrpc`, the parsing process must keep track of the parameter definition order and "translate" the "named" parameters passed from the MCP tool call to the properly ordered format.  This is effectively a JSON object to JSON array conversion.

## Implementation notes

- There are tools that should help parse an OpenRPC specification and to help build a converter that can convert an MCP server's named parameters to positional parameters (for `positioned-jsonrpc`) available in the github.com/open-rpc/meta-schema Go package.
- A more comprehensive description of the OpenRPC schema as well as a full parser in the github.com/open-rpc/meta-schema Go package.
- The `mcp.NewMiddleware` function should probably be renamed `mcp.NewOpenAPIMiddleware`.

**Let's write a specification for the above requirements.  Grill me if there are ambiguities or omissions**