# Migrar una base MySQL con vistas, funciones y procedimientos

## Antes

- Versión del motor destino **igual o mayor** que la de origen.
- `lower_case_table_names` se fija solo al inicializar el datadir: decidirlo antes de crear la instancia (si hay que cambiarlo, recrear con la base vacía).
- Los **definers** de vistas y rutinas deben existir en destino antes del restore.
- Los usuarios no viajan en el dump (viven en la base `mysql`). Migrarlos conservando la password:

```sql
SET print_identified_with_as_hex = ON;
SHOW CREATE USER 'usuario'@'%';
SHOW GRANTS FOR 'usuario'@'%';
```

## Dump

```bash
mysqldump -u root -p --single-transaction --routines --triggers --events <base> > <base>.sql
```

Sin `--routines` el destino queda sin funciones ni procedimientos, sin ningún error.

## Restore

Con binlog activo MySQL rechaza funciones sin `DETERMINISTIC` (ERROR 1418/1419):

```sql
SET GLOBAL log_bin_trust_function_creators = 1;
```

```bash
mysql -u root -p <base> < <base>.sql
```

Volver a `0` al terminar, salvo que los devs creen rutinas a diario; en ese caso dejarlo en el compose (`--log-bin-trust-function-creators=1`) en vez de dar `SUPER` o `SET_USER_ID`, y que creen las rutinas sin `DEFINER` o con `DEFINER=CURRENT_USER`.

## Vistas que el restore rechaza

ERROR 1166 por alias automáticos de más de 64 caracteres en vistas con `UNION`: recrearla a mano con alias explícitos en todos los `SELECT`, mismas columnas y mismo resultado. Dejar el `CREATE VIEW` corregido guardado: hay que repetirlo después de cada restore.

## Validar

Contar en origen y destino:

```sql
SELECT table_type, COUNT(*) FROM information_schema.tables WHERE table_schema = '<base>' GROUP BY table_type;
SELECT routine_type, COUNT(*) FROM information_schema.routines WHERE routine_schema = '<base>' GROUP BY routine_type;
```

Más conteos de filas de las tablas principales.

## Refrescar sin perder rutinas

Si el origen no tiene rutinas (migración anterior sin `--routines`) y el destino sí, refrescar datos **sin** `DROP DATABASE`.
