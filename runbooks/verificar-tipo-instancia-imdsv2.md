# Verificar el tipo real de instancia EC2 sin consola AWS

Las planillas se desincronizan del recurso real (se encontraron servidores anotados como `t3.*` que eran `c5a`/`c6i`). Antes de replicar un servidor en otro ambiente, preguntarle al propio servidor. Solo requiere SSH (o un pipeline vía SSM).

```bash
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-type; echo
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/ami-id; echo
free -h | grep Mem
nproc
lsblk -d -o NAME,SIZE
cat /etc/os-release | grep PRETTY
```

- Si el `curl` sin token no devuelve nada, el servidor tiene IMDSv2 obligatorio: usar la variante con token.
- La RAM visible es menor que la nominal (el hipervisor reserva una parte): ~15 GiB en 16 GB, ~7.3 GiB en 8 GB, ~3.5 GiB en 4 GB.

## Al pedir una máquina equivalente

- Pedir con el tipo verificado, disco y OS. La AMI exacta no suele venir en la solicitud: infraestructura usará la última versión.
- Familias Intel `c5` permiten escalar verticalmente cambiando el tipo (conserva IP privada y EBS).
- Si no existe la combinación pedida (ej. 4 vCPU / 16 GB en `c5`), la más chica que cumple puede tener más vCPU: dejarlo explicado en la solicitud.
