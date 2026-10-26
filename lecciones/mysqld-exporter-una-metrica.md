# `mysqld_exporter` solo exponía `mysql_up`

**Síntoma:** el exporter respondía, pero con una sola métrica (`mysql_up`) en vez de cientos. Los paneles del dashboard de MySQL quedaban vacíos.

**Causa 1:** el usuario `exporter` tenía un plugin de autenticación que la versión del exporter ya no aceptaba.

```sql
ALTER USER 'exporter'@'%' IDENTIFIED WITH mysql_native_password BY '<password>';
```

(o `caching_sha2_password`, según la versión del exporter y del motor: revisar el log del exporter).

**Causa 2:** el usuario solo tenía `PROCESS, REPLICATION CLIENT` y `SELECT` sobre `performance_schema`. El collector `info_schema.tables` (tamaño de la base) necesita leer las tablas reales:

```sql
GRANT SELECT ON <base>.* TO 'exporter'@'%';
```

**Para no repetirlo:** el tipo de usuario `MONITORING` del pipeline de BDD exige la base de destino y otorga ese `SELECT`. Validar con `curl http://localhost:9104/metrics | grep -c '^mysql_'` después de crear el usuario.
