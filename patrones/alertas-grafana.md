# Alertas en Grafana

## Set base (un grupo, todos los ambientes)

| Alerta | Condición | Severidad |
|---|---|---|
| BDD offline | `mysql_up == 0 or pg_up == 0` → `IS BELOW 1` | Crítica |
| Servidor offline | `up == 0` → `IS BELOW 1` | Crítica |
| Deadlocks | `rate(...) > 0` | Crítica |
| Conexiones BDD altas | ≥ 80 % del máximo | Advertencia |
| CPU alta | > 80 % por 5 min | Advertencia |
| RAM alta | > 85 % por 5 min | Advertencia |
| Disco | > 90 % | Crítica |
| Buffer pool MySQL | hit ratio < 90 % | Advertencia |

Sin filtro por `vpc`: una sola regla cubre todos los ambientes, no se duplica por ambiente.

## Configuración obligatoria

- **"Alert state if no data or all values are null" → Normal.** Las queries filtran (`== 0`, `> umbral`), así que lo normal es que no devuelvan nada; con el default "No Data" dispara cada ciclo.
- La condición tiene que ir en la misma dirección que el filtro de la query (ver lección `alerta-que-nunca-dispara.md`).
- Mientras los servidores se apagan todas las noches (ambiente sin alta formal), **pausar** las alertas "offline" y anotar que hay que reactivarlas.

## Notificaciones por correo vía servicio interno

```
Regla → Notification Template → Contact Point (webhook) → API del servicio de notificaciones → template de correo
```

- Contact Point tipo webhook con el header de clave interna del servicio.
- El template de Grafana **itera sobre `.Alerts`** (no `.CommonLabels`) para listar todos los servidores cuando se agrupan varias alertas.
- Destinatarios en un solo lugar (el template), no en cada regla.
- Regla de red: observabilidad → servicio de notificaciones TCP 443.
