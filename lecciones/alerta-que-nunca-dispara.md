# Una alerta de "offline" que nunca habría disparado

**Contexto:** reglas de Grafana para "BDD offline" y "Servidor offline".

**Bug:** la query filtraba `up == 0` (devuelve 0 cuando algo está caído) y la condición era `IS ABOVE 0`. El valor que llega a la condición ya es `0`, y "¿0 > 0?" es siempre falso. La alerta **nunca** iba a disparar con datos reales; se habría descubierto en el primer incidente.

**Solución:** condición `IS BELOW 1`. Regla general: filtro de la query y condición en la misma dirección.

**Segundo problema, el inverso:** cuando nada está caído la query no devuelve series, y el estado por defecto "No Data" también notifica. Configurar en cada regla: *Alert state if no data or all values are null → Normal*.

**Cómo probarlo:** detener un exporter o un servidor de prueba y confirmar que la alerta pasa a Firing y llega el correo, antes de darla por terminada.
