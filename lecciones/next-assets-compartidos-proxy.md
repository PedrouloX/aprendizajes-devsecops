# Dos apps Next.js bajo el mismo dominio pelean por `/_next/`

**Contexto:** el sitio principal (Next.js) y unos micrositios (también Next.js, en otro servidor) se publicaron bajo el mismo dominio por proxy de Nginx (`/micrositio/` → servidor de micrositios). Un micrositio no tenía prefijo propio para sus assets y los pedía en `/_next/...` de la raíz, que es del sitio principal.

**Solución:** cadena de fallback por 404. Como los assets de Next.js llevan hash único, cada archivo solo existe en una app:

```nginx
location /_next/ {
    proxy_pass http://app-principal;
    proxy_intercept_errors on;
    error_page 404 = @micro_a;
}
location @micro_a {
    proxy_pass https://micrositios;
    proxy_set_header Host micro-a.ejemplo.cl;
    proxy_ssl_name micro-a.ejemplo.cl;
    proxy_ssl_server_name on;
    proxy_intercept_errors on;
    error_page 404 = @micro_b;
}
```

Lo mismo para `/assets/`, `/favicon/`, `manifest.json`.

**Descartado:** decidir por `Referer` (lo usaba el servidor legacy): también desvía los clics desde el micrositio hacia el sitio principal.

**Mejor a futuro:** que cada micrositio use `basePath`/`assetPrefix` propio, así no hace falta la cadena.

**Detalles:** upstream por IP y no por nombre (para que `nginx -t` no dependa del DNS), con `Host` y SNI propios de cada micrositio (vhost por SNI en el destino).
