# Redirecciones al pasar un sitio nuevo al dominio principal

## Una regla general de redirect redirigió de más

En el servidor legacy se agregó una regla que mandaba toda página pública al dominio nuevo, con varias exclusiones. Esa misma noche redirigió páginas que debían quedarse en el legacy (accesos internos). Se comentó la regla general y quedó solo la de un prefijo acotado.

**Lección:** redirigir **listas explícitas** de rutas, no "todo menos X". Una lista de exclusiones nunca está completa.

## 302 vs 301

- `301` queda en la caché del navegador: si después se corrige, quien ya lo recibió sigue redirigido hasta limpiar caché.
- Para rutas que podrían cambiar de destino más adelante, usar `302` aunque sea "definitivo".
- `301` solo para lo que de verdad no cambia (ej. `www` → raíz).
- Probar siempre en incógnito.

## Loops

Si el sitio nuevo devuelve ciertas rutas al legacy (`/privado/*` → legacy) y el legacy redirige "todo" al sitio nuevo, se arma un loop. Excluir esas rutas en el legacy.

## Las redirecciones que no están en el servidor web

Algunas venían del propio CMS (páginas tipo enlace). Antes de buscar una regla en Apache/Nginx, revisar si la responde la aplicación (cookie de sesión del CMS en la respuesta).

## Respaldos

Un respaldo del archivo por cada cambio, con hora, y verificar con `diff` que los servidores balanceados queden idénticos. En RHEL `cp` como root suele ser alias de `cp -i`: usar `\cp` en scripts.
