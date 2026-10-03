# `ruido-comb` — combinar los dos ruidos de entrenamiento que más ayudaron: ¿suman?

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-03 06:53 → 07:05 (3 semillas) y 07:33 → 07:44 (semillas 4 y 5); dos unidades de systemd en el dev, `Result=success`, `NRestarts=0`, 0 fallos |
| **Instancias alquiladas** | **0** — droplet dev |
| **Coste real** | **0 $** |
| **Runs** | **25**: `limpio`, `gaussiano@0.2-linea`, `recorte@0.6-linea`, la **secuencial** `recorte@0.6+gaussiano@0.2-linea` y la **mezcla** `recorte@0.6~gaussiano@0.2-linea`, × 5 semillas (3 del plan + 2 por la regla escrita en el criterio antes de correrlas) |
| **Dataset / red** | los de #28 (`uci-optdigits-8px-r20261002`, 180 + 180 copias en línea / 1617 val; 3 × Conv 3×3 C = 8 + GAP, 1.338 parámetros, `lr = 3e-3`, 3996 pasos), **mismos pesos iniciales** |
| **Artefactos** | [`experimentos-cnn/2026-10-03-ruido-combinado/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-03-ruido-combinado); pesos en el almacén `foveal-vision-data/experimentos-cnn-resultados/ruido-comb/` |

Continuación de #28, abierta como experimento propio porque aquel criterio excluía combinar.
Pregunta: ¿aplicar juntos gaussiano σ = 0,2 y recorte α = 0,6 (ambos en línea) sube la exactitud
de val por encima del mejor de los dos solos? Dos formas: **secuencial** (recorte y, encima,
gaussiano) y **mezcla** (cada imagen uno de los dos, al 50 %). Δ pareado por semilla, umbral
`max(2·SE, 0,01)`.

## Resultado: indistinguible, como se escribió antes — y con 5 semillas también

| escenario | acc val (5 semillas) | Δ vs `limpio` | **Δ vs `gaussiano@0.2-linea`** ± SE | umbral | veredicto |
|---|---|---|---|---|---|
| `gaussiano@0.2-linea` (el mejor simple) | 0,902 ± 0,034 | +0,033 | referencia | — | — |
| **secuencial** | **0,916 ± 0,012** | **+0,047** | **+0,014** ± 0,012 | 0,025 | indistinguible |
| mezcla | 0,898 ± 0,021 | +0,029 | −0,004 ± 0,011 | 0,021 | indistinguible |

- Con las 3 semillas del plan: secuencial 0,919 contra 0,912 del simple, +0,007 ± 0,009 (umbral
  0,018). El criterio decía que lo decidirían 5; se añadieron la 4 y la 5 con la misma regla y
  **tampoco**: la secuencial subió su ventaja (+0,014) pero el gaussiano solo resultó **más
  variable** (una semilla a 0,85; sd 0,034 contra 0,012 de la secuencial) y el umbral creció con
  él. Se cierra sin ampliar más, como mandaba la enmienda.
- La **secuencial** es la exactitud más alta **y más estable** vista en #28 y aquí, con la CE de
  val más baja (0,297 contra 0,395). Contra `recorte@0.6-linea` **sí** supera el umbral (+0,017 ±
  0,005, umbral 0,011): añadir gaussiano al recorte ayuda; añadir recorte al gaussiano no se
  distingue.
- La **mezcla** es la forma más lejos de sumar: repartir equivale a media dosis de cada uno.
- **Ninguna resta.** Los dos simples salieron **bit a bit iguales** que en #28 (en las semillas 1–3 ; mismas huellas de
  copia): el código copiado hace lo mismo y la comparación es limpia.

## Lo que quedó pendiente

- Nada: se amplió a 5 semillas y el criterio cierra ahí. Lo que quedó enseñado es que el efecto
  del gaussiano solo es menos robusto entre semillas que el de la combinación.
- No mueve `ESTADO.md` (otra red, otro dato). El aviso a Telegram del cierre no salió (unidad
  lanzada desde una sesión de Claude Code, sin `BOT_TOKEN`).
