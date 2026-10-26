# Postgres acepta cualquier password por localhost

**Síntoma:** se "validó" la password de un rol con `docker exec ... psql -h 127.0.0.1` y funcionó, pero la app no podía entrar. En otro caso, una password documentada como root no correspondía a ningún rol y nadie lo había notado.

**Causa:** la imagen oficial de Postgres trae en `pg_hba.conf` reglas `trust` para el socket local, `127.0.0.1/32` y `::1/128`. Solo `host all all all` exige `scram-sha-256`. Por localhost cualquier password entra.

**Solución:** validar por la IP del contenedor (o la IP y puerto publicados) y siempre con control negativo:

```bash
sudo docker exec -e PGPASSWORD='<password>' <contenedor> sh -c 'psql -h "$(hostname -i)" -U <usuario> -d <base> -tAc "SELECT current_user"'
sudo docker exec -e PGPASSWORD='incorrecta' <contenedor> sh -c 'psql -h "$(hostname -i)" -U <usuario> -d <base> -tAc "SELECT 1"'
```

La segunda debe fallar con `password authentication failed`.

**Relacionado:** un pipeline que hace `psql -U postgres` por socket local tampoco usa la password que recibe, y falla en motores donde el rol `postgres` no existe.
