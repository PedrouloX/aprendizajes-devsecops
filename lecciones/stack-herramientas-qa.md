# Stack de herramientas QA en un solo servidor

## Postgres compartido sin conexiones

SonarQube y Squash TM compartían un Postgres con `max_connections=100` y mantenían ~40 conexiones permanentes entre los dos. Se subió a 200:

```yaml
command: postgres -c max_connections=200
```

## Integraciones por red interna

La integración Squash TM → MantisBT apuntaba a la URL pública (pasando por Nginx y TLS). Se cambió al nombre del contenedor en la red Docker (`http://mantis:80`). Si la integración se configuró por la UI y no hay forma de editarla, se puede cambiar en la tabla de servidores externos de la base de Squash.

## Herramienta detenida rompe Nginx

Con DefectDojo detenido por falta de uso, Nginx no podía reiniciar (no resolvía el upstream). Antes de reiniciar Nginx: levantar la herramienta o comentar su `server.conf`. Ver `nginx-t-upstream-sin-contenedor.md`.

## SonarQube

- `client_max_body_size 50M` en el Nginx del dominio de Sonar (los reportes grandes fallan con el default).
- Para que reciba cobertura en Vitest: `reporter: ['lcov', 'html']` en la configuración de cobertura del proyecto.

## MantisBT lento

Recarga la página completa en cada acción. El servidor respondía en ~50 ms; la demora percibida (2–5 s) era VPN + render. No optimizable desde infraestructura: dejarlo explicado para no seguir buscando.
