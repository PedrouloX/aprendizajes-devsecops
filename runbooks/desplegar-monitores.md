# Desplegar monitores en un servidor

Paso posterior a los scripts base (`01`–`04`). Usa el pipeline `deploy-observabilidad`.

## Antes

1. `/opt/monitores` existe con dueño el usuario de deploy (lo crea `03-create-apps-dir.sh`; en máquinas viejas, a mano o con una acción del pipeline de lifecycle):

```bash
sudo mkdir -p /opt/monitores && sudo chown <usuario-deploy>:<usuario-deploy> /opt/monitores
```

2. Reglas de red, validadas desde el origen:
   - Observabilidad → servidor TCP 9200, 9080 (+ 9104 MySQL / 9187 Postgres)
   - Servidor → observabilidad TCP 4317
   - Runners → servidor TCP 22
3. Si hay exporter de BDD: usuario `exporter` creado (ver `levantar-bdd-postgres-redis-compose.md`, paso 8).
4. Que no haya otra corrida del mismo pipeline en curso.

## Variables

| Variable | Valor |
|---|---|
| `TARGET_IP` | IP del servidor |
| `SERVER_LABEL` | `<proyecto>-<amb>` |
| `VPC` | `dev` / `qa` / `prd` (revisar el default) |
| `SERVER_ROLE` | `APP` / `MIXED` → variante apps; `DATABASE` → variante infra |
| `DB_ENGINE` | `NONE`, `MYSQL`, `POSTGRES` (+ `DB_PORT`, `DB_NAME`, `EXPORTER_DB_USER`, `EXPORTER_DB_PASSWORD`) |

## Validar

```bash
sudo docker ps --filter name=otel-agent --filter name=cadvisor --filter name=node-exporter
```

- Targets `up` en Prometheus.
- El servidor visible en el dashboard de contenedores filtrando por `SERVER_LABEL`.
- Si la app debe enviar trazas: desde su contenedor, `otel-agent:4317` responde (misma red Docker).

## Re-desplegar sobre monitores antiguos

Mismo `SERVER_LABEL` que ya tenía (para no partir el histórico en Grafana), un servidor a la vez. Si falla por permisos en `/opt/monitores`, corregir el dueño y lanzar un **pipeline nuevo** (reintentar el job usa el commit y las variables viejas).
