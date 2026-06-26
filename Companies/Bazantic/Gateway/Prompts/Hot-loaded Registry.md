---
tags:
  - bazantic
  - gateway
---
# Hot-loaded Registry

The Bazantic registry allows an authenticated client to programatically retrieve each service configuration via the Bazantic Universal API (BUA).  A REST client has been generated based on the BUA's OpenAPI specification and allows the required authentication but we need to periodically check for updates and, if any new or changed services are present, parse the new information and trigger the `router` update.

The prerequisite packages are already present:
- The `bua.Client` provides access to the "registry".
- The `timed.Task`provides a framework for operations that should be run on a recurring basis.
- The `reload.Trigger` instructs the `router` to rebuild and swap in a new instance without downtime.

## Requirements

Given the prerequisite packages listed above, the remaining work involves the following requirements:

- Create a `bua.Client`

## Configuration

The following configuration should be added to `config.Config` to support the hot-reloaded registry:

- The URL of the target BUA instance
- The JWT needed for authenticated access to the BUA instance
- The symmetric encryption key needed to decrypt the secrets needed to authenticate to each provider
- A string representing the duration (in Go format) to wait between checking for changes to the gateways returned by BUA

## Implementation notes

- 