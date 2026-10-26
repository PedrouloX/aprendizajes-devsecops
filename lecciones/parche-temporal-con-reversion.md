# Todo parche temporal con su checklist de reversión

**Contexto:** para que un sitio funcionara mientras llegaba su certificado, se agregó al template de deploy una opción `EXPOSE_PORTS=true` (publicar el puerto del contenedor al host) y el frontend apuntó al backend por `IP:puerto`. Meses después el certificado no había llegado y el parche seguía en el template compartido.

**Lo que funcionó:** escribir en el mismo momento del parche la lista exacta para revertirlo:

1. Quitar el bloque del script de deploy.
2. Quitar el `export` de la variable en el script del orquestador.
3. Borrar la variable del repo.
4. Cambiar la URL del consumidor (sin puerto).
5. Copiar el certificado al servidor.
6. Recrear el `server.conf` con SSL.

**Regla:** un parche temporal en algo compartido (template, script de todos) entra solo con:

- Variable que lo activa explícitamente (default apagado).
- Checklist de reversión escrito.
- Dueño del pendiente que lo destraba (quién trae el certificado, la regla, etc.).
