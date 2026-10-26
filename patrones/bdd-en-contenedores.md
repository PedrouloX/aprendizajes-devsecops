# Bases de datos en contenedores

## Reglas

- **Siempre Docker Compose, nunca `docker run`**: el compose es la documentación viva del servicio.
- **`mem_limit` obligatorio** en todo contenedor nuevo (y `memswap_limit` igual al límite en servidores sin swap).
- **Redis**: además del `mem_limit`, `--maxmemory` por debajo del límite (ej. `mem_limit: 512m` + `--maxmemory 384mb`). Sin `maxmemory`, al llegar al límite Docker mata el proceso (OOM) en vez de que Redis rechace escrituras.
- Redis siempre con `requirepass` y persistencia (`--appendonly yes --appendfsync everysec`) desde el día uno.
- **Versiones fijas** (`postgres:18.3`, `redis:8.6.1-alpine`, `mysql:8.0.45`), nunca `latest` ni tags flotantes como `mysql:8.0`.
- Restaurar un dump solo en una versión igual o superior a la de origen.
- **Root separado del rol de aplicación.** La app nunca se conecta como superusuario. Si usa Prisma, el rol de app lleva `CREATEDB` (shadow database), no superusuario.
- Usuarios estándar con nombre fijo en todos los ambientes:
  - `exporter` con `pg_monitor` (o `PROCESS, REPLICATION CLIENT` + `SELECT` sobre la base en MySQL) para el exporter de métricas.
  - `backup_reader` de solo lectura para respaldos.
- Nombres de base sin guiones (`app_db`). Con guion hay que citarlos siempre en SQL.

## Estructura en el servidor

```
/opt/db-compose/<proy>/
  docker-compose.yml
  .env                         600 root:root
  init/01-create-app-user.sql  600 999:999
/opt/databases/<proy>-postgres-<amb>/   999:999
/opt/databases/<proy>-redis-<amb>/      999:999
```

- Compose y datos en carpetas separadas.
- Postgres 18+: montar en `/var/lib/postgresql`, no en `/var/lib/postgresql/data`.
- Puertos publicados al host por servicio (ej. `8201`, `8202`…) cuando la app o la VPN están en otro servidor; sin publicar si la app vive en el mismo host y la red Docker basta.

## Redis pertenece a la API

Redis no es parte del motor de base de datos: es de la capa API/worker de cada dominio. Puede vivir en el servidor de BDD por comodidad, pero dos Redis de servicios distintos son instancias distintas. Y un Redis "del servicio X" puede terminar siendo usado por otros (colas, caché): revisar `CLIENT LIST` antes de asumir quién lo usa.

## Servicios co-ubicados

Un servicio que la app resuelve por nombre de contenedor en la misma red Docker (ej. MeiliSearch junto a la API) no requiere regla de firewall. Se monta en el servidor de la app, no en el de BDD.
