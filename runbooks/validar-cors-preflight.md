# Validar CORS con un preflight (PowerShell)

Después de agregar un origen a `CORS_ORIGINS` (variable `ENV` del repo + redeploy):

```powershell
curl.exe -s -o NUL -D - -X OPTIONS https://api.ejemplo.cl/<ruta-que-llegue-a-la-api> -H "Origin: https://app.ejemplo.cl" -H "Access-Control-Request-Method: GET"
```

Esperado: `204` con `Access-Control-Allow-Origin: https://app.ejemplo.cl`.

- Usar una ruta que Nginx envíe a la API (con su `SERVICE_PATH`/`API_PATH`). En `/` puede responder el propio Nginx con `405`.
- Probar también un origen que **no** esté en la lista: no debe devolver la cabecera. Algunas APIs responden `500` en vez de omitirla; no es un problema de seguridad pero conviene reportarlo.
- `curl.exe`, no `curl`, en PowerShell (el alias apunta a `Invoke-WebRequest`).

## Verificar cabeceras de caché

```powershell
curl.exe -sI https://app.ejemplo.cl/ | findstr /i cache-control
```
