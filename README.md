# Fix: Budibase self-hosted REST datasource timeout (504 / thread timeout exceeded)

## Problema

Al ejecutar queries REST en Budibase self-hosted, las peticiones que tardan 
más de 15-60 segundos fallan con uno de estos errores:

- `thread timeout exceeded 15000ms`
- `504 Bad Gateway`
- `upstream prematurely closed connection`

## Causa

Budibase tiene múltiples capas de timeout con valores por defecto muy bajos:

| Variable | Default | Capa |
|---|---|---|
| `QUERY_THREAD_TIMEOUT` | 15000ms | Thread de ejecución de queries |
| `HTTP_SERVER_TIMEOUT_MS` | 60000ms | Servidor HTTP Node.js |
| `HTTP_KEEPALIVE_TIMEOUT_MS` | 15000ms | Conexiones keepalive |
| `PROXY_TIMEOUT_SECONDS` | 120s | Nginx proxy |

## Solución

Agregar estas variables en el `docker-compose.yml`:

### app-service
```yaml
app-service:
  environment:
    QUERY_THREAD_TIMEOUT: 240000
    AUTOMATION_THREAD_TIMEOUT: 240000
    HTTP_SERVER_TIMEOUT_MS: 240000
    HTTP_REQUEST_TIMEOUT_MS: 240000
    HTTP_KEEPALIVE_TIMEOUT_MS: 240000
    HTTP_HEADERS_TIMEOUT_MS: 245000
```

### proxy-service
```yaml
proxy-service:
  environment:
    - PROXY_TIMEOUT_SECONDS=240
```

> ⚠️ `HTTP_HEADERS_TIMEOUT_MS` siempre debe ser mayor que `HTTP_KEEPALIVE_TIMEOUT_MS` o Node.js lanzará un warning.

## Versión probada
- Budibase `3.14.0`
- Budibase `3.28.2`

## Referencias

- [Budibase hosting settings docs](https://docs.budibase.com/docs/hosting-settings)
