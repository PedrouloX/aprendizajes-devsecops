# Los jobs `stop` del pipeline borraban configuración manual

**Contexto:** los orquestadores tenían jobs `stop:<ambiente>` (con `environment: action: stop`) que detenían los contenedores y limpiaban upstream y location de Nginx.

**Problema:** en producción había archivos de Nginx escritos a mano (proxys a micrositios, redirecciones) que dependían de esos upstreams. Un `stop` los habría dejado rotos, y el botón estaba a un clic en la UI de environments. Además, al cambiar la lista de dominios de una app, el `stop` posterior no limpia el dominio que salió.

**Decisión:** sacar los jobs `stop` de los pipelines. Detener contenedores queda para los administradores por SSH, a conciencia.

**Detalles técnicos que sí sirvieron mientras existieron:**

- Filtrar réplicas por prefijo `<app>-<env>-` en vez de iterar `REPLICAS` (si cambió el número de réplicas, iterar deja contenedores vivos).
- SSH inline con comillas dobles: un heredoc rompe el parseo YAML de GitLab.
- `stop:prd` sin variable de confirmación extra: ya está atado al pipeline del deploy.
- Recuperación: un push o merge a la rama vuelve a desplegar.
