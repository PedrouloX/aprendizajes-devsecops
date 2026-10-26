# Copias de la llave del certificado wildcard legibles por todos

**Hallazgo:** al aprovisionar servidores con los scripts base, la carpeta `certs/` (certificado + llave privada del wildcard) se subía al home del operador y quedaba ahí con la llave en `644`. En un servidor de BDD, que ni siquiera tiene Nginx, la llave estaba igual.

**Riesgo:** cualquier usuario del servidor podía leer la llave privada del wildcard de la organización.

**Corrección:**

```bash
chmod 600 ~/certs/*.key
```

y borrar la copia del home donde la llave ya está en `/opt/nginx/certs/` (con `600`).

**Para no repetirlo:** que el script de instalación mueva la llave a su destino con `600` y borre el origen, y no subir `certs/` a servidores que no corren Nginx. Revisar periódicamente:

```bash
sudo find / -name '*.key' -perm /044 2>/dev/null
```
