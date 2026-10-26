# Nginx no atendía un dominio recién creado

**Síntoma:** dos dominios nuevos daban `000` por HTTPS desde la VPN y conexión rechazada en 80/443, aunque el contenedor Nginx publicaba los puertos y `nginx -t` daba OK. `curl` local: `Connection reset by peer`. Otros servidores creados igual funcionaban.

**Causa:** el script que crea el server block corre `nginx -t` pero no recarga. Nginx seguía con la configuración de su arranque, que no tenía ningún `listen`.

**Solución:**

```bash
sudo docker exec nginx nginx -s reload
```

Los dominios pasaron a `404` (correcto, sin apps aún).

**Para no repetirlo:** que el script de creación termine con `nginx -t && nginx -s reload`. No perder tiempo en firewall de host si los servidores no lo usan: revisar primero qué configuración tiene cargada Nginx (`nginx -T`).
