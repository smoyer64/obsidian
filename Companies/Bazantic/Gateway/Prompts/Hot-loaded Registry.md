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

- Create a `bua.Client`and make a request for the list of available `gateway`s. which must include the `If-Modified-Since`header.
- If the HTTP response status code is `304 (Not Modified)`, the process should log that no `gateway`s have changed and return to the waiting state.
- If the HTTP response status code is `200 (Ok)`, the process should swap the pointer to the new list of `gateway`s and trigger a router rebuild.  It should also save a copy of the returned JSON to the cache directory.  The value of the `Last-Modified` header should also be saved to a file in the cache directory.
- The returned list of `gateway`s will replace the existing `[]config.Provider` in the router configuration.  When looping through these entries during the router rebuild, the existing fields should be mapped to the new, equivalent fields.
## Configuration

The following configuration should be added to `config.Config` to support the hot-reloaded registry:

- The URL of the target BUA instance
- The JWT needed for authenticated access to the BUA instance
- The symmetric encryption key needed to decrypt the secrets needed to authenticate to each provider
- A string representing the duration (in Go format) to wait between checking for changes to the gateways returned by BUA

## Implementation notes

- The new BUA hot-reloaded registry should be in the `registry` package.
- The `wire` DI system should be used to provide all the reloader's dependencies via the constructor.  Likewise, `wire` should be used to inject the needed trigger and to add the reloader to the timed tasks managed by the top-level application.
- The call to BUA should be roughly equivalent to the following `cURL` command:
	```
	curl 'https://api.bazantic.com/v1/gateways?expanded=true&status=live' -H 'Authorization: Bearer bzr_Gx_OHK1HD4wWtiQ81Bow_by3gmK4PZcS8nngmG3bC9M' -H 'If-Modified-Since:Wed, 17 Jun 2026 13:00:06 GMT'
	```
- The `openapi.Refresher`is a good example of how this new package should work with the major difference being the use of the `ETag` and `If-None-Match` headers instead of the `Last-Modified` and `If-Modified-Since` headers.

**Write a specification that describes the above requirements in a manner sufficient to perform the task break-down and implementation autonomously.  Grill me to resolve omissions and ambiguities**