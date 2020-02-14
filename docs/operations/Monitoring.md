# Monitoring

To keep you always in track of Kelon's current state, request/response statistics and many more performance metrics, Kelon provides two telemetry provider out of the box:

- [Prometheus](https://prometheus.io/)
- [Application Insights](https://docs.microsoft.com/de-de/azure/azure-monitor/app/app-insights-overview) by Microsoft

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

## Application Insights

Application Insights has a push based model which is why there is no /metrics endpoint exposed when selecting Application Insights as telemetry provider. Instead you have to provide at least the `instrumentation key` from your previously created Application Insights Application from Azure-Portal. Afterwards Kelon pushes all available metrics into the Azure-Cloud on its own.

Following metrics are currently provided for Application Insights

- Basic system stats
    - % Processor Time
    - Heap Memory Used
    - % Heap Memory Used
    - IO Data Bytes/sec
    - Data In-Bytes/sec
    - Data Out-Bytes/sec
- Full request stats
    - HTTP
    - GRPC (Proxied over HTTP internally)
- Database request (including queries)
    - Dependency-Name/Type displayed in Azure's application overview can be configured via the datastore.yml [here](/operations/Configuration#datastoreyml)
- Errors (Including cause)
- End-to-End-tracking of all requests going into Kelon
- Entire log (Log levels can be configured [here](/operations/Configuration/#application-insights))