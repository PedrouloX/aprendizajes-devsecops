# ci-templates: orquestadores compartidos

Un repo `ci-templates` con un orquestador por tipo de app (Node.js, Next.js, React/estático, certificación QA) y scripts bash compartidos. Cada repo de aplicación lo consume.

## Regla de oro

El `.gitlab-ci.yml` de cada app contiene **solo `include`**:

```yaml
include:
  - project: 'infraestructura/ci-templates'
    ref: 'main'
    file: 'pipelines/node-app/node-app.gitlab-ci.yml'
```

Sin stages ni variables. Todo lo configurable va en Settings → CI/CD → Variables del repo o del grupo.

## Resolución del servidor destino

- La app define `SERVER_ROLE` (ej. `BACKEND`, `FRONTEND`, o un prefijo propio si tiene máquina dedicada).
- El grupo define `SRV_<AMBIENTE>_<SERVER_ROLE>_HOST` con la IP.
- El ambiente sale del nombre del environment de GitLab, así que agregar un ambiente nuevo (ej. `pprod`) no requiere tocar los scripts, solo los jobs y las variables `SRV_PPROD_*`.

En `sh` de Alpine la expansión indirecta `${!var}` no existe:

```sh
HOST=$(eval echo "\$SRV_${ENV_UPPER}_${SERVER_ROLE}_HOST")
```

## Nginx generado por el deploy

- `DNS_NAME` acepta varios dominios separados por coma; el primero es el principal.
  - El **upstream** se escribe una sola vez, en la carpeta del dominio principal (Nginx carga todos los upstreams; repetirlo da `duplicate upstream`).
  - La **location** se escribe en la carpeta de cada dominio.
- `SERVICE_PATH` es obligatorio cuando varias apps comparten dominio (en Next.js, con su `basePath`).
- `EXTRA_PATHS` (opcional) publica rutas adicionales hacia el mismo upstream, en el mismo archivo, para que el `stop`/limpieza las borre junto con la principal. Validar formato, duplicados y conflicto con otros servicios **antes** de escribir nada.
- Antes de escribir, verificar que ninguna otra app use la misma ruta en ese dominio. Si está tomada, abortar sin tocar el servidor: falla solo la app mal configurada, no el resto.
- `nginx -t` bloqueante y con su salida visible en el log (un `set -e` en el script remoto puede cortar antes de imprimir el error).

## Convenciones

- Contenedores: `<SERVICE_NAME>-<ENV>-<N>` (ej. `api-portal-uat-1`).
- Label `com.<org>.repo.id=${CI_PROJECT_ID}` en el `docker run`, útil para descubrir apps (autoscaling, inventario).
- `.gitattributes` con `* text=auto eol=lf` en el repo de templates.
- Versionar el repo (`VERSION` + `CHANGELOG.md`) y validar con **CI Lint** en cada rama afectada antes de mergear (desde un repo que use el orquestador, pegando el contenido).
- Para probar un cambio grande: apuntar temporalmente el `include` de los repos afectados a la rama feature, desplegar, y devolverlos a `ref: main` con un commit `[skip ci]`.

## Acceso de otros grupos

Si un grupo distinto (ej. QA) consume el template vía `include: project:`, agregar **el grupo completo** como miembro de `ci-templates` con rol **Reporter** (mínimo `read_code`). Sin eso el pipeline falla con `Project ... not found or access denied`, aunque tengan acceso al subgrupo padre.

## Runners

- Builds pesados (Next.js con Turbopack) saturaban el runner compartido y gatillaron alertas de costo. Solución: runner dedicado con `cpus = "2"` en `config.toml` y un tag propio; se movió **todo** el flujo de ese orquestador al runner dedicado, no solo el build.

## MR automático develop → uat

Job programado (schedule sobre `develop`, variable `AUTO_MR=true`):
1. Compara `uat...develop`; si no hay commits, termina sin MR.
2. Reutiliza el MR abierto o crea uno.
3. Espera a que el MR tenga pipeline y recién ahí activa `merge_when_pipeline_succeeds`; si no puede, falla mostrando la respuesta de la API.

Requiere un Project Access Token en una variable Masked + Protected.
