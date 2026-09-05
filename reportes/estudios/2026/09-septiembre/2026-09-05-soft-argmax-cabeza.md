# `soft-argmax` — leer las coordenadas como esperanza en vez de regresarlas

| | |
|---|---|
| **Inicio (UTC)** | 2026-09-04 23:39 |
| **Fin (UTC)** | 2026-09-05 04:07 |
| **Reloj** | 4 h 28 min (4,46 h de entrenamiento puro) |
| **Instancias alquiladas** | **0** — corrió en el droplet dev, 2 vCPU |
| **Coste real** | **0 $**. Aquí el coste es **reloj**, no factura |
| **Runs** | 3 (A · B · C), 37 épocas cada uno, semilla **1**, receta `plan40` |
| **Dataset** | `dirty1000-80px-16px-r20260827` (`sha256:3df67624…`) — el **no preprocesado** |
| **Artefactos** | [`foveal-vision/experimentos/2026-09-04-soft-argmax/`](https://github.com/stalinbeltran/foveal-vision/tree/main/experimentos/2026-09-04-soft-argmax) — pesos, `metrics.jsonl`, `resumen.json` y las figuras de mapas de calor |
| **Criterio** | escrito **antes** y empujado a `main` antes de la primera época: [`instrucciones/02-criterio.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/experimentos/2026-09-04-soft-argmax/instrucciones/02-criterio.md) |

> Este reporte **resume y enlaza**. El veredicto completo, con el desglose por posición, las
> figuras y las cinco limitaciones, vive en el
> [README del experimento](https://github.com/stalinbeltran/foveal-vision/blob/main/experimentos/2026-09-04-soft-argmax/README.md).

---

## Qué se preguntaba

Toda la familia de redes de este proyecto lee las coordenadas de la esquina **regresándolas**:
la cabeza es una `Linear` cuyas 12 salidas son 4 esquinas × `[exists, x, y]`. `soft-argmax` es
la alternativa clásica en detección de puntos clave: producir un **mapa de calor** por esquina,
pasarlo por un softmax y tomar la **esperanza** de la posición sobre una rejilla de coordenadas.

La pregunta, a igualdad de cuerpo y de presupuesto de cabeza: **¿baja `pos_err_px`?**

Se montó sobre `plana-20-4k7` —la más fuerte de la serie plana y la única con 4 canales— y
sustituyendo **sólo `(x, y)`**: `exists` se queda con su `Linear` de siempre, para que el f1
fuera el control.

## Qué salió: **el error de posición cae 3,9×, y NO es la pila conv**

Todos en la época que guarda su `best.pt` (elegida por `val_loss`, la misma regla en los cuatro).
`pos_err_px` va en píxeles de la ventana etiquetada de 16 px, sobre las **14.724 esquinas
verdaderas** de validación:

| | época | f1 | **`pos_err_px`** | borde | interior |
|---|---:|---:|---:|---:|---:|
| **ancla** `plana-4k7-s1` | 34 | 0,8398 | **2,224** | 3,088 | 1,961 |
| **A** soft-argmax | 37 | 0,8657 | **0,571** | 0,940 | 0,459 |
| **B** soft-argmax + dispersión (λ=0,1) | 34 | 0,8576 | **0,559** | 0,838 | 0,474 |
| **C** *control*: misma pila conv, lectura `Linear` | 37 | 0,8533 | **1,414** | 2,094 | 1,207 |

**El control C es lo que hace legible el resultado**, y se descompone limpio porque A y C
comparten todo menos la lectura:

```
ancla 2,224 px  ──(−0,810: la PILA CONV)──►  C 1,414 px  ──(−0,843: la LECTURA)──►  A 0,571 px
```

Las dos mitades son casi iguales y **se componen**. Y el soft-argmax gana su mitad con **1,66×
menos cabeza** que C (19.289 contra 32.096 parámetros): a C se le dio de más a propósito, para
que este resultado no pudiera atribuirse al tamaño.

⚠ **Referencia lejana**: la foveada de producción `fov16-mask-p20` —4 capas, mismo dataset,
misma métrica— llega a **1,120 px**. A la baja con una plana de **una** capa. ⚠ Pero detecta
mucho peor (f1 0,954 contra 0,866): son dos cosas distintas.

## ⚠⚠ El control de f1 FALLÓ, y así hay que leerlo

El criterio pedía que el f1 no se moviera más de ±0,01 respecto al ancla. **Se movió en los
tres**: A +0,0259 · B +0,0178 · C +0,0135.

**Pero se movió también en C, que no lleva soft-argmax**, así que lo que mueve el f1 es la pila
conv y la co-adaptación del cuerpo (compartido y entrenado), no la lectura. Aun así A y C
difieren +0,0124: **no se puede afirmar que el soft-argmax deje `exists` intacto**.

⚠ **Lo que el fallo NO contamina es el titular.** `pos_err_px` se promedia sobre las esquinas
marcadas por `exists_true` —la verdad del terreno—, así que **las cuatro redes se evalúan sobre
exactamente el mismo conjunto de 14.724 esquinas**. Que una detecte más o menos no cambia ni una
de las que entran en la media.

## El sesgo de contracción existe, está medido, y lo tapa la ganancia

Era el riesgo estructural: *medido el 2026-09-04 sobre el `windows.npz`*, el **24,1 %** de las
esquinas cae en el **primer o último píxel** de la ventana, y la esperanza de un softmax llega
mal a los extremos de su rejilla.

Se ve donde tenía que verse: el ancla ya sufre en el borde (**1,57×** su interior) y **A sufre
más (2,05×)**. ⚠ Pero el borde de A (0,940 px) es mejor que el **interior** del ancla (1,961 px):
el sesgo es real y queda **completamente tapado** por la ganancia. Sin el desglose habría sido
invisible.

Por eso la rejilla se diseñó abarcando la **vista entera** (`[-0,375 · 1,375]` en unidades de
fóvea) y no `[0,1]`: le da al softmax masa que colocar fuera del objetivo. ⚠ **No se corrió la
variante recortada**, así que cuánto aporta esa decisión está razonado, no medido.

## B: inerte en el agregado, pero **redistribuye**

−0,012 px global, **por debajo** del umbral de 0,15 → por el criterio, **B ≈ A**. ⚠ Pero el
desglose va por donde el mecanismo predice: el borde mejora un **11 %** (0,940 → 0,838), el
interior empeora un 3 %, y la β aprendida sube de 1,695 a 2,046 (mapas más picudos). Una semilla
y un solo λ: acota, no declara.

## ⚠⚠ El precio no es de parámetros: **+0,2 % de params, ×266 de CÓMPUTO**

| | params de cabeza | **MACs de cabeza / ventana** |
|---|---:|---:|
| ancla `Linear(1604, 12)` | 19.260 | **19.248** |
| A/B (pila conv → mapas) | 19.289 (+0,2 %) | **5.126.416 (×266)** |

Una `Linear` global se aplica **una vez** por ventana; una pila convolucional, **en cada una de
las 400 posiciones**. La red entera pasa a costar **×30**, y se midió con el reloj: **149-161
s/época** contra los **38,9** del ancla, en la misma máquina.

**Igualar parámetros no iguala cómputo.** Para un eje que se lee como «cabeza», la intuición
falla en la dirección cara — y es el número que mandaría si esto llegara a producción.

## Lo que quedó pendiente

1. **Una semilla por brazo.** El umbral (0,15 px) sale de la dispersión época a época del ancla
   —un **suelo**, no el ruido entre réplicas, que sigue sin medirse—. La ganancia de A es **11×
   ese suelo**, así que no es fragilidad de umbral; pero «acota, no declara» sigue valiendo. Lo
   que declararía: 5 semillas.
2. **Nunca se ha probado en la FOVEADA**, que es la red de producción y tiene 4 capas y una
   cabeza mucho mayor. Es el paso obvio y **no está hecho**.
3. **La rejilla extendida no tiene control** (`[0,1]` contra `[-0,375 · 1,375]`).
4. **λ del regularizador sin barrer**: 0,1 está *dimensionado* (8,3 % del término de posición al
   arrancar), no optimizado.
5. **El coste en cómputo no está optimizado.** Nadie ha mirado si una pila más barata —menos
   canales ocultos, kernel 3×3, o el mapa a media resolución— conserva la ganancia. Dado el ×266,
   es lo primero que habría que medir antes de proponer esto para nada real.
