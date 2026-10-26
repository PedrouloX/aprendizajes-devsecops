# OpenAPI y portal de documentación

## Estándar (Definition of Done de APIs)

- Toda API activa tiene `openapi/openapi.json` en el repo.
- Job `validate:openapi` en el MR, con Spectral **versión fijada** (`@stoplight/spectral-cli@6.11.0`) y un ruleset propio sin `extends: "spectral:oas"` (incompatibilidad con `nimma` en esa versión).
- Empieza con `allow_failure: true` durante la adopción y pasa a bloqueante después.
- El skeleton de API trae el `openapi.json` base y un README con la estructura.

## Publicación

- Portal VitePress; rama publicada `uat`.
- Job `sync:openapi:<amb>` después del deploy: copia el `openapi.json` al repo del portal inyectando la versión de `package.json` y la URL del servidor del ambiente. Commit con `[skip ci]`.
- Como los commits del sync no disparan pipeline, el portal se actualiza con un MR automático programado `develop → uat`.
- Página por API con Swagger UI (componente Vue). Catálogo grande: un `index.md` por dominio en el sidebar.
- **CORS:** cada API publicada en el explorer debe incluir el dominio del portal en `CORS_ORIGINS` en todos los ambientes, o el "Try it out" falla. Dejarlo escrito en el README del skeleton.

## Contenido del portal

- Nunca secretos: solo nombre de la variable, dónde está configurada y a quién pedir acceso.
- Escapar `{{` fuera de bloques de código (`\{\{`).
- Archivos incluidos sin frontmatter YAML.
