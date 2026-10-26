# MySQL: tres trampas de configuración

## `lower_case_table_names` solo al inicializar

Se fija al crear el datadir y no se puede cambiar después. Si el proyecto lo necesita (ej. código que mezcla mayúsculas), decidirlo antes de levantar la instancia. Con `=1` la base `APP_DB` se guarda como `app_db` y ambos nombres funcionan. Para cambiarlo: recrear la instancia con la base vacía.

## Dos `root` con passwords distintas

`root@%` (por red) y `root@localhost` (lo que usa `docker exec ... mysql -uroot`) son usuarios distintos y pueden tener password distinta. Si `docker exec` da `Access denied` con la password documentada, conectar por TCP:

```bash
docker exec -it <contenedor> mysql -h <ip-host> -P <puerto> -uroot -p
```

Para que root no sea accesible por red, dejar solo `root@localhost` y operar con `docker exec`. Control negativo: desde la red debe dar `Access denied for user 'root'@'<ip>'`.

## Exporter con plugin de autenticación viejo

Ver `mysqld-exporter-una-metrica.md`.

## Buffer pool en servidor compartido

En un servidor con varios motores y sin swap, acotar `innodb_buffer_pool_size` explícitamente y dejarlo acordado con el dueño del proyecto. El default o un valor grande puede tumbar el servidor completo por memoria (pasó, y terminó en una máquina dedicada para esa base).
