# Detectar upstreams de Nginx hacia contenedores que ya no existen

Síntoma: `nginx -t` falla con `host not found in upstream "<contenedor>:80"` y bloquea el deploy de **cualquier** app del servidor. Señal típica: el paso `nginx -t` demora ~5 s (timeout de resolución DNS).

## Ver el error real

```bash
sudo docker exec nginx nginx -t
```

## Upstreams con contenedor caído

```bash
for h in $(sudo grep -rhoP 'server\s+\K[^:;\s]+' /opt/nginx/conf.d/*/upstreams/ | sort -u); do sudo docker ps --format '{{.Names}}' | grep -qx "$h" && echo "OK     $h" || echo "CAIDO  $h"; done
```

## Qué dominios contienen upstreams de un ambiente (ej. `-dev-`)

```bash
for d in $(sudo grep -rlP 'server\s+[^:;\s]+-dev-\d+' /opt/nginx/conf.d/*/upstreams/ | xargs -n1 dirname | xargs -n1 dirname | sort -u); do echo "== $d"; sudo grep -rhoP 'server\s+\K[^:;\s]+' "$d/upstreams/"; done
```

## Deshabilitar (mover, no borrar)

```bash
sudo mkdir -p /opt/nginx/disabled
sudo mv /opt/nginx/conf.d/<DNS_NAME> /opt/nginx/disabled/
sudo docker exec nginx nginx -t && sudo docker exec nginx nginx -s reload
```

## Restaurar

```bash
sudo mv /opt/nginx/disabled/<DNS_NAME> /opt/nginx/conf.d/ && sudo docker exec nginx nginx -t && sudo docker exec nginx nginx -s reload
```

## Locations duplicadas

```bash
sudo grep -rn 'location' /opt/nginx/conf.d/<DNS_NAME>/locations/
```

Si varias apps tienen `location /` en el mismo dominio, mover las sobrantes a una carpeta de respaldo fuera de `conf.d` y pedir que cada app defina su `SERVICE_PATH`.

Archivos de swap de editor (`.*.swp`) no los carga Nginx (no terminan en `.conf`), pero conviene limpiarlos.
