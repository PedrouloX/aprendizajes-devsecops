# El Redis "del servicio X" lo usaban otros tres

**Contexto:** había que vaciar el Redis del servicio de identidad en QA (`FLUSHALL`). Por su nombre se asumía exclusivo.

**Hallazgo:** con `CLIENT LIST` aparecieron conexiones desde otro servidor: el servicio de notificaciones (colas Bull + Pub/Sub) y la API del portal (colas y caché). El nombre del contenedor no dice quién lo usa.

**Antes de un `FLUSHALL`:**

```bash
redis-cli -a '<pass>' --no-auth-warning CLIENT LIST | awk '{print $2}' | sort | uniq -c
redis-cli -a '<pass>' --no-auth-warning --scan --pattern 'bull:*' | head
```

Revisar que las colas no tengan jobs pendientes (`wait`, `delayed`, `repeat`, `active`). Si las hay, coordinar con los dueños.

**Efecto esperado:** los clientes siguen conectados y los workers recrean sus llaves de control al instante; los usuarios con sesión activa deben volver a iniciar sesión.

**Para no repetirlo:** documentar los consumidores de cada Redis en el inventario, no solo el "dueño".
