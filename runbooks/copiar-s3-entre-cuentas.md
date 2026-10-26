# Copiar un bucket S3 a otra cuenta AWS

Cuando los buckets están en cuentas distintas y no hay una política cruzada, copiar en dos pasos desde un servidor con espacio en disco: bajar con las credenciales de origen (solo lectura) y subir con las de destino.

## Configuración que no se cae

Con la configuración por defecto `aws s3 sync` murió por memoria (exit `137`, `OOMKilled`) con 512 MB y con 2 GB. Funcionó así:

```bash
mkdir -p /var/tmp/s3-copia
docker run -d --name s3-bajada --memory 3g --restart on-failure:10 \
  -e AWS_ACCESS_KEY_ID=<origen> -e AWS_SECRET_ACCESS_KEY=<origen> \
  -v /var/tmp/s3-copia:/data amazon/aws-cli:2.22.35 \
  sh -c 'aws configure set default.s3.max_concurrent_requests 4 && aws configure set default.s3.max_queue_size 100 && aws s3 sync s3://<bucket-origen> /data'
```

Seguir el avance con `docker logs -f s3-bajada` y borrar el contenedor al terminar (`--restart` no se combina con `--rm`).

Subida igual, con las credenciales de destino y `aws s3 sync /data s3://<bucket-destino>`.

## Llaves que no caben en disco

- Un objeto `a.webp` y otro `a.webp/desktop.webp` (archivo y "carpeta" con el mismo nombre).
- Rutas con doble barra (`carpeta//archivo`).

Copiarlos objeto por objeto con `aws s3 cp s3://origen/<llave> s3://destino/<llave>` (con credenciales que tengan acceso a ambos, o vía archivo temporal con otro nombre).

## Restos de cortes

Cada caída deja descargas a medias con un sufijo aleatorio de 8 caracteres. Buscarlas y borrarlas también en el destino si alcanzaron a subirse.

## Validar

Comparar las listas de llaves en ambos sentidos:

```bash
aws s3 ls s3://<bucket> --recursive | awk '{$1=$2=$3=""; print substr($0,4)}' | sort > llaves.txt
diff llaves-origen.txt llaves-destino.txt
```

0 diferencias. Si el origen sigue recibiendo escrituras durante la copia, repetir un `sync` final.

Al terminar, borrar los temporales del servidor y revisar `df -h`.
