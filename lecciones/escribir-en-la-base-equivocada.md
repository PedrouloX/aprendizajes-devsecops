# Escribir en la base equivocada

**Síntoma:** un dev pidió usuarios de prueba; se cargaron, pero no pudo iniciar sesión y perdió un día de trabajo. En otro caso se creó un usuario y se pidió una regla de firewall hacia una copia obsoleta de una base.

**Causa:** en el mismo motor había dos bases con nombres parecidos (una vieja, remanente de cuando dos ambientes compartían servidor) y la copia antigua de otra base seguía arriba después de migrarla a un servidor nuevo. Se eligió por el nombre, sin confirmar cuál usaba la app. El dato correcto ya estaba documentado y no se leyó.

**Regla:** antes de cualquier `INSERT`/`UPDATE`/`DELETE`, de crear usuarios o de pedir reglas hacia una base:

1. Revisar el inventario: ubicación actual de la base.
2. Confirmar la base real en la app:
   ```bash
   sudo docker exec <app> printenv DATABASE_URL
   ```
3. Confirmar quién está conectado:
   ```sql
   SELECT datname, usename, client_addr, application_name FROM pg_stat_activity;
   ```
4. Si no se puede confirmar, no se escribe.

**Para no repetirlo:** marcar las copias viejas como **obsoleta / no usar** en el inventario el mismo día en que se migra, y apagarlas (`ALLOW_CONNECTIONS false`) en cuanto sea posible. Ver `runbooks/dar-de-baja-una-base.md`.
