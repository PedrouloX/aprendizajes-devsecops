# Agregar `mem_limit` a un contenedor existente

## 1. Ver qué tiene límite

```bash
sudo docker inspect -f '{{.Name}} {{.HostConfig.Memory}}' $(sudo docker ps -q)
```

`0` = sin límite. `536870912` = 512 MiB, `1073741824` = 1 GiB.

## 2. Medir antes de tocar

```bash
sudo docker stats --no-stream
free -h
```

Elegir un límite con margen sobre el consumo actual y que el total del servidor quepa en la RAM (sin swap, el OOM killer elige él).

## 3. Respaldar y editar el compose

```bash
cd /opt/<ruta-compose>
sudo cp docker-compose.yml docker-compose.yml.bak-$(date +%Y%m%d-%H%M)
```

Editar **solo insertando líneas** (ej. `mem_limit` después de `container_name`). Si el compose tiene passwords en línea (`--requirepass`), no reemplazar esa línea con `sed`.

```bash
sudo diff docker-compose.yml.bak-* docker-compose.yml
sudo docker compose config --quiet && echo OK
```

## 4. Recrear solo ese servicio

```bash
sudo docker compose up -d <servicio>
```

No `docker compose up -d` a secas si hay otros servicios que no se quieren tocar.

## 5. Validar

```bash
sudo docker inspect -f '{{.Name}} {{.HostConfig.Memory}}' <contenedor>
sudo docker exec <redis> redis-cli -a '<password>' --no-auth-warning config get maxmemory
```

Y el health del servicio (`pg_isready`, `PING`, `/health`). Es esperable un corte breve en las apps conectadas (ej. Prisma registra `E57P01 terminating connection`).

## Valores de referencia

| Servicio | `mem_limit` | Extra |
|---|---|---|
| Postgres de app | 1g | — |
| Redis | 512m | `--maxmemory 384mb` |
| MeiliSearch | 512m–1g | `MEILI_MAX_INDEXING_MEMORY` a la mitad |
| MongoDB | 1g | `--wiredTigerCacheSizeGB 0.5` |
| Nginx | 256m | `docker update` si lo creó un script sin límite |

## Contenedores que no son de compose

```bash
sudo docker update --memory 256m --memory-swap 256m <contenedor>
```

Se mantiene tras reinicios, se pierde si se vuelve a crear el contenedor.
