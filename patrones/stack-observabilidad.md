# Stack de observabilidad

## Central (servidor de observabilidad en STAFF)

| Servicio | Puerto | Rol |
|---|---|---|
| otel-collector (gateway) | 4317 gRPC, 4318 HTTP, 8889 métricas | Recibe de agentes y apps |
| Prometheus | 9090 | Métricas |
| Loki | 3100 | Logs |
| Tempo | 3200 | Trazas + metrics generator (`service-graphs`, `span-metrics`) |
| Grafana | 443 vía Nginx | Visualización y alertas |

- **Tempo fijado en 2.6.x**: desde 2.10 requiere Kafka. No actualizar sin cambiar la arquitectura.

## Agentes en cada servidor (`/opt/monitores/`)

| Servicio | Puerto host | Nota |
|---|---|---|
| node-exporter | 9200 → 9100 | Métricas del SO |
| cAdvisor | 9080 | Métricas de contenedores |
| otel-agent | — (no publica) | Reenvía a `<ip-observabilidad>:4317` |

Variantes de compose:
- `infra`: servidores sin apps instrumentadas (incluye BDD).
- `apps`: otel-agent en la red Docker de las apps, que le envían OTLP a `otel-agent:4317`; con `filelog` para logs de contenedores.

Límites en los agentes: otel-agent ~1280m, cAdvisor 512m, node-exporter 128m. Es normal que otel-agent use mucha CPU los primeros minutos (lee todos los logs existentes).

## Despliegue con pipeline

Variables: `TARGET_IP`, `SERVER_LABEL` (`<proyecto>-<amb>`, siempre con sufijo de ambiente), `VPC`, `SERVER_ROLE`, `DB_ENGINE` (+ puerto, base, usuario `exporter`).

- **Nunca dos corridas en paralelo**: todas editan el mismo `prometheus.yml` y la última pisa a la otra.
- Respaldo de `prometheus.yml` en cada corrida, conservando solo los últimos N.
- Requisito: `/opt/monitores` con dueño el usuario de deploy (no tiene sudo para crearla).
- Label `vpc` en cada target para filtrar dashboards por ambiente.

## Telemetría de apps Node.js

```
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-agent:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_SERVICE_NAME=<app>-<ambiente>
LOG_NAME=<app>-<ambiente>
LOG_PATH=./logs        relativo; rutas absolutas dan errores de permisos
```

Paquetes: `@opentelemetry/sdk-node`, `auto-instrumentations-node`, `exporter-trace-otlp-grpc`, `exporter-logs-otlp-grpc`, `sdk-logs`, y **`@prisma/instrumentation`** (no `@opentelemetry/instrumentation-prisma`, que no existe) en la misma versión que `@prisma/client`, sin caret.

`instrumentation.ts` debe ser el **primer import** del `server.ts`/`worker.ts`.

En Tempo, los endpoints de Express aparecen como `SPAN_KIND_INTERNAL` con `span_name="request handler - /ruta"`.

## Loki

- Solo `host_name` y `service_name` son labels indexados (van entre `{}`). Otros atributos (ej. `servidor`) son metadata estructurada: `{host_name="x"} | servidor="y"`.
- Nivel: `detected_level` (minúsculas).
- En Loki usar `$__auto`; `$__rate_interval` es de Prometheus.
- Contenedores sin SDK (BDD, Nginx, Redis) llegan por `filelog` y salen como `unknown_service` salvo que tengan `--log-opt tag="{{.Name}}"` y un `transform` en el agente (ver lección `filelog-sin-logs-selinux.md`).

## Rotación de logs Docker

```json
{ "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }
```

## Thresholds de referencia

- Latencia HTTP: amarillo 200 ms, rojo 1 s.
- Latencia BDD: amarillo 100 ms, rojo 500 ms.
