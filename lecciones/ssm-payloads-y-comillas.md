# Pasar archivos y SQL por AWS SSM desde un pipeline

## Comillas que no sobreviven

Pasar YAML o SQL con comillas, dos puntos y símbolos a través de bash local → `aws ssm send-command` → bash remoto falló varias veces con `sed`. Un `GRANT ... TO 'usuario'@'%'` (comillas simples anidadas) rompe la acción `exec-sql`.

**Patrón que funcionó:** codificar el contenido en base64 y decodificarlo en el servidor:

```bash
CONFIG_B64=$(base64 -w0 archivo.yaml)
# remoto:
echo "$CONFIG_B64" | base64 -d > /opt/monitores/otel-agent-config.yaml
```

## Límite de tamaño

El string base64 dentro de `--parameters` se **truncó en silencio** por encima de ~3000 caracteres: el archivo llegó incompleto. Compactar el contenido (sin comentarios ni bloques opcionales) y verificar siempre leyendo de vuelta el archivo completo en el servidor.

## Sentencias complejas

Para SQL con comillas complejas, entrar al motor por SSH y ejecutar un archivo:

```bash
docker exec -i <contenedor> psql -U <root> -d <base> -f - < cambio.sql
```

## Acción con comportamiento "pegado"

Al cambiar sustancialmente los comandos de una acción de diagnóstico ya usada, el cambio no se reflejaba aunque el commit estuviera. Sospecha (no confirmada) de caché del lado de SSM por nombre de acción reutilizado. Lo práctico fue usar un nombre de acción nuevo.
