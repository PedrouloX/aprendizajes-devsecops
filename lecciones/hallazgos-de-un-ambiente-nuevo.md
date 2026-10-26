# Hallazgos al levantar un ambiente completo desde cero

Al montar un ambiente DEV entero (siete servidores, quince proyectos) aparecieron problemas que en el ambiente viejo estaban escondidos. Vale la pena revisar estos puntos en cualquier ambiente nuevo:

- **Egress a Docker Hub:** los `docker pull` fallaban porque la imagen se descarga desde un CDN en S3, no desde `docker.io`. Pedir la salida al dominio del CDN, no solo a Docker Hub.
- **DNS con dos registros A:** un dominio tenía la IP nueva y la del ambiente anterior. El navegador caía al azar en uno u otro. Validar que cada dominio tenga un solo registro.
- **Cuenta de administrador como credencial de app:** dos APIs se conectaban con una cuenta personal con `ALL PRIVILEGES ON *.*`. Reemplazadas por usuarios dedicados con permisos solo sobre su base, en todos los ambientes.
- **Redis sin password ni persistencia:** corregido con `requirepass` + AOF en todos.
- **Remitente de correo (SES):** SES verifica por **dirección exacta**, no por dominio. Un remitente nuevo en el mismo dominio dio `AccessDenied`. Pedir la verificación de la dirección exacta para cada ambiente.
- **El sistema de pipelines no alcanzaba la cuenta AWS del ambiente nuevo** (credenciales del runner limitadas a otra cuenta). Se aprovisionó con los scripts base por SSH.
- **Apps que no existen en producción** (servicios de streaming, apps en desarrollo): decidir explícitamente si van al espejo y dejarlo escrito.
