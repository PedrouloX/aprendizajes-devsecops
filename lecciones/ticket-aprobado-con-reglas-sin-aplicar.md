# Un ticket de reglas cerrado con reglas sin aplicar

**Contexto:** máquina nueva de tres servidores; se pidieron 15 reglas en una planilla adjunta (una fila por puerto). El ticket se dio por resuelto.

**Síntoma:** días después, el deploy y la instalación de monitores fallaban con `ssh: connect to host ... port 22: Operation timed out`. Al revalidar el set completo faltaban tres reglas de runners → TCP 22 y una de la API hacia el servicio de notificaciones.

**Lección:** antes de aprobar o cerrar un ticket de red, validar **cada** regla desde su origen (incluido el 22 desde el servidor de runners), no solo las que se van a usar primero. Dejar el resultado por regla en la documentación.

**Formato de solicitud que funcionó:**

- Glosa corta en texto plano, sin tablas en el cuerpo.
- DNS y firewall en tickets separados.
- Planilla adjunta con una fila por origen/destino/puerto.
- Ninguna regla sin una prueba de conectividad fallida que la justifique.
