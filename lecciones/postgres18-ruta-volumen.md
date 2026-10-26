# Postgres 18 no arranca con el volumen en `/var/lib/postgresql/data`

**Síntoma:** al levantar `postgres:18` por primera vez:

```
Error: in 18+, these Docker images are configured to store database data in a format which is compatible with "pg_ctlcluster" ... there appears to be PostgreSQL data in /var/lib/postgresql/data (unused mount/volume)
```

**Causa:** desde la 18, la imagen organiza los datos por versión mayor dentro de `/var/lib/postgresql`.

**Solución:** montar el volumen en `/var/lib/postgresql`:

```yaml
volumes:
  - /opt/databases/<proy>-postgres-<amb>:/var/lib/postgresql
```

**Para no repetirlo:** el compose de plantilla ya lo trae así; revisar compose viejos copiados de versiones anteriores.
