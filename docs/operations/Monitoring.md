# Monitoring

To keep you always in track of Kelon's current state, request/response statistics and many more performance metrics, Kelon provides two telemetry provider out of the box:

- [Prometheus](https://prometheus.io/)

One of them can be selected per running Kelon instance via the `telemetry-service`-flag. There are also more configuration options for each telemetry provider which you can finde in the [Configuration](/operations/Configuration#telemetry) section.

# Stats per telemetry provider

## Prometheus

After enabling Prometheus as telemetry provider, Kelon exposes all collected metrics over the HTTP-Endpoint `GET http://<host>:<port>/metrics`

Following metrics are currently provided for Prometheus

- Full system and Golang stats
- Full request stats
  - HTTP
  - GRPC (Proxied over HTTP internally)
- Database requests
- Errors (Currently only counter)