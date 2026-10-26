# Estándar de creación de repos (GitLab)

- Crear con README; rama por defecto `main`.
- Ramas: `develop`, `uat`, `main` y `pprod` si el proyecto tiene pre producción.
- Protección:
  - `develop`: Developers + Maintainers (push y merge)
  - `uat` y `pprod`: solo Maintainers
  - `main`: protegida por defecto
- Environments `dev`, `uat`, `pprod`, `prd` en **todos** los repos, incluido el `-workspace` (aunque no despliegue).
- Variables base solo en repos que despliegan:
  - `SERVER_ROLE` (una sola, scope All)
  - `ENV` tipo File vacía (la entrega el desarrollador)
  - `REPLICAS=1`, `DNS_NAME`, `DEPLOY_WAIT_TIME=5`
- Repos sin despliegue (`-workspace`): ramas, protección y environments, sin variables.
- Solicitante como Developer en cada repo, sin tocar los miembros heredados.
- Con máquina dedicada: crear `SRV_<AMB>_<ROLE>_HOST` en el grupo cuando se asigne la IP.

## Automatización

- Ejecutar vía API con un Group Access Token **temporal** (rol Owner, scope `api`, vence al día siguiente). Borrarlo al terminar.
- Las variables CI/CD se pueden cargar con un script PowerShell local contra la API.
