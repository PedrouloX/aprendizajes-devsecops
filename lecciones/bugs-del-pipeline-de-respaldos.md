# Bugs encontrados al construir el pipeline de respaldos

## Restore de Postgres incompleto, sin error

`pg_dump` no incluye `DROP TABLE` por defecto. Al restaurar sobre una base con datos, el `COPY` choca con filas existentes y se descartan en silencio. Solución: `pg_dump --clean --if-exists` (mysqldump ya trae `DROP TABLE IF EXISTS`).

## Una acción que "terminaba OK" sin hacer nada

`delete-backup` reportaba éxito y no borraba. El bloque `delete-backup)` nunca se agregó al `case "$BACKUP_ACTION"`. En bash un `case` sin coincidencia no hace nada y sigue. Siempre cerrar con:

```bash
*) echo "Acción desconocida: $BACKUP_ACTION"; exit 1 ;;
```

## Credenciales pisadas entre ambientes

Las variables con credenciales se nombraban solo por `DB_NAME`. Crear la misma base en otro ambiente sobrescribió la password del primero. Solución: sufijo de ambiente en el nombre (`DB_PASSWORD_<db>_<amb>`) y `environment_scope` real en la variable.

## Ambiente de prueba que queda

Para probar se crearon bases `test_*`, usuarios y respaldos en S3. Anotar al crearlos qué hay que borrar al final.
