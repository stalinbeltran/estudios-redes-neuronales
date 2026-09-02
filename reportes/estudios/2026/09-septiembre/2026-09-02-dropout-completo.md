# `do-v` — el estudio completo de `dropout`: el vigente se queda, y ahora con un `p`

*`#20` · tipo `estudios` · [índice de reportes](../../../README.md)*

| | |
|---|---|
| **Inicio (UTC)** | 2026-09-01 21:43:55 |
| **Fin (UTC)** | 2026-09-02 01:57:10 |
| **Reloj** | 253,1 min (4 h 13 min) — **de la flota entera**, ver §4 |
| **Instancias alquiladas** | **23** *(11 entrenaron; las 12 restantes fallaron al arrancar)* |
| **Coste real** | **1,8656 $ la flota entera** · ~**1,011 $** atribuibles a este estudio *(derivado, §4)* |
| Recorrido | `do-v` — 4 valores × 5 semillas = **20 runs**, 20/20 terminados |
| Dataset | `dirty1000-80px-16px-r20260827` |
| Base / receta | `ws16-p2-d2-L4` (168.652 params) · `plan40` |
| Criterio | escrito antes de mirar en [`foveal-vision/docs/plan-dropout-2026-08-28.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-dropout-2026-08-28.md) |

---

## 1. Qué se corrió, y por qué existía

El tanteo [`#17`](../../08-agosto/2026-08-28-dropout-tanteo.md) (2 semillas) ya apuntaba a que
`dropout` no ayuda. **`do-v` no lo repite**: aporta el punto **0,05**, que nadie había mirado
y era donde podía quedar una ganancia, y sube a **5 semillas**, que bajan el `p` mínimo
alcanzable de 0,333 a **0,0079** — lo que convierte «el vigente gana» en una declaración al
5 % en vez de una impresión.

El rango `{0 · 0,05 · 0,1 · 0,2}` **no se eligió al lanzar**: lo fijó la tabla que el plan
escribió antes de tener un número, y vive en `TABLA_PICO` dentro de
`foveal-vision/scripts/estudio_dropout.py`.

⚠ El recorrido estaba **creado desde el 2026-08-31 20:54 y sin lanzar**. Se verificó antes de
correrlo (`estudio_progreso.py --sweep do-v --tabla`: 0/20, `status: queued`) porque se dudaba
de si se había corrido y perdido con algún server. **No se había perdido: no se había corrido.**

## 2. Lo que salió

| `dropout` | f1 (media) | sem | min | max | épocas | s/época |
|---:|---:|---:|---:|---:|---|---:|
| **0,0** *(vigente)* | **0,9341** | 0,0022 | 0,9296 | 0,9416 | 47 · 48 · 54 · 58 · 70 | 36,5 |
| 0,2 | 0,9319 | 0,0045 | 0,9180 | 0,9430 | 46 · 58 · 59 · 70 · 85 | 56,1 |
| 0,1 | 0,9283 | 0,0069 | 0,9108 | 0,9454 | 34 · 35 · 41 · 74 · 88 | 53,4 |
| **0,05** *(el punto nuevo)* | **0,9248** | 0,0053 | 0,9119 | 0,9406 | 32 · 33 · 43 · 56 · 58 | 52,0 |

**Contraste contra el vigente `0,0`, permutación exacta (252 arreglos, `p` mínimo 0,008):**

| `dropout` | diferencia | `p` |
|---:|---:|---:|
| 0,2 | −0,0021 | 0,690 |
| 0,1 | −0,0058 | 0,421 |
| 0,05 | −0,0092 | **0,151** |

## 3. El veredicto

**El vigente `dropout = 0,0` se queda, y esta vez con capacidad de declarar.** Ninguno de los
tres valores alcanza significación al 5 %; el `p` más bajo es 0,151. Con 5 contra 5 el
contraste **sí podía** declarar, así que «no hubo señal» es un resultado y no una limitación
del diseño.

Tres cosas que este estudio añade a lo que ya decía el `#17`:

1. **El punto `0,05` es el PEOR de los cuatro** (0,9248), no un óptimo escondido entre 0,0 y
   0,1. Era la única razón de coste que justificaba los 20 runs, y queda cerrada.
2. **El eje no es monótono**: 0,0 → 0,05 → 0,1 → 0,2 da 0,9341 → 0,9248 → 0,9283 → 0,9319, o
   sea baja y vuelve a subir. Con 5 semillas eso ya no se puede achacar al azar tan fácilmente
   como con 2, y refuerza la sospecha del `#17` de que **`patience` fija es un confound** a lo
   largo de este eje: las épocas van de 32 a 88.
3. **`dropout` cuesta reloj**: 36,5 s/época con 0,0 contra 52-56 con el resto, o sea ~1,5×.
   Un mando que no mejora la calidad y encarece cada run.

⚠ **R1 ✅ el recorrido es válido**: los 20 runs pararon por `patience`, entre 32 y 88 épocas,
ninguno cerca del tope de 150. Miden calidad, no presupuesto.

**El eje `dropout` queda CERRADO.** El detalle del criterio y su lectura viven en el plan; este
reporte no los copia.

## 4. ⚠ El coste no se puede atribuir limpiamente, y hay que decirlo

Los cuatro recorridos de esta noche —`do-v` y los tres de strides del [`#21`](2026-09-02-strides-rama-tanteo.md)—
se corrieron en **una sola flota** (`estudio_flota.py --sweep` repetido). Eso significa que
**`flota.json` guarda el coste TOTAL en los cuatro recorridos por igual**: los cuatro dicen
`usd: 1.8656`, `maquinas_alquiladas: 23`, `reloj_min: 253.1`.

El 1,011 $ de la cabecera es un **reparto derivado**, no un dato: se calcula por minutos-máquina
leídos del log (`do-v` = 908,0 de 1608,1 min-máquina = 56,5 % de los 1,7898 $ que entrenaron).
Los 0,0758 $ restantes se fueron en **12 arranques fallidos** (`sshd` que no contestaba), que la
flota destruyó y apuntó en su lista negra; no son atribuibles a ningún estudio.

**Es una limitación de la herramienta, no de esta corrida:** una flota con varios `--sweep` no
sabe repartir su factura. Anotado para quien vuelva a lanzar así.

## 5. Lo que quedó pendiente

- **El confound de `patience`.** Las épocas van de 32 a 88 según el `dropout`, así que cada
  brazo recibió un presupuesto de entrenamiento distinto. Atacarlo pide un diseño 2-D
  (`dropout` × `patience`), y **no se ha hecho**.
- **El tanteo de `patience` (`pa-t`)** sigue pendiente, con su criterio ya escrito.
- **Nada que aplicar.** El vigente no se mueve, así que no hay cambio de configuración que
  llevar a ningún sitio.
