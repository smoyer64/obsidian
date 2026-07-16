
Write a specification for the following new feature:
# Agent-Card generation

When the Bazantic `gateway` generates an MCP server for a gateway that provides an OpenAPI or OpenRPC specification, we have all the information we need to also generate an `agent-card.json` file that complies with the A2A protocol specification.

## Mirror format

While the abstracted `agent-card` has better discoverability, we don't have an LLM local to the `gateway` to determine how tools should be grouped.  We're also rejecting the flat-client format which has poor discoverability.  This leaves us with the mirror format, which provides an `agent-card` that lists the details about the MCP server's prompts, resources and tools in a single document.  Here's an example of an `agent-card.json` file in the mirror format:

```
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "id": "database-mcp-agent",
  "name": "Database Operations Agent",
  "version": "1.0.0",
  "description": "An agent that directly mirrors the raw schema-inspection and querying capabilities of the PostgreSQL MCP server.",
  
  "prompts": [
    {
      "name": "optimize-query",
      "description": "Instructs the LLM to write a performance-optimized SQL query based on a provided schema and user requirements."
    }
  ],

  "resources": [
    {
      "uri": "postgres://production-db/schema-summary",
      "name": "Database Schema Summary",
      "mimeType": "application/json",
      "description": "A high-level structural map of all active tables, primary keys, and foreign-key relationships."
    }
  ],

  "tools": [
    {
      "name": "db_list_tables",
      "description": "Retrieves a flat list of all public table names present in the active database connection.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "schema": {
            "type": "string",
            "description": "The database schema to inspect. Defaults to 'public'.",
            "default": "public"
          }
        }
      }
    },
    {
      "name": "db_execute_read",
      "description": "Executes a read-only SQL query (SELECT) against the database and returns the results as a JSON array.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "sql_query": {
            "type": "string",
            "description": "The read-only SQL query to run. Must begin with SELECT."
          },
          "limit": {
            "type": "integer",
            "description": "Maximum number of rows to return. Defaults to 100.",
            "default": 100
          }
        },
        "required": [
          "sql_query"
        ]
      }
    }
  ]
}
```

## Requirements

The agent-card generation system must meet the following requirements:

1. The generated agent-card must be compliant with the specification
2. The generated agent-card must be structured as the example above with one tool entry per MCP tool.
3. The generated agent-card must be served "next to" the associated MCP server - if the MCP server is at `/mcp`, then the assocated agent-card should be `/.well-known/agent-card.json`.
4. The generation of an agent-card must trigger an asynchronous "publisher".
5. The generation of an agent-card should be included in the router's rebuild loop.

## Implementation notes

1. The agent-card publisher will be a future feature.  This task should simply create an interface to trigger the asynchronous publishing and a stub implementation.
2. The agent-card should only contain the tools that are successfully generated from an OpenAPI or Open RPC specification.
3. The agent-card should only be served if the associated MCP server successfully starts up
## References

1. https://a2a-protocol.org/latest/specification/
2. https://modelcontextprotocol.io/specification/2025-11-25

**Grill me to resolve any ambiguities or omission in the above requirements document.
