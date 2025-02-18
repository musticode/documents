# Traefik API and WebSocket configuration 


API : 8080
WS : 8887

http -> 


```yml
# Traefik Configuration for WebSocket
traefik.enable=true

# HTTP Router (WebSocket support)
traefik.http.routers.http-websocket.entryPoints=http
traefik.http.routers.http-websocket.rule=Host(`i00004gcg4kco04co80g44cc.188.132.197.160.sslip.io`) && PathPrefix(`/`)
traefik.http.routers.http-websocket.service=websocket-service

# Service (points to the WebSocket server on port 8887)
traefik.http.services.websocket-service.loadbalancer.server.port=8887

# HTTP Router (API support)
traefik.http.routers.http-api.entryPoints=http
traefik.http.routers.http-api.rule=Host(`i00004gcg4kco04co80g44cc.188.132.197.160.sslip.io`) && PathPrefix(`/api`)
traefik.http.routers.http-api.service=api-service

# Service (points to the API server on port 8080)
traefik.http.services.api-service.loadbalancer.server.port=8080
```

- API Configuration 2

```yml
# HTTPS Router (API support)
traefik.http.routers.https-api.entryPoints=https
traefik.http.routers.https-api.rule=Host(`i00004gcg4kco04co80g44cc.188.132.197.160.sslip.io`) && PathPrefix(`/api`)
traefik.http.routers.https-api.service=api-service
traefik.http.routers.https-api.tls=true

# Service (points to the API server on port 8080)
traefik.http.services.api-service.loadbalancer.server.port=8080
```