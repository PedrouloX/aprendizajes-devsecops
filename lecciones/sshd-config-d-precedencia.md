# El cambio en `sshd_config` no tomaba efecto

**Síntoma:** el script que habilita `PasswordAuthentication` para el equipo de infraestructura editaba `/etc/ssh/sshd_config` y el acceso con contraseña seguía rechazado en algunas máquinas.

**Causa:** en imágenes cloud existe `/etc/ssh/sshd_config.d/50-cloud-init.conf` con `PasswordAuthentication no`, y los archivos de `sshd_config.d/` tienen precedencia (el primer valor leído gana). Además, el reemplazo in-place con `sed` no era confiable si la línea no existía tal cual.

**Solución:** borrar la directiva existente y agregarla de nuevo en el archivo que gana, luego validar y recargar:

```bash
sudo sed -i '/^PasswordAuthentication/d' /etc/ssh/sshd_config.d/50-cloud-init.conf
echo 'PasswordAuthentication yes' | sudo tee -a /etc/ssh/sshd_config.d/50-cloud-init.conf
sudo sshd -t && sudo systemctl reload sshd
sudo sshd -T | grep -i passwordauthentication
```

`sshd -T` muestra el valor efectivo.
