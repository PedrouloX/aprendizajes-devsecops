# Paso a producción de una app al dominio principal

Caso: una app nueva (Next.js) reemplaza al portal legacy en `ejemplo.cl`; el legacy queda en un subdominio.

## Capas y responsables

| Capa | Qué se configura |
|---|---|
| CDN / WAF | Publicación de `ejemplo.cl` y `www.ejemplo.cl` |
| Balanceador | Enrutamiento hacia el servidor front |
| Nginx del front | Dominio, redirect `www`, rutas especiales |
| Servidor legacy | Redirecciones de páginas públicas al dominio nuevo |

Coordinar la hora con todos: cada capa la hace un equipo distinto.

## Nginx del front

1. **Respaldar** `conf.d` completo antes de tocar nada (`tar` con fecha y hora en el home).
2. Crear el dominio con su pipeline/script (`dns-ssl` / `nginx-create-server`) y el redirect `www` → `301`.
3. Cambiar `DNS_NAME` de la app y desplegar.
4. **Retirar el dominio anterior moviendo su carpeta** fuera de `conf.d` (`/opt/nginx/retirados/<dominio>-<fecha>`). Si queda, el redeploy con el nuevo `DNS_NAME` da `duplicate upstream`.
5. Rutas especiales en archivos manuales dentro de la carpeta del dominio (sobreviven a los deploys):
   - Páginas que siguen en el legacy: `302` (no `301`, que queda en caché del navegador).
   - Micrositios: proxy con `Host` y SNI propios (`proxy_ssl_name`), upstream por IP.
6. Copia aparte de los archivos manuales y comando para restaurarlos.

## Actualizar en las apps

- `DNS_NAME`, `ENV` de la app, CORS de la API, redirect URIs del proveedor de identidad.

## Servidor legacy (Apache)

- Insertar el bloque nuevo **después** de las reglas existentes, sin modificar líneas previas; respaldo del archivo por cada cambio.
- Redirigir **una lista explícita** de rutas, con exclusiones para: páginas privadas (evita loops), URLs con UUID, raíz, login, recursos técnicos del CMS, acciones de portlets y rutas que ya atiende el mismo archivo.
- Si hay más de un servidor legacy, verificar con `diff` que el contenido quede idéntico (los nombres de archivo pueden diferir).

## Validaciones (tabla con resultado esperado)

| Prueba | Esperado |
|---|---|
| `https://ejemplo.cl/` | `200` de la app nueva |
| `https://www.ejemplo.cl/prueba` | `301` → `https://ejemplo.cl/prueba` |
| Ruta privada del legacy en el dominio nuevo | `302` al subdominio legacy, con query string |
| Micrositios | `200`, sin errores de assets |
| Legacy: raíz, login, URL con UUID | quedan en el legacy |

Probar en incógnito: un `301` recibido antes de una corrección queda en caché.

```powershell
curl.exe -kI --resolve www.ejemplo.cl:443:<ip-front> https://www.ejemplo.cl/prueba
```
