# El `tar` para llevar el compose a otro ambiente salió vacío

**Síntoma:** se armó un `.tar.gz` con la configuración de la BDD en un ambiente para replicarla en otro. Al extraerlo no había nada: pesaba 45 bytes.

**Causa:** no se identificó. El punto es que `tar` no falla si no encuentra lo que se esperaba (ruta mal armada, permisos sin `sudo`): empaqueta lo que hay y termina OK.

**Para no repetirlo:** revisar el contenido **antes** de moverlo:

```bash
sudo tar --exclude='.env' -czf ~/db-compose-<proy>.tar.gz -C /opt db-compose/<proy>
tar -tzvf ~/db-compose-<proy>.tar.gz
```

Y en general, para replicar un compose entre ambientes es más rápido y seguro recrear los archivos con el runbook (cambian sufijos de ambiente y passwords de todos modos). Nunca incluir el `.env`.
