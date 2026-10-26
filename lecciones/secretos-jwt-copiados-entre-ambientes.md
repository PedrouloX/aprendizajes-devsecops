# Copiar los secretos de producción a pre producción

**Contexto:** al armar el `ENV` de pre producción a partir del de producción, lo cómodo era copiar todo, incluidos `JWT_SECRET` y las claves entre servicios.

**Problema:** la cookie de sesión vive en el dominio padre (`.ejemplo.cl`) en todos los ambientes. Con el mismo `JWT_SECRET`, un token emitido en pre producción es válido en producción.

**Decisión:** generar secretos nuevos para cada ambiente:

```bash
openssl rand -hex 64   # JWT
openssl rand -hex 48   # claves de servicio
```

Escribirlos directo en las variables CI/CD y dejar un registro de qué apps comparten cada uno (sin el valor en documentos públicos).

**Lo mismo aplica** a credenciales de servicios externos: si se reutiliza una (ej. correo de producción en preprod), poner una protección explícita en la app (`EMAIL_ENABLED=false` en el colector) y dejar la decisión escrita.
