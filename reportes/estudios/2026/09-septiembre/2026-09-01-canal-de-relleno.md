# `mask_channel` — decirle a la red qué píxeles se ha inventado (2026-09-01)

| | |
|---|---|
| **Qué se corrió** | **un** entrenamiento, `fov16-mask-p20`, para USARLO en inferencia |
| **Inicio (UTC)** | 2026-09-01 **02:10:02** *(del journal de systemd, no derivado)* |
| **Fin (UTC)** | 2026-09-01 **02:57:27** *(ídem)* |
| **Instancias alquiladas** | **1** (Vast 49468618, 12 vCPU / 16 GB, 0,0489 $/h) — y **1 entrenó** |
| **Coste real** | **0,0386 $** · 47,3 min de máquina |
| **Dataset** | `dirty1000-80px-16px-r20260827` |
| **Plan y criterio** | [`foveal-vision/docs/plan-mask-channel-2026-09-01.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-mask-channel-2026-09-01.md), escrito **antes** de mirar |

---

## Qué se preguntaba

El dueño, revisando detecciones a ojo, encontró que **rellenar el borde de la
imagen desvía el entrenamiento**: *«hace parecer que el párrafo se extiende»*.

Desglosado, el fallo estaba donde dijo y **sólo ahí** *(medido el 2026-09-01 sobre
el val, 28.000 ventanas; la medición reproduce exactamente el `val f1` y el
`pos_err_px` que el run registró)*:

| esquina a … del borde | esquinas | recall |
|---|---:|---:|
| **0–1 px** | 380 | **0,608** |
| 1–2 px | 393 | 0,977 |
| > 8 px | 11.130 | 0,939 |

⚠ **No era «las ventanas con relleno son malas»**: su f1 es 0,9490 contra 0,9453
de las que no llevan relleno. Molestaba **la esquina que cae dentro del relleno**.

## Qué se hizo

`pad_mode: edge` replica la fila del borde, y esa réplica es **por construcción**
indistinguible de imagen real que sigue — por eso `pad_mode` ya había salido plano
al barrerlo (`pad-t`, amplitud 0,0045, *«sin señal»*). La información que falta no
es *qué* píxeles inventas, es **que la red sepa que son inventados**.

Y esa información **ya se calculaba en cada paso y se tiraba**: `build_view`
devuelve la cobertura y todos los caminos hacían `view, _cov = ...`.

`mask_channel: coverage` la mete como **segundo canal de entrada** (el relleno,
`1 − cobertura`, celda a celda) **a la rama periférica**. ⚠ Sólo a ella, y no es
una economía: *medido con las dos geometrías vivas, bajo la máscara del centro la
cobertura es 1,000 en todas las celdas* — la fóvea está dentro de la imagen por
construcción. **+144 pesos, +0,085 %.**

## Qué salió

| run | `edge_inputs` | `mask_channel` | f1 global | err px | **recall 0–1 px** |
|---|---|---|---:|---:|---:|
| `demo-fov16-optimo` | off | off | 0,9475 | 1,214 | **0,6079** |
| `fov16-edge-p20` | pad | off | 0,9473 | 1,190 | **0,6737** |
| **`fov16-mask-p20`** | pad | **coverage** | **0,9543** | **1,049** | **0,9737** |

**El agujero del borde se cierra entero.** El recall del último píxel pasa de
0,608 a **0,974** — por encima del 0,937 del interior de la imagen. Y de propina
el f1 global sube (+0,0068) y el error de posición baja un **13,6 %** (1,214 →
1,049 px).

**Contra el criterio escrito antes de mirar**, que pedía recall > 0,75 en el tramo
0–1 px sin perder más de 0,005 de f1: **cumple con holgura** (0,974 y el f1
**sube**).

⚠ **Y `edge_inputs` solo no bastaba**, que era la pregunta de fondo: los 4
escalares a la cabeza mueven el tramo de 0,608 a 0,674 —**menos de una quinta
parte** del hueco—; el canal, que entra en la convolución, lo cierra. Es
consistente con que el problema esté **dentro** de las convoluciones y no en la
cabeza, pero ver §«lo que no contesta».

## Lo que este resultado NO establece

1. ⚠ **Una semilla no declara nada.** La mejora de f1 global (+0,0068) está por
   **debajo** del δ = 0,010 del proyecto y dentro del ruido típico entre semillas.
   **Como resultado de f1, esto no cuenta.**
2. **Lo que sí es difícil de atribuir a ruido es el borde**: +0,366 de recall en
   un tramo concreto, ~70× el ruido típico entre semillas, exactamente donde el
   mecanismo predecía y en ningún otro sitio. Los demás tramos se mueven ≤0,014.
3. **No separa el canal de `edge_inputs`**: la red lleva los dos. Si se quiere
   atribuir, hace falta el tanteo (`mk-t`, 2 modos × 2 semillas).
4. **No mide la métrica de tarea**, sólo f1 de ventana y el desglose. Y el f1 de
   ventana **infravalora** este fallo: una esquina perdida no empeora una caja,
   **borra el párrafo**. El 15,6 % de las imágenes tienen tinta tocando el borde.
5. **No mueve el vigente.** `demo-fov16-optimo` sigue siendo la red de referencia
   hasta que un estudio con semillas diga otra cosa. `fov16-mask-p20` está
   **aprobada para inferir**, que es otra cosa.

## Lo que quedó pendiente

- **`mk-t`**: el tanteo que convertiría esto en declaración (2 modos × 2 semillas,
  ≈0,2 $ a los 43 s/época medidos aquí). **No lanzado.**
- **La métrica de párrafo** sobre imágenes con tinta en el borde — la medida que
  diría cuánto vale esto de verdad. Pide una fuente con verdad, que este dev no
  tiene.
- **`do-v`** (dropout, 5 semillas) y **`ei-t`** siguen sin lanzarse: esto no los
  adelantó, los interrumpió.
- ⚠ **`pad_mode: zero` pinta un marco NEGRO** en estos datos (papel claro, mediana
  244), y la decisión C11 dice que no se usan ceros *«porque cero significa no hay
  tinta»*. O el convenio cambió o la nota está invertida. **Sin mirar.**
- ⚠ **`fv-train` reserva el nombre antes de validar el dataset**: un fallo de
  dataset deja un `runs/<name>/` muerto y el reintento contesta «ya existe». Es el
  callejón que la regla U5.9 dice que no debe existir. **Sin arreglar.**
