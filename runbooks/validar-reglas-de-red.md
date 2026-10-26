# Validar reglas de red desde el origen

Probar **siempre desde el origen real** de la regla y pedir solo las que fallan.

## Desde el PC por VPN (PowerShell)

```powershell
Test-NetConnection <ip> -Port 443
Resolve-DnsName app.ejemplo.cl
```

Más rápido para muchos puertos (`Test-NetConnection` tarda en los que fallan):

```powershell
foreach ($p in 80,443,8201,8202) { $c = New-Object Net.Sockets.TcpClient; try { $c.Connect("<ip>", $p); "ABIERTO $p" } catch { "FALLA $p - $($_.Exception.InnerException.Message)" } finally { $c.Close() } }
```

## Desde un servidor Linux

Sin depender de `nc`:

```bash
timeout 5 bash -c '</dev/tcp/<ip>/443' && echo CONECTA || echo NO CONECTA
```

Con `nc`:

```bash
nc -zv -w5 <ip> 443
```

Desde dentro de un contenedor (la app puede estar en otra red):

```bash
sudo docker exec <contenedor> sh -c 'wget -qO- --timeout=5 https://api.ejemplo.cl/health || echo FALLA'
```

## Desde el runner

Relanzar el job de deploy, o desde el servidor del runner:

```bash
timeout 5 bash -c '</dev/tcp/<ip-maquina>/22' && echo OK
```

## Leer el resultado

| Resultado | Significa |
|---|---|
| Conecta | Regla OK |
| `TIMEOUT` / `curl` `000` tras ~2 min | Regla inexistente, tráfico descartado |
| `REFUSED` | Regla aplicada, nada escuchando en el destino |

Antes de pedir una regla, confirmar que el servicio escucha:

```bash
sudo ss -tlnp | grep ':<puerto> '
```

## DNS

Un registro con dos IP hace que el navegador caiga al azar en el servidor equivocado. Validar que haya **un solo registro A**:

```powershell
Resolve-DnsName app.ejemplo.cl
```

```bash
getent hosts app.ejemplo.cl
```
