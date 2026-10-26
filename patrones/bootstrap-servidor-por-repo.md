# Bootstrap de servidor con un repo por servidor

Variante más simple que el sistema de pipelines: cada servidor tiene su repo en `infraestructura/servidores/<servidor>` y un repo base `inicio-maquina` con el `provision.sh` genérico.

## provision.sh (base)

1. Actualizar paquetes.
2. Instalar Docker CE + plugin Compose, `systemctl enable docker --now`.
3. `daemon.json` con rotación de logs (`json-file`, `max-size 10m`, `max-file 3`).
4. Crear usuario técnico de deploy y agregarlo al grupo `docker`.

El repo base se incluye en cada repo de servidor con `include: project:` (habilitar job token access para el grupo).

## Jobs por servidor

Todos con `needs: []`, independientes: `provision`, `bootstrap`, `deploy`, `restart`, `restart:nginx`, `stop`, `reset:volumes`.

## Orden del bootstrap (no alterar)

```
1. SSH → mkdir + chown        directorios con permisos correctos
2. scp → copiar archivos       recién cuando existen los directorios
3. SSH → sed + docker compose up
```

**Nunca levantar contenedores antes de asignar permisos.** Si Postgres/MySQL inicializa con el dueño equivocado, el volumen queda inservible y hay que hacer `reset:volumes`.

```bash
sudo mkdir -p /opt/<srv>/postgres/data /opt/<srv>/nginx/conf.d
sudo chown -R <operador>:<operador> /opt/<srv>
sudo chown -R 999:999 /opt/<srv>/postgres/data
```

## Secretos

`.env.template` en el repo con placeholders; el pipeline los reemplaza desde variables CI/CD:

```bash
sed -e "s/PLACEHOLDER_USER/$POSTGRES_USER/" \
    -e "s/PLACEHOLDER_PASSWORD/$POSTGRES_PASSWORD/" \
    /tmp/.env.template > /opt/<srv>/.env
```

Variables por repo: `SSH_PRIVATE_KEY` (File), `SSH_USER`, `TARGET_HOST`.

## Scripts base para máquinas de apps

Para aprovisionar a mano por SSH (subidos por SFTP y ejecutados en orden):

| Script | Qué hace |
|---|---|
| `01-provision.sh` | Docker, usuario de deploy, `daemon.json` |
| `02-nginx-install.sh` | Contenedor Nginx + red Docker de apps |
| `03-create-apps-dir.sh` | `/opt/apps/logs` y `/opt/monitores` con dueño el usuario de deploy |
| `04-nginx-create-server.sh -d <dominio>` | Carpeta y `server.conf` del dominio |

En máquinas solo de BDD: `01` y `03`, crear la red Docker a mano y no instalar Nginx. `02` crea Nginx sin `mem_limit`: aplicarlo después con `docker update --memory 256m --memory-swap 256m nginx`.
