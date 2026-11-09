---
name: clima
description: Consulta el clima actual y el pronóstico de los próximos días en San Luis Potosí, México. Úsala cuando el usuario pregunte por el clima, el tiempo, la temperatura, si va a llover, o invoque /clima.
allowed-tools: Bash(curl:*)
---

# Clima en San Luis Potosí, México

Obtén el clima y preséntalo **en español**. No requiere API keys.

La ubicación es **siempre San Luis Potosí, S.L.P., México** (lat `22.1565`, lon `-100.9855`, zona horaria `America/Mexico_City`). No detectes la ubicación por IP ni busques otras ciudades, aunque se pase un argumento.

## 1. Consultar el clima (Open-Meteo)

```bash
curl -s -m 10 "https://api.open-meteo.com/v1/forecast?latitude=22.1565&longitude=-100.9855&current=temperature_2m,apparent_temperature,relative_humidity_2m,precipitation,weather_code,wind_speed_10m,wind_direction_10m,is_day&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max,uv_index_max,sunrise,sunset&timezone=America%2FMexico_City&forecast_days=3"
```

**Respaldo** si Open-Meteo falla: `curl -s -m 10 "https://wttr.in/San+Luis+Potosi,Mexico?format=4&lang=es"`.

## 2. Interpretar `weather_code` (WMO)

| Código | Condición | | Código | Condición |
|---|---|---|---|---|
| 0 | ☀️ Despejado | | 61, 63, 65 | 🌧️ Lluvia ligera / moderada / fuerte |
| 1 | 🌤️ Mayormente despejado | | 66, 67 | 🌧️🧊 Lluvia helada |
| 2 | ⛅ Parcialmente nublado | | 71, 73, 75 | 🌨️ Nieve ligera / moderada / fuerte |
| 3 | ☁️ Nublado | | 77 | 🌨️ Granos de nieve |
| 45, 48 | 🌫️ Niebla | | 80, 81, 82 | 🌦️ Chubascos ligeros / moderados / violentos |
| 51, 53, 55 | 🌦️ Llovizna ligera / moderada / densa | | 85, 86 | 🌨️ Chubascos de nieve |
| 56, 57 | 🌧️ Llovizna helada | | 95, 96, 99 | ⛈️ Tormenta (con granizo en 96/99) |

Si `is_day` es 0 y el código es 0 o 1, usa 🌙 en lugar de ☀️/🌤️.

Dirección del viento (grados → punto cardinal): N 0°, NE 45°, E 90°, SE 135°, S 180°, SO 225°, O 270°, NO 315°.

## 3. Formato de respuesta

```
📍 San Luis Potosí, S.L.P., México

Ahora: <emoji> <condición>, <temp>°C (sensación <aparente>°C)
💧 Humedad <h>%  ·  💨 Viento <v> km/h <dirección>  ·  🌧️ Precipitación <p> mm

Próximos días:
| Día | Condición | Mín / Máx | Prob. lluvia | UV |
|---|---|---|---|---|
| Hoy | ... | 14° / 26° | 10% | 7 |
| Mañana | ... | ... | ... | ... |
| <día de la semana> | ... | ... | ... | ... |

🌅 Amanecer <hh:mm> · 🌇 Atardecer <hh:mm>
```

Cierra con **una** recomendación breve y práctica (paraguas si prob. lluvia ≥ 40%, protector solar si UV ≥ 6, abrigo si mín ≤ 10 °C, hidratación si máx ≥ 32 °C). Si el usuario hizo una pregunta concreta (p. ej. "¿va a llover mañana?"), respóndela directamente primero.
