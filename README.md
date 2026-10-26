# Aprendizajes DevSecOps

Base de conocimiento personal de Pedro Llantén: patrones, lecciones y runbooks
de infraestructura y DevSecOps, destilados de proyectos reales.

## Regla principal
Nada de datos de clientes. Todo lo que entra aquí está sanitizado:
- IPs reales → rangos genéricos (`10.0.x.x`) o placeholders (`<ip-back>`)
- Hosts, dominios y nombres de proyectos → genéricos (`<servidor-back>`, `app.ejemplo.cl`)
- Sin credenciales, tokens, llaves ni números de ticket

Si un aprendizaje no se entiende sin el dato del cliente, no entra.

## Estructura
- `patrones/` — cómo se hacen las cosas: ambientes DEV/QA/PRD, ramas,
  variables CI/CD, ci-templates.
- `lecciones/` — un archivo por problema: qué falló, por qué y cómo se resolvió.
- `runbooks/` — procedimientos paso a paso reutilizables.
- `skills/` — skills de Claude en versión genérica.

## Cómo se mantiene
Los cambios se proponen y se revisan antes de cada commit.
