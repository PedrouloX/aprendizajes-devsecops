# Alta de una base en los respaldos automáticos

Requisito: la base levantada con el pipeline de aprovisionamiento (los repos derivan el nombre del contenedor del `DB_NAME`). Confirmarlo antes con `lifecycle-bdd` → `list-containers` sobre la IP (de paso valida que el runner entra).

1. `lifecycle-bdd` → `create-user`, `NEW_USER_ACCESS=BACKUP`, `ENGINE`, `DB_NAME`, `TARGET_IP`, `ENVIRONMENT`, `NEW_USER_TARGET_DB`, `ROOT_PASSWORD`. Crea `backup_reader` y guarda la password como `DB_PASSWORD_<db>_<amb>`.
2. Copiar ese valor al repo de respaldos como `BACKUP_PASSWORD_<db>` (masked).
3. `respaldo-bdd` → `backup` manual, luego `list-backups` y revisar el tamaño: un `.sql.gz` de pocos bytes es un dump vacío.
4. Crear dos schedules sobre la rama donde vive el pipeline:
   - `backup` cada hora
   - `cleanup-retention` diario a las 03:00 (solo necesita `ENVIRONMENT`, `DB_NAME`, `BACKUP_ACTION`)

## Usuario de monitoreo con el mismo pipeline

`create-user` con `NEW_USER_ACCESS=MONITORING`. En MySQL exige la base de destino para dar `SELECT` sobre ella (sin eso el exporter no puede calcular el tamaño de la base). Guardar su password en una variable distinta a la de `backup_reader`.

## Alternativa sin pipeline (dump manual entre ambientes)

Si el pipeline no cubre el caso (ej. traspaso puntual), correr el dump desde un equipo con VPN que llegue a ambos ambientes: entre VPC de distintos ambientes no hay ruta, por diseño. Ver `traspaso-datos-entre-ambientes.md`.
