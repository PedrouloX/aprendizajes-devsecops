# Arquitectura DevSecOps de referencia

GitLab CE como orquestador, herramientas QA y de seguridad en un stack aparte, observabilidad centralizada.

## Etapas del pipeline

| Etapa | Herramienta | Cuándo | Bloquea |
|---|---|---|---|
| Guard | script propio | siempre | sí |
| Secretos | Gitleaks | MR → develop, release | sí |
| SAST | Semgrep | MR → develop, release | sí |
| Calidad | SonarQube (Quality Gate) | push develop | no (hasta que los devs tengan tests) |
| Tests unitarios | del proyecto | push develop | no |
| SCA | Trivy | MR | no |
| Build + deploy | ci-templates | según rama | — |
| DAST | OWASP ZAP | manual en QA/PRD | — |

Hallazgos SAST, secretos, SCA y DAST se suben a **DefectDojo** como concentrador (`auto_create_context=true`). Si nadie revisa DefectDojo, desactivarlo con una variable (`DEFECTDOJO_ENABLED`) en vez de dejar uploads que nadie mira.

## DAST con ZAP

- `dast:zap:baseline`: pasivo + spider acotado, 5–10 min, seguro en horario laboral.
- `dast:zap:full`: activo con scope limitado, hasta 3 h, **fuera de horario** y con QA avisado.
- Ambos manuales. `DAST_TARGET_URL` solo en las apps con UI (decisión de equipo, no restricción técnica).
- Apps cuyo Nginx no expone un dominio/ruta público real no se pueden escanear tal cual.

## QA

- Squash TM para casos de prueba y campañas; MantisBT para defectos. Integrarlos por la red Docker interna (`http://mantis:80`) en vez de pasar por el Nginx público.
- Pruebas E2E del equipo QA (Playwright/Python) en el mismo orquestador de certificación, con un tag de runner propio. Publicar resultados aunque haya casos en rojo (ver lección `pytest-corta-antes-de-publicar.md`).
- Quality Gate exigente (90 % cobertura, 0 blocker, Maintainability A, duplicación < 3 %) definido pero sin activar hasta que haya tests reales.

## Observabilidad

Apps → otel-agent local → gateway → Prometheus / Loki / Tempo → Grafana. Ver `stack-observabilidad.md`.
