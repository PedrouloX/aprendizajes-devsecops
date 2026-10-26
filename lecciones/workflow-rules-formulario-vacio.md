# El formulario de "Run pipeline" aparece vacío

**Síntoma:** en un pipeline con variables de formulario (`variables:` con `description` y `options`), la pantalla de Run pipeline dejó de mostrar campos.

**Causa:** se agregó la confirmación de acciones destructivas como regla con `when: on_success` dentro de `workflow:rules`. `workflow:rules` solo acepta `always` / `never`; el pipeline quedó inválido para la UI.

**Solución:** la lógica de confirmación va en el `rules:` **del job**:

```yaml
db:action:
  rules:
    - if: '$DB_ACTION =~ /^(drop-database|drop-user|reset-container|exec-sql)$/ && $CONFIRM_DESTRUCTIVE != "yes"'
      when: never
    - when: on_success
```

`workflow:rules` solo decide si el pipeline corre o no.
