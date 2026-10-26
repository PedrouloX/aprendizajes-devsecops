# Flujo de ramas y ambientes

## Ambientes

| Ambiente | Rama GitLab | Environment GitLab | Sufijo DNS / ENV | Uso |
|---|---|---|---|---|
| DEV | `develop` | `dev` | `dev` | Integración diaria |
| QA | `uat` | `uat` | `qa` | Certificación (los nombres en GitLab quedaron como `uat` por historia) |
| Pre producción | `pprod` | `pprod` | `preprod` | Espejo de PRD |
| PRD | `main` | `prd` | — | Producción |

> El nombre de la rama/environment y el sufijo de DNS no tienen por qué coincidir. Documentarlo en un solo lugar y no "corregirlo" en uno de los dos sin revisar el otro: un `DNS_NAME` con el sufijo equivocado fue causa de errores.

## Flujo de promoción

```
push develop        → build + deploy DEV
develop ──merge──▶ uat    → deploy QA (los tags vX.Y.Z-rc.N son de QA)
uat     ──MR────▶ pprod  → deploy PREPROD (automático al merge)
pprod   ──MR────▶ main   → tag vX.Y.Z → deploy PRD
uat     ──MR────▶ main   → permitido solo para urgencias
hotfix/*, release/* ──▶ main
```

- Ramas válidas: `feature/*`, `fix/*`, `hotfix/*`, `release/*`, `develop`, `uat`, `pprod`, `main`, `revert-*` (las genera GitLab al revertir desde la UI).
- No reutilizar los tags `-rc` para pre producción: los devs los usan a diario para QA.
- Los tags se crean a mano después del deploy, no inmediatamente tras el push.

## Job `pipeline:guard`

Primer job de todo pipeline. Valida:

1. **Nomenclatura** de la rama.
2. **Origen del MR según el destino** (solo en `merge_request_event`):
   - destino `uat` → solo desde `develop`
   - destino `pprod` → solo desde `uat`
   - destino `main` → solo `hotfix/*`, `release/*`, `uat` o `pprod`
3. **Pertenencia real del tag a su rama**, no solo el nombre:

```bash
git merge-base --is-ancestor "$CI_COMMIT_SHA" origin/main
```

   Requiere `GIT_DEPTH: "0"` en el job (solo afecta pipelines de tag).

## Por qué así

- GitLab Free no tiene restricción nativa origen → destino de MR. Las Push Rules son Premium y además tienen un bug que bloquea merges a ramas no default. El guard en el pipeline + **"Pipelines must succeed"** activo en cada repo es lo que convierte la validación en bloqueo real.
- Si el guard vive en un template compartido, verificar que la regla exista en **todos** los orquestadores: se documentó como aplicada en los tres y solo estaba en dos.
