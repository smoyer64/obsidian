
feat(tofu): recreate posthog mcp dashboard in signoz

# PostHog MCP Dashboard in SigNoz

PR #575 added telemetry for enhanced MCP analytics, with data being sent to both SigNoz (OpenTelemetry via gRPC) and PostHog (as direct event ingest.)  Write a specification in `/tofu/docs/specs` that creates a new OpenTofu unit in `/tofu/ops/

## Requirements

1. Create a new `root.hcl` in `/tofu/ops`that describes attributes common to all the operations infrastructure needed for Bazantic.
2. Set the remote statue bucket name for `/tofu/ops` to `souk-tofu-ops`.
3. Create a unit to IAC SigNoz dashboards and alerts in `/tofu/ops/signoz`.
4. Use the `signoz/signoz` provider.
5. Refer to PR #575 for additional details on what OpenTelemetry signals are sent as well as the attributes included with each.
6. Also refer to the MCP Analytics Instrumentation specification in `souk` for the complete specification.
7. Create a 
8. Duplicate the PostHog MCP dashboard shown in the image below as a SigNoz dashboard which can be "deployed" to our existing SigNoz instance: ![[image (8).png]]
9. Tag the new dashboard with `gateway` and `mcp`.

## Implementation notes

1. I can provide the credentials to write to our SigNoz instance as environment variables.  Update the `/tofu/.envrc_sample` file to indicate what those new environment variables are.
2. Provide a drop-down select list for `all`, `dev`, `pre` and `prd` environments that filters the contents of the dashboard by just the selected environment(s).
3. Provide a drop-down select list with the ability to type in the first few letters that either selects `all` gateways or one selected `gateway` identified by its `slug`.

**Grill me to resolve any ambiguities or omissions in the above.**

## References

1. [signoz/signoz provider](https://search.opentofu.org/provider/signoz/signoz/latest)
2. [MCP Analytics Instrumentation](./gateway/docs/specs/2026-07-18-mcp-analytics-instrumentation-design.md)