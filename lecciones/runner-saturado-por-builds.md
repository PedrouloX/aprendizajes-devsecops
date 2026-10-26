# Los builds de Next.js saturaban el runner compartido

**Síntoma:** cada build de una app Next.js (Turbopack) dejaba el servidor de runners al 100 % de CPU; infraestructura levantó dos alertas de costo en la misma semana.

**Solución:** un runner dedicado en el mismo host, limitado por configuración, y un tag propio:

```toml
[[runners]]
  name = "nextjs-build-limited"
  [runners.docker]
    cpus = "2"
```

Todos los jobs del orquestador Next.js (no solo `build`) pasaron a ese tag, para no mezclar los dos runners en un mismo pipeline.

**Resultado:** build de ~30 s sin saturar el host.

**Relacionado:** revisar también el límite de memoria de los contenedores del runner; en otro runner hubo que subirlo de 2 g a 4 g.
