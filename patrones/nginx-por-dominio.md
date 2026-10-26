# Nginx en contenedor, una carpeta por dominio

## Estructura

```
/opt/nginx/
  certs/                      cert + llave (llave 600)
  conf.d/<DNS_NAME>/
    server.conf               80 → 301 HTTPS; 443 con el certificado
    upstreams/<SERVICE>-<ENV>-upstream.conf
    locations/<SERVICE>-<ENV>-location.conf
  disabled/                   dominios sacados de servicio (no borrados)
  retirados/                  dominios dados de baja
```

- Dueño de `conf.d/`: el usuario técnico de deploy (sin sudo genérico).
- Nginx y las apps en la misma red Docker; las apps se resuelven por nombre de contenedor.
- `server.conf` con `http2 on;`.
- Redirect `www` → dominio en una carpeta aparte `www.<DNS_NAME>/` con solo un `server.conf` `301`.

## Reglas

- **El deploy de una app solo reescribe su propio archivo.** Archivos manuales (redirecciones, proxys a sitios legacy) en las mismas carpetas sobreviven a los deploys. Dejarlos documentados y respaldados aparte, porque cualquier job de limpieza genérico sí podría borrarlos.
- **Después de crear un server block, recargar** (`nginx -s reload`). `nginx -t` OK no significa que Nginx esté usando la configuración.
- `nginx -t` valida **todo** el servidor: un upstream hacia un contenedor que ya no existe bloquea el deploy de cualquier otra app. Al apagar apps, mover su dominio a `disabled/` en vez de dejar el upstream colgando.
- Upstreams hacia servidores fuera de Docker: usar IP, no nombre DNS, para que `nginx -t` no dependa de la resolución.
- Dominios fuera del wildcard de la organización: certificado propio por dominio (`-c` / `-k` en el script de creación).
- Un wildcard `*.ejemplo.cl` normalmente incluye la raíz `ejemplo.cl` como SAN: revisarlo con `openssl x509 -noout -text` antes de pedir otro certificado.

## SPA (React)

`index.html` es el único archivo sin hash. Sin `Cache-Control` el navegador aplica caché heurística y los usuarios ven la versión anterior después de cada deploy:

```nginx
location = /index.html {
    add_header Cache-Control "no-cache";
}
```

Los bundles con hash pueden ir con caché larga.
