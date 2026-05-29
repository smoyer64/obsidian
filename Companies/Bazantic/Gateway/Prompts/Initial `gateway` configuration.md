
The initial specification for the Bazantic Gateway configuration is:

1.  In @internal/config/config.go, there should be a `Config` struct that has the following fields:
	1. Env (`string` required), 
	2. Cloud` (`string` required)
	3. OTEL` (`bool` optional, `false` if absent)
	4. LogLevel (`LevelVar` optional, `INFO` if absent)
	5. LogFormat (`string` )
	these fields that are loaded from environment variables in the form `GATEWAY_XXX` using the github.com/caarlos0/env/v11 library.
2.  The struct in #1 should contain the appropriate field tags be validated using the github.com/go-playground/validator/v10 library.