# Reglas de red para una máquina nueva

Arquitectura de referencia: una VPC por ambiente (DEV, QA, PREPROD, PRD) y una VPC STAFF compartida con GitLab, runners y observabilidad. Acceso de operadores por VPN.

## Set estándar a revisar con cada máquina

| Desde | Hacia | Puerto | Para qué |
|---|---|---|---|
| Runners (STAFF) | máquina | TCP 22 | Deploy por SSH |
| Observabilidad (STAFF) | máquina | TCP 9200, 9080 (+ 9104 MySQL / 9187 Postgres si hay exporter) | Scrape de node-exporter, cAdvisor, exporter de BDD |
| máquina | Observabilidad (STAFF) | TCP 4317 | otel-agent → gateway |
| VPN | máquina | TCP 80, 443 | Acceso web |
| VPN | máquina BDD | puertos de BDD publicados | Cliente de BDD de los devs |
| máquina (API) | servicio de identidad del ambiente | TCP 443 | Validación de sesión server-side |
| máquina (API) | servicio de notificaciones | TCP 443 | Envío de notificaciones |
| máquina (API) | máquina BDD | puertos de BDD | Solo si la BDD está en otro servidor |

No hacen falta `Observabilidad → máquina TCP 4317` ni `máquina → Observabilidad TCP 443` (salvo que la app consulte la API de Grafana).

## Cómo trabajar

1. **Probar desde el origen real** y pedir solo las reglas que fallan. Confirmar antes que el servicio escucha en destino (`ss -tlnp`).
2. Interpretar el resultado:
   - `TIMEOUT` → la regla no existe (tráfico descartado).
   - `REFUSED` → la regla existe y no hay nada escuchando (normal antes de levantar monitores o Nginx).
3. **Entre servidores de la misma subred también hace falta regla** (security groups, no routing).
4. Las reglas internas suelen ser **por puerto**: cada puerto nuevo de la BDD necesita su propia regla.
5. Dentro del mismo host, por nombre de contenedor en la red Docker: sin regla.
6. **Antes de aprobar/cerrar un ticket, validar el set completo** desde cada origen. Un ticket se dio por cerrado con tres reglas sin aplicar y se descubrió días después por un deploy que fallaba.
7. Cuando una solución pasa a otro ambiente, pedir desde el inicio las reglas que faltaron en el ambiente anterior. Dejarlas escritas como checklist del proyecto.

## Pre producción

Si pre producción replica PRD, conviene que cada servidor tenga el mismo último octeto que su equivalente (`10.0.x.11` ↔ `10.0.y.11`). Toda solicitud nueva (DNS, firewall, CORS, redirect URIs) debe incluir pre producción aunque el equipo del proyecto todavía no lo considere.
