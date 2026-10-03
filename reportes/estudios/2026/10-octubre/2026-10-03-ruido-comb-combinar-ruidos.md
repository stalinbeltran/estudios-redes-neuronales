# `ruido-comb` — combinar los dos ruidos de entrenamiento que más ayudaron: ¿suman?

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-03 06:53 → 07:05 (una unidad de systemd en el dev, `Result=success`, `NRestarts=0`, 0 fallos) |
| **Instancias alquiladas** | **0** — droplet dev |
| **Coste real** | **0 $** |
| **Runs** | **15**: `limpio`, `gaussiano@0.2-linea`, `recorte@0.6-linea`, la **secuencial** `recorte@0.6+gaussiano@0.2-linea` y la **mezcla** `recorte@0.6~gaussiano@0.2-linea`, × 3 semillas |
| **Dataset / red** | los de #28 (`uci-optdigits-8px-r20261002`, 180 + 180 copias en línea / 1617 val; 3 × Conv 3×3 C = 8 + GAP, 1.338 parámetros, `lr = 3e-3`, 3996 pasos), **mismos pesos iniciales** |
| **Artefactos** | [`experimentos-cnn/2026-10-03-ruido-combinado/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-03-ruido-combinado); pesos en el almacén `foveal-vision-data/experimentos-cnn-resultados/ruido-comb/` |

Continuación de #28, abierta como experimento propio porque aquel criterio excluía combinar.
Pregunta: ¿aplicar juntos gaussiano σ = 0,2 y recorte α = 0,6 (ambos en línea) sube la exactitud
de val por encima del mejor de los dos solos? Dos formas: **secuencial** (recorte y, encima,
gaussiano) y **mezcla** (cada imagen uno de los dos, al 50 %). Δ pareado por semilla, umbral
`max(2·SE, 0,01)`.

## Resultado: indistinguible, como se escribió antes

| escenario | acc val | Δ vs `limpio` | **Δ vs `gaussiano@0.2-linea`** ± SE | umbral | veredicto |
|---|---|---|---|---|---|
| `gaussiano@0.2-linea` (el mejor simple) | 0,912 ± 0,021 | +0,042 | referencia | — | — |
| **secuencial** | **0,919 ± 0,014** | **+0,049** | **+0,007** ± 0,009 | 0,018 | indistinguible |
| mezcla | 0,903 ± 0,021 | +0,032 | −0,010 ± 0,014 | 0,028 | indistinguible |

- La **secuencial** da la exactitud más alta vista en #28 y aquí (0,919) y la CE de val más baja
  (0,298 contra 0,359), **pero no se distingue del mejor simple**: las semillas se reparten
  (+0,023, +0,051, +0,072 sobre `limpio`) y con 3 el SE es del tamaño del efecto. Contra
  `recorte@0.6-linea` sí supera el umbral (+0,020 ± 0,009, umbral 0,018): añadir gaussiano al
  recorte ayuda; añadir recorte al gaussiano no se distingue.
- La **mezcla** es la forma más lejos de sumar: repartir equivale a media dosis de cada uno.
- **Ninguna resta.** Los dos simples salieron **bit a bit iguales** que en #28 (mismas huellas de
  copia): el código copiado hace lo mismo y la comparación es limpia.

## Lo que quedó pendiente

- **5 semillas para la secuencial**, que es lo único que decidiría (SE ~0,006, umbral ~0,012).
  No se ha hecho.
- No mueve `ESTADO.md` (otra red, otro dato). El aviso a Telegram del cierre no salió (unidad
  lanzada desde una sesión de Claude Code, sin `BOT_TOKEN`).
