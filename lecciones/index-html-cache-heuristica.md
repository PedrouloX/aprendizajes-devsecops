# Después de cada deploy los usuarios seguían viendo la versión anterior

**Síntoma:** en una SPA React, tras desplegar, los usuarios veían la versión vieja hasta limpiar caché.

**Causa:** el Nginx interno de la imagen no ponía `Cache-Control` a `index.html`. Sin esa cabecera el navegador aplica **caché heurística** (~10 % del tiempo desde `Last-Modified`) y reusa el archivo sin consultar. `index.html` es el único sin hash y apunta a los bundles viejos (que tenían caché de un año).

**Solución:**

```nginx
location = /index.html {
    add_header Cache-Control "no-cache";
}
```

Con `no-cache` el navegador revalida siempre con `ETag` y recibe `304` si no cambió. Cubre `/` y el fallback del router. Ojo con el orden: si hay rewrites por `SERVICE_PATH`, deben seguir siendo la primera regla.

**Efecto residual:** quien ya tenía el `index.html` viejo en caché lo usa hasta que vence o hace Ctrl+F5. Pasa una sola vez.

**Verificar:**

```powershell
curl.exe -sI https://app.ejemplo.cl/ | findstr /i cache-control
```
