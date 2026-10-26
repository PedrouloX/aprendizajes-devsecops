# Dar de baja una base de datos (Postgres)

Primero apagar, después borrar. Así se puede revertir si aparece un consumidor que nadie conocía.

## 1. Confirmar que nadie la usa

```sql
SELECT datname, usename, client_addr, application_name, state
FROM pg_stat_activity WHERE datname = '<base>';
```

Revisar también la `DATABASE_URL` de las apps que podrían usarla (`docker exec <app> printenv DATABASE_URL`). Una sesión de un cliente de BDD (pgAdmin) desde una IP desconocida no es una app, pero hay que identificarla.

## 2. Respaldar y verificar el respaldo

```bash
sudo docker exec <contenedor> pg_dump -U <root> -Fc <base> > ~/migracion/<base>-<AMB>-$(date +%Y%m%d).dump
sudo docker exec -i <contenedor> pg_restore -l < ~/migracion/<base>-<AMB>-*.dump | head
```

Para roles y permisos, si la restauración podría ir a otra instancia:

```bash
sudo docker exec <contenedor> pg_dumpall -U <root> --globals-only > ~/migracion/<instancia>_roles_$(date +%Y%m%d).sql
```

## 3. Apagar (reversible)

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '<base>' AND pid <> pg_backend_pid();
ALTER DATABASE "<base>" RENAME TO "<base>_obsoleta_<fecha>";
ALTER DATABASE "<base>_obsoleta_<fecha>" ALLOW_CONNECTIONS false;
```

Revertir: `ALLOW_CONNECTIONS true` y `RENAME` al nombre original.

## 4. Borrar (después de un tiempo sin reclamos)

```sql
DROP DATABASE "<base>_obsoleta_<fecha>" WITH (FORCE);
```

Revisar si quedaron roles sin uso (dueños de objetos solo en esa base) y eliminarlos.

## MySQL

Equivalente: `information_schema.processlist` sin conexiones a la base, `mysqldump --single-transaction --routines --triggers --events`, `DROP DATABASE`, y limpiar usuarios y grants huérfanos (los usuarios viven en la base `mysql` y no se van con el `DROP`).

## Documentar

Marcar la base como **obsoleta / no usar** en el inventario apenas se decide, aunque siga arriba. Una copia vieja sin marca terminó recibiendo un usuario nuevo y una regla de firewall por error.
