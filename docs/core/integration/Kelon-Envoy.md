Kelon implements Envoy's [External Authorization API](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter#config-http-filters-ext-authz) which makes it quite easy to integrate it with Envoy. This setup becomes quite handy if you want to keep all your currently deployed services untouched and deploy Kelon together with Envoy as central gateway which authorizes the entire traffic coming in your cluster.

**The complete deployment of Kelon and Envoy as Gateway for our Appstore-Example inside Kubernetes is also available [here](https://github.com/Foundato/kelon-examples/tree/master/appstore-envoy-kelon-kube-mgmt-example)**

## Minimal setup

To enable the Kelon's GRPC-Encpoint which is used by Envoy to authorize all request you just have to set the flag `--envoy-port` to a valid port and that's it.

The minimal config to setup Envoy with is following:

```yaml
admin:
  access_log_path: "/dev/null"
  address:
    # Address and Port for envoy's admin interface
    socket_address:
      address: 0.0.0.0
      port_value: 8001
static_resources:
  listeners:
    - address:
        # Address and Port Envoy is listening on
        socket_address:
          address: 0.0.0.0
          port_value: 8000
      filter_chains:
        - filters:
            - name: envoy.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
                codec_type: auto
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  # Forward all incoming traffic to your service after all filters have passed
                  virtual_hosts:
                    - name: backend
                      domains:
                        - "*"
                      routes:
                        - match:
                            prefix: "/"
                          route:
                            cluster: service
                # Entire filter chain which has to pass before 
                # Envoy sends requests to your service
                http_filters:
                  - name: envoy.ext_authz
                    config:
                      with_request_body:
                        max_request_bytes: 8192
                        allow_partial_message: true
                      failure_mode_allow: false
                      grpc_service:
                        google_grpc:
                          target_uri: "<uri to kelon>:<port under which kelon waits for envoy requests>"
                          stat_prefix: ext_authz
                        timeout: 0.5s
                  - name: envoy.router
                    typed_config: {}
clusters:
  - name: service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: service
      endpoints:
        - lb_endpoints:
          - endpoint:
              address:
                # Your deployed service which you want to protect
                socket_address:
                  address: <uri to your deployed service>
                  port_value: <port of your service>
```

## CORS

While this is a working setup of Envoy and Kelon, we also recommend to add at least Envoy's [CORS-Filter](https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/cors.html) in front of ext_auth to correctly terminate all CORS-Headers of inconming traffic. Following Config of envoy shows how to include Envoy's CORS-Filter:

```yaml
admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8001
static_resources:
  listeners:
    - address:
        socket_address:
          address: 0.0.0.0
          port_value: 8000
      filter_chains:
        - filters:
            - name: envoy.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
                codec_type: auto
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: backend
                      domains:
                        - "*"
                      # =======================================
                      # Configure CORS-Filter for each service
                      cors:
                        allow_origin_string_match:
                          - safe_regex:
                              google_re2: {}
                              regex: \*
                        allow_methods: "*"
                        allow_headers: "*"
                        filter_enabled:
                          default_value:
                            numerator: 100
                            denominator: HUNDRED
                          runtime_key: cors.www.enabled
                        shadow_enabled:
                          default_value:
                            numerator: 0
                            denominator: HUNDRED
                          runtime_key: cors.www.shadow_enabled
                      routes:
                        - match:
                            prefix: "/"
                          route:
                            cluster: service
                      # =======================================
                http_filters:
                  # =======================================
                  # Add CORS filter before Kelon inside the filter chain
                  - name: envoy.cors
                    typed_config: {}
                  # =======================================
                  - name: envoy.ext_authz
                    config:
                      with_request_body:
                        max_request_bytes: 8192
                        allow_partial_message: true
                      failure_mode_allow: false
                      grpc_service:
                        google_grpc:
                          target_uri: "<uri to kelon>:<port under which kelon waits for envoy requests>"
                          stat_prefix: ext_authz
                        timeout: 0.5s
                  - name: envoy.router
                    typed_config: {}
clusters:
  - name: service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: service
      endpoints:
        - lb_endpoints:
          - endpoint:
              address:
                socket_address:
                  address: <uri to your deployed service>
                  port_value: <port of your service>
```

## Authentication (Auth0 & JWKS)

If you want to keep `Authorization` as well as `Authentication` completely away from your services, you can also add Envoy's [JWT-Auth-Filter](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/security/jwt_authn_filter) to your filter chain. With this setup all of your services can completely focus on their main task... and this is business-logic and not authenticating or authorizing it!

Following Config of envoy shows how to include Envoy's JWT-Auth-Filter:

```yaml
admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8001
static_resources:
  listeners:
    - address:
        socket_address:
          address: 0.0.0.0
          port_value: 8000
      filter_chains:
        - filters:
            - name: envoy.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
                codec_type: auto
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: backend
                      domains:
                        - "*"
                      cors:
                        allow_origin_string_match:
                          - safe_regex:
                              google_re2: {}
                              regex: \*
                        allow_methods: "*"
                        allow_headers: "*"
                        filter_enabled:
                          default_value:
                            numerator: 100
                            denominator: HUNDRED
                          runtime_key: cors.www.enabled
                        shadow_enabled:
                          default_value:
                            numerator: 0
                            denominator: HUNDRED
                          runtime_key: cors.www.shadow_enabled
                      routes:
                        - match:
                            prefix: "/"
                          route:
                            cluster: service
                http_filters:
                  - name: envoy.cors
                    typed_config: {}
                  # =======================================
                  # Add and configure JWT-Auth-Filter
                  - name: envoy.filters.http.jwt_authn
                    config:
                      providers:
                        provider1:
                          issuer: <issuer of your tokens. i.e. https://dev-kelon.eu.auth0.com/>
                          remote_jwks:
                            http_uri:
                              uri: <uri where to fetch public keys from. i.e. https://dev-kelon.eu.auth0.com/.well-known/jwks.json>
                              cluster: auth0
                              timeout:
                                seconds: 5
                      rules:
                        # Completely ignore health checks for authentication
                        - match:
                            prefix: /health
                        - match:
                        # Completely ignore login for authentication
                            prefix: /api/login
                        # Authenticate all requests against /api/apps
                        - match:
                            prefix: /api/apps
                          requires:
                            provider_name: provider1
                  # =======================================
                  - name: envoy.ext_authz
                    config:
                      with_request_body:
                        max_request_bytes: 8192
                        allow_partial_message: true
                      failure_mode_allow: false
                      grpc_service:
                        google_grpc:
                          target_uri: "<uri to kelon>:<port under which kelon waits for envoy requests>"
                          stat_prefix: ext_authz
                        timeout: 0.5s
                  - name: envoy.router
                    typed_config: {}
clusters:
  - name: service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: service
      endpoints:
        - lb_endpoints:
          - endpoint:
              address:
                socket_address:
                  address: <uri to your deployed service>
                  port_value: <port of your service>
  # =======================================
  # You also have to specify a cluster which describes the authentication endpoint
  - name: auth0
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: auth0
      endpoints:
        - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: <uri to your auth provider. i.e. dev-kelon.eu.auth0.com>
                    port_value: 443
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.api.v2.auth.UpstreamTlsContext
        common_tls_context:
          validation_context:
            trusted_ca:
              filename: "/etc/ssl/certs/ca-certificates.crt"
        sni: <server name indication of your auth provider. i.e. dev-kelon.eu.auth0.com>
```