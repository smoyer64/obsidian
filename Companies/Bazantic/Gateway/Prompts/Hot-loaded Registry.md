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
- The ``