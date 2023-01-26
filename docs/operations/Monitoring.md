# Monitoring

To keep you always in track of Kelon's current state, request/response statistics and many more performance metrics and traces, Kelon provides two telemetry provider out of the box:

- [Prometheus](https://prometheus.io/)
- [OpenTelemetry](https://opentelemetry.io/)

One of them can be selected per running Kelon instance via the `metric-provider`-flag. There are also more configuration options for each telemetry provider which you can finde in the [Configuration](/operations/Configuration#telemetry) section.

## Metrics

Both metrics implementations (Prometheus and OpenTelemetry) provide a common set of metrics:
* Full system and Golang stats
* Full request stats
  * HTTP
  * GRPC (Proxied over HTTP internally)
* Database requests
* Errors (Currently only counter)

### Prometheus
After enabling Prometheus as metrics provider, Kelon exposes all collected metrics over the HTTP-Endpoint `GET http://<host>:<port>/metrics`

### OpenTelemetry
After enabling Prometheus as metrics provider, Kelon exposes all collected metrics using the `otlp` protocol.


## Tracing

All traces are exported using the `otlp` protocol.