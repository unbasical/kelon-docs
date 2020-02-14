# Kelon and services

To integrate Kelon with your service you basically have two options

1. Add middleware-code to your service which intercepts calls and sends them to Kelon for authorization

2. [Hook Kelon into a Proxy like Envoy](/core/integration/Kelon-Envoy) which takes all incoming traffic, authorizes it agains Kelon and then forwards it to your service

## Middleware-Interceptors

The core thing that each interceptor for Kelon needs to do is to gather all information about the incoming request and send it via a HTTP-POST to kelon for authorization (no mather which framework or programming language you want to use)

A possible call to Kelon for an incoming `GET /api/apps/1` might look like this:

**POST http://{Path to kelon}:{Port of kelon}/v1/data/allow**
```
{
    "input": {
        "method": "GET",
        "path": "/api/apps/1",
        "token": "<JWT-Token from request header>"
    }
}
```

Please note that the only **required fields** inside the input object are **method** and **path**! All other fields can be added as you like and are available inside your policies by accessing them with i.e. `data.input.token`.