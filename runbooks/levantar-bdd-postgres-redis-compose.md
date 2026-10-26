# Levantar PostgreSQL + Redis con Docker Compose

Para una máquina nueva de un proyecto: Postgres y Redis en contenedores, root separado del usuario de aplicación, persistencia, límites de memoria y validación. Todo por SSH con un usuario con `sudo`.

| Marcador | Significado | Ejemplo |
|---|---|---|
| `<proy>` | Prefijo del proyecto | `app` |
| `<amb>` | `dev`, `qa`, `preprod`, `prd` | `qa` |
| `<db>` | Base (sin guiones) | `app_db` |
| `<PG_PORT>` | Puerto host de Postgres | `8201` |
| `<REDIS_PORT>` | Puerto host de Redis | `8202` |

## 0. Requisitos

Máquina con Docker y la red de apps creada:

```bash
sudo docker network ls | grep apps-network
```

## 1. Passwords

Cuatro (root, app, Redis, exporter), 20 caracteres alfanuméricos para no romper URLs:

```bash
for i in 1 2 3 4; do tr -dc 'A-Za-z0-9' </dev/urandom | head -c 20; echo; done
```

Nunca reutilizar las de otro ambiente.

## 2. Carpetas

```bash
sudo mkdir -p /opt/db-compose/<proy>/init /opt/databases/<proy>-postgres-<amb> /opt/databases/<proy>-redis-<amb>
sudo chown -R root:root /opt/db-compose && sudo chmod -R 755 /opt/db-compose
sudo chown 999:999 /opt/databases/<proy>-postgres-<amb> /opt/databases/<proy>-redis-<amb>
```

- Configuración `root:root` 755 (nunca a nombre de un usuario personal, nunca 750).
- Datos `999:999` (UID de Postgres/Redis dentro del contenedor; en el host aparece como `systemd-coredump`).

## 3. `.env`

```bash
sudo tee /opt/db-compose/<proy>/.env > /dev/null <<'EOF'
POSTGRES_DB=<db>
POSTGRES_ROOT_USER=<proy>_root
POSTGRES_ROOT_PASSWORD=<password root>
POSTGRES_APP_USER=<proy>_app
POSTGRES_APP_PASSWORD=<password app>
REDIS_PASSWORD=<password redis>
EOF
sudo chmod 600 /opt/db-compose/<proy>/.env
sudo cut -d= -f1 /opt/db-compose/<proy>/.env
```

El último comando lista solo los nombres, sin mostrar las passwords.

## 4. `docker-compose.yml`

```bash
sudo tee /opt/db-compose/<proy>/docker-compose.yml > /dev/null <<'EOF'
services:
  <proy>-postgres-<amb>:
    image: postgres:18.3
    container_name: <proy>-postgres-<amb>
    restart: unless-stopped
    mem_limit: 1g
    memswap_limit: 1g
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_ROOT_USER}
      POSTGRES_PASSWORD: ${POSTGRES_ROOT_PASSWORD}
    volumes:
      - /opt/databases/<proy>-postgres-<amb>:/var/lib/postgresql
      - /opt/db-compose/<proy>/init:/docker-entrypoint-initdb.d
    ports:
      - "<PG_PORT>:5432"
    networks:
      - apps-network

  <proy>-redis-<amb>:
    image: redis:8.6.1-alpine
    container_name: <proy>-redis-<amb>
    restart: unless-stopped
    mem_limit: 512m
    memswap_limit: 512m
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes --appendfsync everysec --maxmemory 384mb
    volumes:
      - /opt/databases/<proy>-redis-<amb>:/data
    ports:
      - "<REDIS_PORT>:6379"
    networks:
      - apps-network

networks:
  apps-network:
    external: true
EOF
cd /opt/db-compose/<proy> && sudo docker compose config --quiet && echo "COMPOSE OK"
```

- Postgres 18+: volumen en `/var/lib/postgresql`. Con la ruta antigua no arranca.
- Ajustar `mem_limit` a la RAM del servidor.

## 5. Script de init (usuario de aplicación con permiso mínimo)

```bash
sudo tee /opt/db-compose/<proy>/init/01-create-app-user.sql > /dev/null <<'EOF'
CREATE ROLE <proy>_app WITH LOGIN PASSWORD '<password app>';
GRANT CONNECT ON DATABASE <db> TO <proy>_app;
GRANT USAGE, CREATE ON SCHEMA public TO <proy>_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO <proy>_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO <proy>_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO <proy>_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO <proy>_app;
EOF
sudo chown 999:999 /opt/db-compose/<proy>/init/01-create-app-user.sql
sudo chmod 600 /opt/db-compose/<proy>/init/01-create-app-user.sql
```

- Dueño `999` y `600`: lo lee el usuario `postgres` del contenedor. Con `root` + `600` el init falla.
- `GRANT ALL ON DATABASE` **no** da permisos sobre tablas: por eso los `GRANT` explícitos y los `ALTER DEFAULT PRIVILEGES`.
- Si la API usa Prisma: `ALTER ROLE <proy>_app CREATEDB;`.
- **Solo corre la primera vez** (carpeta de datos vacía).

## 6. Levantar y verificar

```bash
cd /opt/db-compose/<proy> && sudo docker compose up -d
sleep 10
sudo docker ps --filter "name=<proy>-" --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
sudo docker logs <proy>-postgres-<amb> 2>&1 | grep -iE "01-create-app-user|error|ready to accept"
```

Esperado: `running /docker-entrypoint-initdb.d/01-create-app-user.sql` y `ready to accept connections` (dos veces).

Validar **por IP, no por localhost** (la imagen trae `trust` para conexiones locales) y con control negativo:

```bash
sudo docker exec -e PGPASSWORD='<password app>' <proy>-postgres-<amb> sh -c 'psql -h "$(hostname -i)" -U <proy>_app -d <db> -c "CREATE TABLE _t (id int); DROP TABLE _t;"'
sudo docker exec -e PGPASSWORD='incorrecta' <proy>-postgres-<amb> sh -c 'psql -h "$(hostname -i)" -U <proy>_app -d <db> -c "SELECT 1"'
sudo docker exec <proy>-redis-<amb> redis-cli -a '<password redis>' --no-auth-warning ping
sudo docker inspect -f '{{.Name}} {{.HostConfig.Memory}}' <proy>-postgres-<amb> <proy>-redis-<amb>
```

Debe responder `DROP TABLE`, `password authentication failed`, `PONG` y los límites en bytes.

## 7. Si el init no corrió

```bash
sudo docker exec -i <proy>-postgres-<amb> psql -U <proy>_root -d <db> < /opt/db-compose/<proy>/init/01-create-app-user.sql
```

## 8. Usuario de monitoreo

```bash
sudo docker exec <proy>-postgres-<amb> psql -U <proy>_root -d <db> -c "CREATE ROLE exporter WITH LOGIN PASSWORD '<password exporter>';" -c "GRANT pg_monitor TO exporter;"
```

## 9. URLs

Misma red Docker y mismo host (nombre del contenedor, puerto interno):

```
DATABASE_URL=postgresql://<proy>_app:<pass>@<proy>-postgres-<amb>:5432/<db>
REDIS_URL=redis://:<pass>@<proy>-redis-<amb>:6379
```

API en otro servidor o cliente por VPN (IP y puerto publicado; requiere regla de red):

```
DATABASE_URL=postgresql://<proy>_app:<pass>@<ip-bdd>:<PG_PORT>/<db>
REDIS_URL=redis://:<pass>@<ip-bdd>:<REDIS_PORT>
```

## 10. Reglas de red a probar

Ver `validar-reglas-de-red.md`: VPN → puertos de BDD, runners → 22, servidor → servicio de identidad 443.

## 11. Copiar el compose a otro ambiente

Mejor crear los archivos de nuevo con los pasos 3–5. Si se usa `tar`, excluir el `.env` y revisarlo antes de moverlo:

```bash
sudo tar --exclude='.env' -czf ~/db-compose-<proy>.tar.gz -C /opt db-compose/<proy>
tar -tzvf ~/db-compose-<proy>.tar.gz
```
