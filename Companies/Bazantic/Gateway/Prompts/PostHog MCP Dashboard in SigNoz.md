
feat(tofu): 

# PostHog MCP Dashboard in SigNoz

PR #575 added telemetry for enhanced MCP analytics, with data being sent to both SigNoz (OpenTelemetry via gRPC) and PostHog (as direct event ingest.)  Write a specification in `/tofu/docs/specs` that creates a new OpenTofu unit in `/tofu/ops/

## Requirements

1. Create a new `root.hcl` in `/tofu/ops`that describes attributes common to all the operations infrastructure needed for Bazantic.