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

The 
## References

1. https://a2a-protocol.org/latest/specification/