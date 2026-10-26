# Variables CI/CD

## Dónde va cada cosa

| Nivel | Qué |
|---|---|
| Grupo | `SSH_PRIVATE_KEY` (File), certificado y llave SSL (File, la llave Protected), `SRV_<AMB>_<ROLE>_HOST`, URLs de herramientas (SonarQube, DefectDojo), tokens de herramientas (Masked), red Docker, nombre del contenedor Nginx |
| Proyecto | `SERVER_ROLE`, `DNS_NAME`, `ENV`, `REPLICAS`, `SERVICE_PATH`, `DEPLOY_WAIT_TIME`, `DAST_TARGET_URL` |

## Reglas

- `ENV` es tipo **File**, uno por environment (scope `dev`, `uat`, `pprod`, `prd`). No usar un único `ENV` con scope All para varios ambientes.
- `SERVER_ROLE` puede ir con scope All si es igual en todos los ambientes.
- Una variable "All (default)" con el valor de PRD es un error frecuente: usar scope explícito.
- Variables de build (Vite, `NEXT_PUBLIC_*`) quedan dentro de la imagen: cambiarlas obliga a recompilar, no basta redesplegar.
- `SSH_PRIVATE_KEY` como File se carga con `ssh-add "$SSH_PRIVATE_KEY"`, no con `echo | ssh-add`.
- Protected solo llega a ramas protegidas: si `develop` no está protegida, sus jobs no ven esas variables.

## Secretos compartidos entre apps

- Algunas claves deben coincidir entre apps del mismo ambiente (ej. el `AUDIT_SYSTEM_KEY` del cliente = `SYSTEM_KEY` del colector de auditoría; la clave M2M del servicio de identidad = la que reciben los clientes). Mantener una tabla única por ambiente con qué app usa cada una y con qué nombre de variable.
- **Generar secretos nuevos por ambiente, nunca copiar los de PRD.** Si la cookie de sesión vive en el dominio padre (`.ejemplo.cl`) y dos ambientes comparten `JWT_SECRET`, un token de preprod es válido en producción.
- Si una variable se rechaza silenciosamente (auditoría sin sesión que no registra nada), comparar la clave del cliente con la del servidor antes de buscar en el código.
