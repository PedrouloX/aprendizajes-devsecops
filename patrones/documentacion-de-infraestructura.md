# Documentación de infraestructura (docs de contexto)

## Estructura que funcionó

- Un `contexto-sesion.md` corto como índice: estado general, decisiones recientes y la lista de docs de referencia con su fecha de actualización.
- Un `referencia-<tema>.md` por tema: infraestructura (servidores, DNS, firewall), bases de datos, CI/CD, observabilidad, respaldos.
- **Topología de red: un archivo por ambiente.** Un solo archivo con todos los ambientes se desactualizó y quedó obsoleto sin que nadie lo notara.
- Runbooks aparte de las referencias.
- Versiones reemplazadas a una carpeta `historico/` con sufijo de fecha; la vigente conserva el nombre.

## Reglas

- Cada cambio actualiza la fecha del encabezado del doc **y** su línea en el índice.
- Fuente de verdad para tipos de instancia y especificaciones: el propio servidor (metadata), no la planilla. Ver `runbooks/verificar-tipo-instancia-imdsv2.md`.
- Anotar el origen de cada dato (prueba con `nc`, `docker inspect`, solicitud, captura) y la fecha.
- Las reglas de red se registran **con la prueba** que las validó, no con el ticket.
- Sin nombres de personas: solo roles.
- Docs internos de contexto vs portal publicado: en el portal nunca secretos.
- Escribir documentos sin reescribirlos a mano: aplicar cambios con reemplazos puntuales y revisar el `diff` antes de subir (una reescritura completa desde la salida de una herramienta puede venir truncada y borrar contenido).
