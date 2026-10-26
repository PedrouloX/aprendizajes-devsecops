# El compose quedó dentro del volumen de datos

**Síntoma:** MySQL no inicializa: `--initialize specified but the data directory has files in it`.

**Causa:** el `docker-compose.yml` estaba guardado en la misma carpeta que se montaba como `/var/lib/mysql`. Para el motor el datadir "no estaba vacío".

**Solución:** separar configuración y datos:

```
/opt/db-compose/<proy>/docker-compose.yml
/opt/databases/<proy>-mysql-<amb>/        ← solo esto se monta
```

**Variante relacionada:** si el volumen ya tiene datos de un intento anterior, el motor ignora las passwords nuevas del `.env` y las rechaza (usa las del primer arranque). Hay que vaciar el volumen (o un `reset-container`) antes de reprovisionar.
