# `dim-gen` — reducir las dimensiones de la imagen contra la capacidad de generalizar, con el 10 % del dato para entrenar

| | |
|---|---|
| **Inicio (UTC)** | 2026-10-02 01:49:00 (primer alquiler) |
| **Fin (UTC)** | 2026-10-02 06:24:49 (última máquina destruida; el cierre terminó a las 06:25:46) |
| **Reloj** | 4 h 36 min. La primera oleada acabó a las 04:53; dos reintentos (`s1`, `s4`) lo estiraron |
| **Instancias alquiladas** | **13** en Vast.ai *(10 trabajaron; 3 murieron antes de trabajar: sshd sin responder ×2, clave SSH rechazada ×1)*. 12–18 vCPU, 0,0516–0,0649 $/h, Xeon E5-2673 v3 / E5-2680 v4 / E5-2686 v4 / Core i7-5820K |
| **Coste real** | **1,2023 $** *(del libro: 1,1883 $ las que trabajaron + 0,0140 $ los tres intentos fallidos)* |
| **Runs** | **30**: `W ∈ {128, 64, 32, 16, 8}` × semillas 1–5, más el control `w128-de16` × 5. 4000 pasos de lote 20, Adam `lr` 1e-3, L1, sin selección (`last.pt`) |
| **Dataset** | `parrafos1000-584px-r4-r20260908b` (publicado en el repo de datos), recortado a 128 px y reducido a `W` por suma exacta de bloques; **train 100 / val 900** |
| **Artefactos** | código, criterio, métricas, informe y libro: [`experimentos-cnn/2026-10-01-dimension-generalizacion/`](https://github.com/stalinbeltran/experimentos-cnn/tree/tema-2/2026-10-01-dimension-generalizacion) *(rama `tema-2` hasta fusionar)*; **los 30 pesos** y los logs, en el almacén: `foveal-vision-data/experimentos-cnn-resultados/dim-gen/` (rama `tema-2`) |
| **Criterio** | escrito **antes** de entrenar y commiteado el día anterior (`d43cfa4`), con dos enmiendas fechadas antes de la primera corrida (4000 pasos en vez de 1000; el control en máquina propia): [`instrucciones/02-criterio.md`](https://github.com/stalinbeltran/experimentos-cnn/blob/tema-2/2026-10-01-dimension-generalizacion/instrucciones/02-criterio.md) |

> Este reporte **resume y enlaza**. El veredicto entero, la tabla por semilla, el desglose por
> factor y las limitaciones viven en el
> [README del experimento](https://github.com/stalinbeltran/experimentos-cnn/blob/tema-2/2026-10-01-dimension-generalizacion/README.md).

---

## Qué se preguntaba

El dueño pidió (2026-10-01) un experimento «dedicado exclusivamente a medir el efecto que tiene
la reducción de las dimensiones de la imagen sobre la capacidad de generalizar de la cnn», con el
10 % del dataset para entrenar y el 90 % para validar, `L` capas fijas sin padding y un kernel
que ve siempre la misma fracción `f` del ancho de la imagen. El plan descartó `f = 0,1` con
`L = 4` (deja un mapa de 0,6·W y una cabeza que escala con W²) y usó la regla `f = 1/L = 0,25`:
mapa final 4×4 y cabeza de 516 parámetros constantes para todo `W`. Lo que crece con `W` son
los parámetros de las convoluciones (×152 entre 8 y 128 px), y por eso entró un **control**:
`w128-de16`, la imagen de 16 px repetida hasta 128 con la red de `w128`.

## Qué salió: **menos resolución, mejor generalización, hasta 16 px**

IoU de la caja predicha sobre las **900 imágenes no vistas**, media ± sd de 5 semillas:

| `W` | parámetros | IoU val | IoU train | brecha |
|---:|---:|---|---|---|
| 128 | 205.348 | 0,305 ± 0,047 | 0,515 | +0,21 ± 0,26 |
| 64 | 51.748 | 0,395 ± 0,072 | 0,740 | +0,34 ± 0,18 |
| 32 | 13.348 | 0,514 ± 0,061 | 0,850 | +0,34 ± 0,04 |
| **16** | 3.748 | **0,612 ± 0,049** | 0,723 | +0,11 ± 0,03 |
| 8 | 1.348 | 0,579 ± 0,076 | 0,669 | +0,09 ± 0,02 |
| control `w128-de16` | 205.348 | 0,496 ± 0,047 | 0,817 | +0,32 ± 0,03 |

Con el criterio escrito antes (piso 0,2464; umbral `max(2·SE_dif, 0,01)`):

- **`W* = 16`** y el **`W` mínimo suficiente es 8 px** (dentro del umbral de W\*).
- **La resolución estorba: SÍ** (128 px queda 0,31 por debajo de W\*, umbral 0,05), **y la
  brecha crece con `W`** (+0,09 a 8 px → +0,21 a 128; +0,49 en las semillas de 128 que
  entrenaron). Forma **(iii)**: máximo interior.
- **El control separa las dos causas y las dos existen**: la red grande con sólo 16 px de
  información pierde 0,116 respecto de `w016` (**capacidad**: más parámetros a igual
  información memorizan más, brecha +0,21 mayor) y con los 128 px de verdad pierde otros
  0,19 (**información fina**). Fracción del camino 16→128 que explica la capacidad: **0,38**
  (0,44 contra las semillas entrenadas). Es el confound que `ESTADO.md` deja «sin cerrar» en
  `border_reduce`; aquí, en otra red y otra tarea, un control lo reparte.

⚠ **Lo que el criterio no previó: en `w128` 3 de 5 semillas y en `w064` 1 de 5 no
entrenaron** (pérdida de train estancada en 0,137 = predecir una caja constante; IoU de train
0,30 = el piso). Mirado después y sin peso en el veredicto: sólo con las semillas entrenadas,
`w128` da 0,349 ± 0,051 y `w064` 0,426 ± 0,019, así que **el orden de la curva y el «estorba»
no cambian**; lo que cambia es la lectura de `w128`, que el criterio etiqueta «mixta» por una
caída de train que no es falta de información sino **fragilidad del optimizador con kernels de
32×32** (`lr` 1e-3, ReLU, inicialización por defecto). Las 5 semillas del control, con la
misma red, sí entrenaron.

## Lo que NO mueve

- **Ningún parámetro de `ESTADO.md`**: la red es la de este experimento (`experimentos-cnn`),
  no la foveada de `foveal-vision`, y la tarea es localizar una caja de tinta, que aguanta el
  desenfoque. Lo que aporta a `ESTADO.md` es un **precedente medido** de cómo separar
  capacidad de resolución con un control de información a parámetros fijos.
- Nada aplicado a producción.

## Lo que quedó pendiente

- **Separar «no entrena» de «no generaliza» a `W ≥ 64`**: repetir `w064` y `w128` con un `lr`
  o inicialización que entrene las cinco semillas (`3e-4`, o `LeakyReLU`). No cambia el
  veredicto; cambia cuánto de la caída de 128 px es optimizador. **No está hecho.**
- **El techo es 128 px** (el dato publicado ya es /4 del render): `W > 128` pediría publicar
  otro dato.
- **Fusionar `tema-2` en `main`** en los tres repos: hasta entonces, un server nuevo no ve nada
  de esto.
- Lo estimado contra lo real: el plan decía «≈2 h y 0,4–0,55 $ sin control»; con el control
  y dos reintentos fueron 4,6 h y 1,20 $. El coste por máquina que trabajó fue de 0,055 a
  0,20 $; las de 12 vCPU tardaron hasta el doble que las de 14–18 en el mismo brazo.
