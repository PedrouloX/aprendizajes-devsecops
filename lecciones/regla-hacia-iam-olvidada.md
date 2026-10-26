# La app te devuelve al portal a los pocos segundos

**Síntoma:** el usuario entra a una app desde el portal de aplicaciones (login centralizado) y a los pocos segundos vuelve al dashboard del portal, sin error visible. Pasó en QA y se repitió en DEV con la misma app.

**Causa:** la API de la app valida la sesión **server-side** contra el servicio de identidad. Faltaba la regla `servidor de la app → servicio de identidad TCP 443`. La llamada se quedaba colgada, la API daba la sesión por inválida y redirigía.

**Diagnóstico:**

1. DNS descartado: una sola IP correcta (`nslookup`, `getent hosts`).
2. Desde el servidor de la app: `nc -zv -w5 <ip-iam> 443` → `TIMEOUT`; `curl` → `000` tras más de 2 minutos (tráfico descartado).
3. Comparar con el ambiente donde funciona: ahí la regla existía.

**Mismo patrón, otra regla:** el worker de la misma app no podía llegar al servicio de notificaciones; las notificaciones no salían hacía semanas en dos ambientes y nadie lo había notado (el job registraba fallas silenciosas).

**Para no repetirlo:** toda app que usa el login centralizado necesita desde el día uno:

- app → servicio de identidad TCP 443
- app → servicio de notificaciones TCP 443 (si notifica)
- app → BDD (si está en otro servidor)

Dejarlas como checklist del proyecto para el siguiente ambiente y validarlas desde el host **y desde el contenedor** antes de entregar.
