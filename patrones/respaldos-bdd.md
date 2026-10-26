# Respaldos de bases de datos a S3

## Componentes

- Un bucket S3 dedicado, acceso público bloqueado, separado por prefijo de ambiente:
  `s3://<bucket>/<ambiente>/<db>/<db>_<fecha>_<hora>.sql.gz`
- Usuario IAM propio del repo de respaldos con permisos R/W/L/D solo sobre ese bucket.
- Pipeline con acciones `backup`, `restore`, `list-backups`, `delete-backup`, `cleanup-retention`.
- Schedules en GitLab: backup horario y limpieza diaria (03:00) por base.

## Decisiones

- **Usuario de respaldo fijo `backup_reader`**, de solo lectura, en el código (no como variable ni en el formulario):
  - MySQL: `SELECT, SHOW VIEW, TRIGGER, EVENT, LOCK TABLES` (cubre `mysqldump --single-transaction` con vistas, triggers y eventos)
  - Postgres: `CONNECT` + `pg_read_all_data`
- **`restore` pide la contraseña de root en cada corrida** (no se guarda) y exige `CONFIRM_DESTRUCTIVE=yes`.
- **Retención por script** (`aws s3api list-objects-v2` + filtro por fecha) y no lifecycle policy de S3, para cambiarla sin pedir nada a infraestructura.
- `pg_dump` con `--clean --if-exists` (mysqldump ya trae `DROP TABLE IF EXISTS`).
- Un servidor apagado hace fallar esa corrida puntual, sin efecto adverso: aceptable en ambientes que se apagan de noche.
- Frecuencia: cada 30 min con retención de 30 días son ~1.440 copias por base; el costo de S3 no es el problema, sí la carga de dumps continuos sobre producción.

## Pendientes de diseño que vale la pena resolver desde el inicio

- La password de `backup_reader` la crea el repo de lifecycle y la usa el de respaldos: que el primero la escriba directo en el segundo, en vez de copiarla a mano.
- No dejar el rol `postgres` fijo en el código: hay motores donde no existe.
