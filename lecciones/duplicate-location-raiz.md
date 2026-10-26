# Once apps en `location /` del mismo dominio

**Síntoma:** ninguna app del servidor podía desplegar: `nginx: [emerg] duplicate location "/" in .../<app>-dev-location.conf:2`.

**Causa:** se desplegaron once micro-frontends con `SERVICE_PATH` vacío en el mismo dominio donde ya había otra app en `/`. Cada deploy dejó su archivo de location aunque `nginx -t` fallaba. Nginx siguió sirviendo la última configuración válida, así que nadie lo notó hasta el siguiente deploy de otra app.

**Solución inmediata:** mover las locations sobrantes a una carpeta de respaldo fuera de `conf.d` y recargar.

**Solución de fondo (en el template de deploy):**

- Antes de escribir upstream y location, verificar que ninguna otra app use esa ruta en ese dominio. Si está tomada, abortar **sin escribir nada** y nombrar el archivo en conflicto. Falla solo la app mal configurada.
- Excluir el propio archivo de la app (un redeploy no choca consigo mismo).
- `location = /x/` y `location ~` no cuentan como conflicto (igual que Nginx).
- `nginx -t` bloqueante y con su salida en el log.

**Regla:** toda app que comparte dominio define `SERVICE_PATH` (y en Next.js el `basePath`).

**Lección del arreglo temporal:** se hizo `nginx -t` no bloqueante para destrabar el servidor; una versión así no debe quedar en `main` más de lo necesario.
