# `sc-t` · `sp-t` · `sd-t` — los strides de rama: la cabeza se puede encoger a un tercio

*`#21` · tipo `estudios` · [índice de reportes](../../../README.md)*

| | |
|---|---|
| **Inicio (UTC)** | 2026-09-01 21:43:55 |
| **Fin (UTC)** | 2026-09-02 01:57:10 |
| **Reloj** | 253,1 min (4 h 13 min) — **de la flota entera**, ver §5 |
| **Instancias alquiladas** | **23** *(11 entrenaron; las 12 restantes fallaron al arrancar)* |
| **Coste real** | **1,8656 $ la flota entera** · ~**0,779 $** atribuibles a estos tres *(derivado, §5)* |
| Recorridos | `sc-t` · `sp-t` · `sd-t` — 4 valores × 2 semillas cada uno = **24 runs**, 24/24 terminados |
| Dataset | `dirty1000-80px-16px-r20260827` |
| Base / receta | `ws16-p2-d2-L4` (168.652 params, 12.800 features) · `plan40` · `epochs` 300 |
| Criterio | escrito antes de mirar en [`foveal-vision/docs/plan-strides-rama-2026-09-01.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-strides-rama-2026-09-01.md) |

---

## 1. Por qué existía

Petición del dueño: **el número de features que llegan a la cabeza es desproporcionado**. Lo
es — *medido el 2026-09-01 con `fv.models.network_trace`*: la base vigente manda **12.800
features a una cabeza de 12 salidas** (1.067 : 1), y esa única `Linear` se lleva el **91,1 %**
de los 168.652 parámetros. `s_center`/`s_periph` la encogen submuestreando cada rama.

El inventario los tenía en el puesto 12, medidos con 1 semilla y borrados, descritos como
*«mandos de coste, no de calidad»*. Este tanteo los reabre por el otro lado.

**Tres recorridos**, porque los dos ejes por separado no contestan si los efectos suman:

| recorrido | qué mueve | ancla |
|---|---|---|
| `sc-t` | `s_center` ∈ {1,2,3,4} | `s_periph` = 1 |
| `sp-t` | `s_periph` ∈ {1,2,3,4} | `s_center` = 1 |
| `sd-t` | **las dos a la vez** (diagonal, vía `couple`) | — |

## 2. Lo que salió

**f1 medio (2 semillas), por recorrido:**

| `s` | `sc-t` (`s_center`) | `sp-t` (`s_periph`) | **`sd-t` (diagonal)** | features (diagonal) |
|---:|---:|---:|---:|---:|
| 1 | 0,9312 ±0,0007 | 0,9315 ±0,0010 | 0,9315 ±0,0010 | 12.800 |
| 2 | 0,9294 ±0,0084 | **0,9330** ±0,0013 | **0,9358** ±0,0001 | **3.200** |
| 3 | **0,9359** ±0,0038 | 0,9282 ±0,0042 | 0,9294 ±0,0042 | 1.568 |
| 4 | 0,9302 ±0,0024 | **0,9052** ±0,0006 | **0,8813** ±0,0022 | 800 |

**R1 ✅ los tres recorridos son válidos**: los 24 runs pararon por `patience`, entre 33 y 72
épocas, ninguno cerca del tope de 300. Subir el tope de 150 a 300 resultó innecesario en la
práctica, pero era la decisión correcta a priori: nadie sabía dónde pararían.

## 3. Lo que dice, y lo que NO puede decir

⚠⚠ **Esto es un TANTEO de 2 semillas: el `p` mínimo alcanzable es 0,333. NADA de aquí declara
significación.** Acota; no cierra.

**El resultado que importa: el diagonal `s`=2.** Da **el mejor f1 de los 24 runs**
(0,9358 frente a 0,9315 del vigente) con **el 31,7 % de los parámetros** (53.452 contra
168.652) y **0,72× el s/época** (27,7 contra 38,3, misma tabla). Cumple las dos condiciones
que el plan escribió antes de mirar para llevar un punto a validación: queda dentro de δ del
vigente —de hecho lo supera— y recorta muy por encima del 30 % exigido.

⚠ Y la consistencia entre semillas es llamativa: **±0,0001** (0,9356 y 0,9359).

**El eje tiene un borde, y está entre 3 y 4.** Con `s`=4 la periferia sola cae a 0,9052 y la
diagonal a 0,8813. No es ruido: los SE son 0,0006 y 0,0022. Que el rango llegara hasta 4 —por
la lección de `borde-ancho`— es lo que permite ver ese borde en vez de suponerlo.

**¿Suman? A `s`=2 hay redundancia; a `s`=4, interacción negativa.** Con `Δ` = pérdida respecto
a `s`=1:

| `s` | Δ(`sc`) | Δ(`sp`) | suma | **Δ(`sd`)** | lectura |
|---:|---:|---:|---:|---:|---|
| 2 | +0,0018 | −0,0015 | +0,0003 | **−0,0043** | la diagonal pierde MENOS que la suma → **redundancia entre ramas** |
| 3 | −0,0047 | +0,0033 | −0,0014 | +0,0021 | interacción negativa leve |
| 4 | +0,0010 | +0,0263 | +0,0273 | **+0,0502** | la diagonal pierde casi el DOBLE → **interacción negativa fuerte** |

O sea: **el signo del efecto conjunto cambia con `s`**. Recortar las dos ramas a la mitad sale
gratis o mejor; recortarlas a un cuarto sale mucho más caro que la suma de sus partes. Los dos
estudios simples solos no habrían visto ninguna de las dos cosas — que era justo el argumento
para correr el tercer recorrido.

**Y la asimetría entre ramas es la esperada, pero por poco.** A `s`=2 gana la periferia
(0,9330 contra 0,9294): submuestrear el anillo —que el diseño declara *de contexto*— duele
menos que submuestrear la fóvea. A `s`=4 la periferia se desploma y el centro no, lo cual
**invierte** esa lectura y no tiene explicación en estos datos.

⚠ **`sc-t` es plano**: amplitud 0,0065, por debajo del umbral de 0,010 que el criterio fijó
como «meseta». Por esa regla, **`s_center` por sí solo se cierra en el tanteo** como mando de
coste puro. Su ganador aparente (`s`=3, 0,9359) tiene un SE de 0,0038 y un rango entre semillas
de 0,9321–0,9397: no es un pico, es ruido.

## 4. ⚠ Un hallazgo que no se buscaba: el mismo run dio dos resultados

Los tres recorridos comparten el brazo `s`=1, que es **exactamente la misma red** (`s_center`=1,
`s_periph`=1), el mismo dataset, la misma receta y las mismas semillas. Es una **réplica técnica
gratuita**, y no salió idéntica:

| | semilla 1 | semilla 2 |
|---|---|---|
| `sc-t` | 0,9305 (ckpt ép. 44) | **0,9319 (ckpt ép. 31)** |
| `sp-t` | 0,9305 (ckpt ép. 44) | **0,9326 (ckpt ép. 37)** |
| `sd-t` | 0,9305 (ckpt ép. 44) | **0,9326 (ckpt ép. 37)** |

La semilla 1 coincide en los tres, bit a bit. **La semilla 2 divergió en uno de los tres**, y no
sólo en el f1: paró en otra época. Y no es la familia de CPU — `sp-t-s2` corrió en un **E5-2670
v3** y `sd-t-s2` en un **E5-2697 v4**, y esos dos **coincidieron entre sí**. Las 19 CPU del
barrido fueron todas E5-26xx v3/v4 (el `--sin-cpu v2` funcionó).

**Por qué importa:** `--cpu E5-26` se usa porque *«dentro de la familia el entrenamiento sale
IDÉNTICO bit a bit entre máquinas»*. Aquí no salió. Dos consecuencias:

1. **Hay una medida empírica del ruido NO atribuible a la semilla: ~0,0007 en f1.** Es un suelo
   mucho más útil que el SE de 2 semillas — que en `sd-t` dio **0,0001**, un δ tan pequeño que
   deja fuera a cualquier punto y hace inservible la regla del punto de corte.
2. **La afirmación del determinismo necesita revisarse**, o acotarse con la condición que aquí
   no se cumplió. No se ha investigado la causa.

## 5. ⚠ El coste no se puede atribuir limpiamente

Los cuatro recorridos de esta noche —estos tres y el `do-v` del [`#20`](2026-09-02-dropout-completo.md)—
corrieron en **una sola flota**, y `flota.json` guarda el coste TOTAL en los cuatro por igual
(`usd: 1.8656` en los cuatro). El 0,779 $ de la cabecera es un **reparto derivado** por
minutos-máquina leídos del log (700,1 de 1608,1 = 43,5 % de los 1,7898 $ que entrenaron). Los
0,0758 $ restantes se fueron en 12 arranques fallidos, no atribuibles a ningún estudio.

## 6. Lo que quedó pendiente

- **`sd-v`: la validación de 5 semillas del diagonal.** Es el siguiente paso obvio y el único
  que puede declarar. Sin él, «el diagonal `s`=2 mejora y cuesta un tercio» es una acotación,
  no un resultado. **Nada se ha aplicado a la base vigente.**
- **El control iso-features NO se corrió**, conforme a la regla R5 del plan: sólo hacía falta si
  el f1 bajaba, y en el punto que interesa (`s`=2) sube. Sigue haciendo falta para explicar la
  caída de `s`=4, que **queda sin atribuir** entre resolución y capacidad.
- **La inversión de la asimetría entre `s`=2 y `s`=4** (a 2 aguanta mejor la periferia, a 4 el
  centro) no tiene explicación en estos datos.
- **`merge: sum` sigue sin medirse en condiciones.** Es el competidor directo: deja 6.400
  features (0,545× los parámetros) **sin tocar la resolución**. En disco hay `mrg-t` y `mrg-v`
  que se dan la vuelta y `mrg-v` nunca terminó (4/6), sobre el dataset anterior. Combinado con
  el diagonal `s`=2 daría 1.600 features (0,203×).
- **`stride_range` sigue roto para este eje**: devuelve `[1]` con `n_layers`=4, así que
  `"s_center": "auto"` entrenaría N veces la misma red sin avisar. Documentado en el §3.4 del
  plan, **no arreglado**; el rango va explícito.
