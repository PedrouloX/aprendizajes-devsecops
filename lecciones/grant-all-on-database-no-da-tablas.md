# `GRANT ALL ON DATABASE` no da permisos sobre las tablas

**Síntoma:** los roles de aplicación conectaban pero cualquier consulta daba `permission denied for table ...`.

**Causa:** en Postgres `GRANT ALL PRIVILEGES ON DATABASE` solo da `CONNECT`, `CREATE` (schemas) y `TEMP`. Las tablas son objetos aparte. Además, las tablas que creen migraciones futuras con otro rol tampoco heredan permisos.

**Solución:**

```sql
GRANT USAGE, CREATE ON SCHEMA public TO <app>;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO <app>;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO <app>;
ALTER DEFAULT PRIVILEGES FOR ROLE <dueño> IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO <app>;
ALTER DEFAULT PRIVILEGES FOR ROLE <dueño> IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO <app>;
```

**Variante:** si las tablas quedaron de otro dueño (restore como superusuario), una migración Prisma con `ALTER TABLE` falla con `must be owner`. Restaurar como el rol de app (`pg_restore --no-owner --no-acl -U <app>`) o pasar el ownership (`ALTER TABLE ... OWNER TO <app>`). Los objetos de extensiones (ej. funciones de `btree_gist`) quedan del superusuario; un `CREATE EXTENSION` lo corre el superusuario.

**MySQL:** el permiso real que necesita la app se descubre con el primer `INSERT` que falla (`ERROR 1142 INSERT command denied`). Preguntar al dev si la app escribe antes de crear un usuario solo con `SELECT`.
