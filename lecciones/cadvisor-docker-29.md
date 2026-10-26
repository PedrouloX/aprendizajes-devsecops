# cAdvisor 0.49 no ve los contenedores con Docker 29

**Síntoma:** los paneles de contenedores quedaban vacíos o en "offline" en los servidores con Docker 29.x.

**Causa:** cAdvisor `v0.49.1` es incompatible con Docker 29.x.

**Solución:** actualizar a `v0.55.1` (o superior).

**Efecto colateral:** desde 0.55 el label del nombre del contenedor pasa de `container_name` a `name`, y los dashboards existentes dejan de filtrar. Se resolvió en Prometheus sin tocar los dashboards:

```yaml
metric_relabel_configs:
  - source_labels: [name]
    target_label: container_name
```

(en los jobs de cAdvisor remoto y local).

**Para no repetirlo:** fijar la versión de cAdvisor en la plantilla de monitores (nunca `latest`) y actualizarla junto con Docker.
