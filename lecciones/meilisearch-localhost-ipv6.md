# MeiliSearch no responde a `localhost` dentro del contenedor

**Síntoma:** `curl http://localhost:7700/health` dentro del contenedor falla, aunque el servicio está arriba y las apps lo usan.

**Causa:** dentro del contenedor `localhost` resuelve a IPv6 (`::1`) y MeiliSearch escucha solo en IPv4.

**Solución:** probar con `127.0.0.1`, o desde otro contenedor de la misma red por nombre:

```bash
docker exec <contenedor> wget -qO- http://127.0.0.1:7700/health
docker exec <app> wget -qO- http://<meilisearch>:7700/health
```

**Otros:** sin clave debe responder `401`; con la master key, `available`. La telemetría anónima viene activa por defecto (`MEILI_NO_ANALYTICS=true` para apagarla).
