# Los logs de los contenedores sin SDK nunca llegaban a Loki

**Contexto:** se querían los logs de MySQL/Postgres en los dashboards. Esos contenedores no tienen SDK OTel; sus logs llegan por el receiver `filelog` del otel-agent, que lee `/var/lib/docker/containers/*/*-json.log`.

**Hallazgo:** `filelog` nunca había funcionado en ningún servidor. No se notó porque las apps instrumentadas mandan sus logs por OTLP, por otro camino.

## Causas (en orden de aparición)

1. **SELinux `Enforcing` (AlmaLinux) bloqueaba el mount** de `/var/lib/docker/containers` en la variante de compose sin apps: le faltaba `:z`. Sin ningún error en los logs de la aplicación; solo en `ausearch -m avc -ts recent`.
2. **`json_parser` incompleto:** faltaban `timestamp` (parsear `time` del JSON de Docker) y `move` (`attributes.log` → `body`).
3. **`service_name=unknown_service`:** el contenedor no se identifica en su propio log. Dos partes:
   - Docker: `--log-driver json-file --log-opt tag="{{.Name}}"` en el contenedor.
   - Agente: copiar ese tag al atributo **de recurso**:

```yaml
processors:
  transform/tag:
    log_statements:
      - context: log
        statements:
          - set(resource.attributes["service.name"], attributes["attrs.tag"]) where attributes["attrs.tag"] != nil
```

   En la práctica la clave quedó como `attrs.tag` plana (con punto literal), no anidada.

4. **Operador `container` descartado:** está pensado para el layout de Kubernetes (`/var/log/pods/...`), no para Docker standalone.

## Lección de método

Varias horas se habrían ahorrado empezando por **lo que recibe el destino**:

```bash
curl -s http://<loki>:3100/loki/api/v1/labels
curl -s http://<loki>:3100/loki/api/v1/label/service_name/values
```

Si el valor no está en Loki, el problema está antes; si está, el problema es la query.
