# Traspaso de datos entre ambientes

Entre VPC de ambientes distintos normalmente no hay ruta. Hacerlo desde un equipo con VPN a ambos, o bajando el dump al PC y subiéndolo por SFTP.

## Postgres (base completa)

```bash
sudo docker exec <contenedor-origen> pg_dump -U <usuario> -Fc <base> > ~/<base>-origen.dump
```

En destino, restaurar **como el rol de aplicación**, para que las tablas queden con el dueño correcto:

```bash
sudo docker exec -i <contenedor-destino> pg_restore -U <proy>_app -d <base> --no-owner --no-acl --exit-on-error < ~/<base>-origen.dump
```

Si hay que reemplazar todo el contenido:

```sql
DROP DATABASE <base> WITH (FORCE);
CREATE DATABASE <base> OWNER <proy>_app;
```

## MySQL (solo datos, el esquema ya existe por migraciones)

```bash
mysqldump -h <ip-origen> -P <puerto> -u <usuario> -p --no-create-info --single-transaction --skip-triggers <base> tabla1 tabla2 > datos.sql
mysql -h <ip-destino> -P <puerto> -u <usuario> -p <base> < datos.sql
```

- Importar en orden de claves foráneas (primero las tablas referenciadas).
- En Windows, `mysql.exe` viene con MySQL Workbench aunque no esté en el PATH.

## Validar

- Conteo por tabla idéntico en origen y destino.
- Cero registros huérfanos en las tablas con FK.
- Dueño de las tablas correcto (`\dt` en Postgres).

## Antes y después

- Respaldar el destino antes de pisarlo, aunque esté casi vacío.
- Revisar que los datos de producción traídos a otro ambiente no arrastren URLs de producción (ej. tablas de aplicaciones registradas) y repetir esa corrección después de cada restore.
- Lo que es solo de producción (notificaciones enviadas, logs de usuarios) no se copia: solo esquema y catálogos.
- Borrar los dumps de los home cuando la migración se cierre.
