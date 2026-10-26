# Un upstream huérfano bloquea el deploy de todo el servidor

**Síntoma:** el deploy de una app falló con solo `[ERROR] Fallo en el despliegue.` El paso `nginx -t` tardaba ~5 s.

**Causa:** cuando un ambiente compartía servidor con otro, sus deploys dejaron upstreams y locations en el Nginx. Al apagar esos contenedores, `nginx -t` (que valida **toda** la configuración) falló con `host not found in upstream "<app>-dev-1:80"`. Cualquier deploy del servidor quedó bloqueado. El script no imprimía la salida de `nginx -t`.

**Solución:** mover (no borrar) los dominios huérfanos a `/opt/nginx/disabled/` y recargar. Ver `runbooks/detectar-upstreams-nginx-caidos.md`.

**Variante:** en otro servidor, Nginx **no podía reiniciar** porque uno de sus upstreams era una herramienta detenida a propósito. Antes de reiniciar Nginx: levantar ese contenedor o comentar su `server.conf`.

**Para no repetirlo:**

- Al apagar o mover apps, limpiar su configuración de Nginx en el mismo cambio.
- Los scripts de deploy deben mostrar la salida de `nginx -t` cuando falla.
- Upstreams hacia servidores externos por IP, para no depender del DNS.
