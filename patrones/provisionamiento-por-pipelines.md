# Provisionamiento de servidores por pipelines

Reemplaza el "crear EC2 en consola + SSH + instalar a mano" por pipelines de GitLab, uno por responsabilidad. Cualquier persona con permisos llena un formulario (Run pipeline) y obtiene siempre el mismo resultado.

## Repos (uno por responsabilidad)

| # | Repo | Qué hace |
|---|---|---|
| 1 | `provision-servidor` | Crea la instancia (tipo, IP privada, disco, rol `APP` / `DATABASE` / `MIXED`). Abre un **Issue** con los datos del servidor y una checklist: el Issue es el expediente del servidor y los demás pipelines comentan en él |
| 2 | `admin-access-servidor` | Habilita acceso SSH usuario/contraseña para el equipo de infraestructura (la contraseña la define infraestructura, el pipeline no la conoce) |
| 3 | `configure-servidor` | Docker, usuario técnico de deploy con su clave pública, Nginx si el rol no es `DATABASE`, carpetas de apps |
| 4 | `dns-ssl` | Agrega un dominio a Nginx (wildcard, autofirmado, sin SSL o certificado propio; redirect `www` opcional). Se corre una vez por dominio |
| 5 | `deploy-observabilidad` | node-exporter, cAdvisor, otel-agent y registro en Prometheus |
| 6 | `lifecycle-servidor` | start/stop/restart/status, disco, contenedores, inspección de contenedor, prueba de conectividad TCP, SELinux, tipo de instancia |
| 7 | `provision-bdd` | Motor MySQL/Postgres/Redis/MeiliSearch como contenedor, base, usuario de app; credenciales a variables masked |
| 8 | `lifecycle-bdd` | Crear/borrar bases y usuarios (tipos `APP`, `READONLY`, `MONITORING`, `BACKUP`), `exec-sql`, reiniciar motor |
| 9 | `respaldo-bdd` | Backup/restore/list/delete/cleanup a S3 (ver `respaldos-bdd.md`) |

## Decisiones de diseño

- **Sin IP pública y sin SSH para crear la máquina**: la instancia nace con un instance profile de SSM y los primeros pasos corren por `aws ssm send-command`. Cada acción queda auditada en AWS.
- **Usuario de deploy sin `sudo` genérico**: solo sudoers acotados a lo que necesita (archivos de Nginx). Está en el grupo `docker`, no necesita `sudo` para Docker.
- **Sin acción `terminate`**: borrar una instancia es manual a propósito. Se apaga y se pide la baja.
- **Acciones destructivas** (`drop-*`, `reset-container`, `exec-sql`, `restore`) exigen `CONFIRM_DESTRUCTIVE=yes` en el `rules:` **del job**, nunca en `workflow:rules`.
- **Credenciales generadas → variables masked** en el propio repo, sufijadas por ambiente (`DB_PASSWORD_<db>_<amb>`), nunca en logs.
- **Las notificaciones al Issue validan la respuesta real de la API** antes de decir "listo".
- **Parches de diagnóstico**: está bien agregar acciones temporales durante un incidente, pero después limpiar. Conservar las genéricas (parametrizadas) y retirar las que tienen nombres de contenedor, bases o contraseñas en duro.

## Flujo para un servidor nuevo

1. Pedir a infraestructura IP, AMI vigente y reglas de red.
2. `provision-servidor` → `admin-access-servidor` → `configure-servidor`.
3. `dns-ssl` por cada dominio (rol APP) / `provision-bdd` por cada motor (rol DATABASE).
4. `deploy-observabilidad`.
5. Mantenimiento diario con `lifecycle-servidor` y `lifecycle-bdd`.

## Límites

- Si las credenciales AWS del runner solo alcanzan una cuenta, el sistema no sirve para otra cuenta (ej. DEV en cuenta separada). Ahí se usan los mismos scripts base por SSH (ver `bootstrap-servidor-por-repo.md` y los scripts `01`–`04`).
