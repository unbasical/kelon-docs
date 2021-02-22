# Testing

## Integration testing
Our core piece of business logic are the datastores, that is why we need to ensure they are functional as expected.
Therefore, a highly configurable integration test suite is provided: 
<ul>
<li>Datastore config</li>
<li>API config</li>
<li>Policies config</li>
<li>OPA config</li>
</ul>
For a test run, you have to define your kelon requests, the expected response and the expected database query.
You can evaluate almost the total process of policy evaluation from the Kelon API request to the translated databse query.

Starting test suite:
```
make integration-test
```

Example: [integration tests](https://github.com/Foundato/kelon/tree/develop/test/integration)

## E2E testing
The E2E-Testing starts a local docker-compose environment and runs a Postman collection against Kelon.
The E2E test result is provided as junit.xml.

Starting E2E-Test
```
make e2e-test
```

Example: [e2e tests](https://github.com/Foundato/kelon/tree/develop/test/e2e)


## Load testing
Load-Testing uses also a local docker-compose environment and k6 for load testing. 
For each database (mongo, mysql and postgres) a separate k6 collection runs against the local environment.
The load test result is provided as junit.xml.

Start load tests:
```
make load-test
```

Translate/update postman collections to k6 collections:
```
make load-test-update-postman
```

Example: [load tests](https://github.com/Foundato/kelon/tree/develop/test/load)
