# `ruido-nist` — qué ruido de entrenamiento ayuda a generalizar en dígitos de 8×8 (con el 10 % del dato)

| | |
|---|---|
| **Inicio / fin (UTC)** | fase 1: 2026-10-03 00:59 → 01:18 · fase 2: 03:56 → 04:29 · fase 3: 05:17 → 05:28 · fase 4: 06:03 → 06:17 (cuatro unidades de systemd en el dev, `Result=success`, `NRestarts=0`, 0 fallos) |
| **Instancias alquiladas** | **0** — droplet dev |
| **Coste real** | **0 $** |
| **Runs** | **150**: fase 1 = `limpio` + 9 tipos a su nivel medio + una 2ª copia de `oblicua`, × 3 semillas (33); fase 2 = los 5 niveles de los 7 tipos que pasaron × 3 (84 nuevas); fase 3 = el mejor nivel de los 5 tipos que ayudaron, con ruido **en línea** (una copia nueva por época), × 3 (15); fase 4 = grosor (`-grueso`, 3–4 px) y número de trazos (`-doble`, 3–4) de los 3 trazos que ayudaban en línea, × 3 (18) |
| **Dataset** | `uci-optdigits-8px-r20261002` (dígitos 8×8 de UCI/NIST vía scikit-learn, el de `dim-nist`), train 180 (+180 copias) / val 1617 limpias |
| **Red** | 3 × Conv(3×3, C = 8) sin padding + GAP + Linear(8→10), 1.338 parámetros; Adam `lr = 3e-3`, 3996 pasos de lote 20, `last.pt` |
| **Artefactos** | [`experimentos-cnn/2026-10-02-ruido-nist/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-02-ruido-nist) (README, `resultados/RESULTADOS.md`, figuras); pesos en el almacén `foveal-vision-data/experimentos-cnn-resultados/ruido-nist/` |

**Pregunta del dueño:** con muy poco train (180 imágenes), ¿qué tipo de «ruido» añadido sólo al
train —borrar píxeles, motas, rectas, curvas, recortes, gaussiano, sal y pimienta— mejora la
exactitud sobre las 1617 no vistas, y cuánto? Con **pesos iniciales idénticos** en todos los
escenarios por semilla (un fichero `init-s<s>.pt`), mismo orden de lotes y la copia ruidosa fija
compartida por las tres semillas: lo único que cambia entre dos escenarios es la copia. La medida
es **Δ pareado por semilla** contra `limpio` (las 180 duplicadas sin ruido), umbral
`max(2·SE, 0,01)`.

## Resultado

**Ningún ruido perjudica a ningún nivel** (el peor Δ de 38 escenarios es −0,010), y **cinco de los
nueve tipos tienen un nivel que ayuda**, sobre una base de 0,870 ± 0,032:

| mejor nivel | Δ exactitud val ± SE | umbral | Δ CE val | forma del eje |
|---|---|---|---|---|
| **`recorte@0.6`** (cutout ¼–½ del lado, α 0,6) | **+0,029** ± 0,011 | 0,021 | **−0,39** | pico interior |
| `curva@0.8` | +0,021 ± 0,009 | 0,017 | −0,31 | pico interior |
| `gaussiano@0.2` | +0,017 ± 0,005 | 0,010 | −0,29 | pico; **cae a 0,000 en σ = 0,3** |
| `vertical@1` | +0,014 ± 0,002 | 0,010 | −0,22 | sube hasta α = 1 (no acotado) |
| `oblicua@1` | +0,012 ± 0,004 | 0,010 | −0,24 | sube hasta α = 1 (no acotado) |
| `borrado@0.4` · `sal-pimienta@0.2` | +0,019 · +0,018 | 0,020 · 0,021 | +0,04 · −0,36 | indistinguibles con 3 semillas |

`externos` y `horizontal` no pasaron de la fase 1 (−0,000 y −0,010 al nivel medio). En la fase 1
sólo `recorte` superaba el umbral; el resto de «ayuda» aparece al subir la intensidad: **la forma
general es «más ruido, mejor» dentro del rango probado**, con `gaussiano` como único caso de
sobredosis. Y un hallazgo que el criterio no pedía: casi todos los ruidos **bajan la entropía
cruzada de val** sin mover la exactitud — la red acierta igual y se equivoca con menos seguridad.

**Fase 3 — en línea contra copia fija (pareado).** Los cinco siguen ayudando contra `limpio`, y
**ninguno empeora** respecto de su copia fija. Dos la superan: **`gaussiano@0.2-linea`, el mejor
resultado del estudio, 0,912 (+0,042 sobre `limpio`, +0,025 ± 0,006 sobre su fija, umbral
0,011)**, y `vertical@1-linea` (+0,011 ± 0,005 sobre su fija, umbral 0,010). `recorte`, `curva` y
`oblicua` quedan indistinguibles de la suya (+0,000, +0,002, +0,007). La CE de val baja otros
0,16–0,39 nats en los cinco. ⚠ Contra lo escrito antes: se esperaba que la variedad sumara en
`recorte` y `curva`; sumó en `gaussiano` y `vertical`. Una copia fija de un cutout ya da lo que
da; el gaussiano cambia entero cada época, y eso es lo que la red aprovecha.

**Fase 4 — grosor y número de trazos, en línea, contra su base (pareado).** **Ninguna de las seis
variantes se distingue de su base**, ni mejor ni peor (Δ entre −0,014 y +0,017, umbrales 0,019–
0,052). Dos quedan entre los mejores absolutos —`curva@0.8-grueso-linea` 0,910 (+0,039 sobre
`limpio`) y `oblicua@1-doble-linea` 0,904 (+0,034)— pero el SE de la comparación contra su base es
del tamaño del efecto. Se esperaba que sumaran en los trazos rectos y no en la curva; salió nada
distinguible en ninguno. El eje de los trazos queda **acotado por el ruido del instrumento**, no
por una caída.

**Lo que no se pudo leer:** el mecanismo. La exactitud de train es 1,000 y su CE ≈ 0,001 en los 38
escenarios; ningún ruido de esta lista deja huella en el ajuste del train limpio.

## Cautelas, escritas con los números

- Efectos de **1–4 puntos con 3 semillas**, y **49 comparaciones a 2·SE sin corrección** (el
  criterio no la fijó): cabría esperar 1–2 «ayuda» por azar. `vertical@1` y `oblicua@1` entran
  porque su SE es diminuto y el umbral cae a δ = 0,01. Los que no dependen de eso son
  `recorte@0.6`, `curva@0.8` y `gaussiano@0.2` (Δ ≥ 0,017, 1,5–2× su umbral).
- **La realización de la copia**: `oblicua@0.6` con dos copias da Δ −0,011 ± 0,022, «no se
  distingue», pero ese SE es del tamaño de la amplitud entre tipos de la fase 1 (0,039). La prueba
  de la realización tiene tan poco poder como el resto. Para `recorte@0.6` no importa.
- El eje de los trazos **no está acotado**: α = 1 es el máximo de opacidad; lo siguiente es grosor
  o número de trazos, no más α. `borrado` sube hasta p = 0,4, también el borde.
- 13 escritores compartidos entre train y val: generaliza a dígitos nuevos, no a escritores nuevos.

## Lo que quedó pendiente

- **Combinar** gaussiano en línea con recorte, que con lo medido es la continuación natural. El
  criterio lo dejó fuera («sería otro estudio»): va en un experimento nuevo.
- **No mueve `ESTADO.md`** (otra red, otro dato). Los avisos a Telegram de los cierres no salieron
  (las unidades se lanzaron desde una sesión de Claude Code, sin `BOT_TOKEN`); el `|| true` evitó
  que eso tumbara nada.

## Cómo se diseñó (lo que vale para repetirlo)

Tres generadores separados por semilla (init como fichero con huella; lotes `100 + s`; ruido
`1000 + 10·t + i`, +5000 por realización). El dígito no existe a 32×32 en este dato, así que el
trazo se dibuja como **máscara** a 32, se reduce a **cobertura** por bloque 4×4 y se compone
`x' = x + α·c·(v − x)`; `borrado`/`externos` van a nivel de bit por Binomial. Está probado que un
ruido de nivel 0 da **los mismos pesos que `limpio` bit a bit** en los nueve tipos. Detalle en
`ESPECIFICACION.md` y `REGLAS.md` del experimento.
