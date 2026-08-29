# ESTADO — en qué quedó cada parámetro, hoy

**Este documento SE REESCRIBE.** Refleja lo que está fijado *ahora*; no lleva historial dentro.
El historial es la tabla cronológica de [`reportes/README.md`](reportes/README.md), que sólo se
añade. Están separados a propósito (R8): mezclar los dos es lo que hacía que la fila de un
parámetro corregida el 28-ago conviviera con la versión vieja en el mismo fichero.

Última revisión: **2026-08-29** (traído entero desde `telegram-coordinator/reportes/README.md`
al centralizar; el contenido no se tocó).

---

## Tabla resumen: en qué quedó cada parámetro

**Para qué sirve esta sección:** los valores de aquí son los que se van a **usar en los
entrenamientos y estudios siguientes**, así que interesa poder leer de un vistazo *qué está fijado,
con cuánta evidencia, y qué sigue abierto* — y poder saltar al reporte que tiene el detalle.

⚠ **Esto no decide nada: refleja lo decidido.** El **vigente** de cada parámetro lo fija la
configuración del repo que entrena (`base_network_value` y `base_recipe_value` de cada
`sweeps/*/spec.json` en `foveal-vision`), y el **veredicto** vive en el documento de plan que
escribió su criterio antes de mirar. Aquí se resume y se enlaza; si una casilla y su fuente
discrepan, manda la fuente.

### Cómo leer la columna «estado»

| estado | significa |
|---|---|
| **cerrado** | barrido con 5 semillas y con el óptimo **interior** al rango: hay un valor peor por arriba y otro por abajo. Es lo más firme que da el protocolo |
| **cerrado por un lado** | acotado sólo por un extremo; por el otro el ganador está en el borde y no se sabe qué hay más allá |
| **tanteo** | 2 semillas. **Acota, no declara ganador** — con 2 contra 2 el *p* mínimo alcanzable es **0,333** |
| **sin cerrar** | medido, pero el recorrido no llegó a poder declarar (semillas incompletas, o *p* mínimo alcanzable por encima del 5 %) |
| **sin medir** | ninguna medida con semillas |

Y dos reglas del proyecto que explican por qué hay ganadores nominales que **no** mueven el vigente:

- **El vigente sólo cambia si `p` < 0,05 y la diferencia supera δ.** Un ganador con `p` = 0,063 no
  mueve nada, aunque gane dos veces seguidas (es el caso de `border_px` = 8).
- **Todo esto es f1 de VENTANA, un proxy** que está medido que **exagera** (en `n_layers` la
  ganancia real fue la mitad). **Ningún eje ha pasado todavía por la métrica de tarea** (R5).

### Red foveada (`ws16-p2-d2-L4` · `regions: split` · ≈167.852 parámetros)

Recorte real 24×24 px, tensor N = 20. Es la base de todos los estudios de la tabla cronológica salvo
los marcados como «plana».

| Parámetro | Vigente | Estado | Óptimo medido | Rango útil / qué se sabe | Reporte |
|---|---:|---|---|---|---|
| **`lr`** | **0,0014** | **cerrado** | 0,0014 | Plano entre 0,00035 y 0,0014; por encima degrada (0,0020 → 0,9055, 0,0028 → 0,8998). Bandas disjuntas. `p` = 0,100 es el **suelo** de 3×3, no un empate | [#3](reportes/estudios/2026/08-agosto/2026-08-23-lr-alto-L4.md) · [#4](reportes/estudios/2026/08-agosto/2026-08-23-lr-alto-L4-b.md) |
| **`batch_size`** | **85** | **cerrado** | 85–192 (zona plana) | Plano entre 57 y 192 (0,9302–0,9351); 38 pierde (`p` = 0,024) y de 192 arriba **baja monótono** (384 · 768 · 1536). ⚠ **Utilizable ya: 192 va 1,08× más rápido por época sin pérdida medible** | [#5](reportes/estudios/2026/08-agosto/2026-08-24-tres-ejes-pasada1.md) · [#8](reportes/estudios/2026/08-agosto/2026-08-25-bs-alto-tanteo.md) |
| **`n_layers`** | **4** | **cerrado** | 4 | 2 → 0,9066 (`p` = 0,008) · 3 → 0,9246 (`p` = 0,040) · **4 → 0,9341** · 5 → 0,9136. ⚠ **A partir de 5 no arranca de forma fiable**: `sem` 7× mayor, semillas bimodales | [#5](reportes/estudios/2026/08-agosto/2026-08-24-tres-ejes-pasada1.md) |
| **`border_px`** | **4** → **8** ⚠ | **CERRADO al 5 %** | **8** *(no aplicado)* | Sube hasta 8 px y **baja** de ahí (10 · 12 · 16). ⚠ **La `p` = 0,063 medida DOS veces queda RESUELTA en el [#14]**: con **10 semillas contra 10** sobre `r20260826`, 8 → 0,9398 contra 4 → 0,9302, **`p` = 0,006** y Δ = +0,0096 > δ. A coste constante en parámetros y **1,33× más rápido por época** | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) · [#9](reportes/estudios/2026/08-agosto/2026-08-26-borde-ancho.md) |
| **`border_reduce`** | **2** | **sin cerrar** *(confundido)* | 1 por f1, **pero no comparable** | Con `border_px` = 8: 4 → 0,9408 · 2 → 0,9472 · **1 → 0,9574** (`p` = 0,008, el mínimo alcanzable). ⚠ **NO es cost-neutral**: N pasa de 20 a 32, **+156 % de parámetros** y 1,77× por época. Capacidad y resolución están **confundidas** en este diseño | [#11](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2.md) |
| **`k_center`** | **3** | **cerrado** | 3 | 3 → 0,9341 · 5 → 0,9226 (`p` = 0,024) · 7 → 0,9206 (`p` = 0,008). Los dos alternativos son peores **y más caros**. ⚠ **Contradice el indicio de 1 semilla de julio**, donde 5 era el mejor por métrica de tarea: eso sigue sin resolverse, porque esto mide el proxy | [#11](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2.md) |
| **`channels`** | **[16]×4** | **cerrado** *(20/20)* | [16]×4 | Subir no aporta: 24 → 0,9303 (`p` = 0,40) y 32 → 0,9307 (`p` = 0,53), a 1,3× y 1,7× por época. **Bajar sí hace daño**: 8 → 0,9021 (`p` = 0,008) y con el `sem` más alto de la tabla. **16 es el suelo útil, no un exceso heredado**. Recalculado con **las 5 semillas de [32]×4**: el veredicto no cambió | [#13](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md) |
| **`pos_weight`** | **1,0** | **cerrado por arriba** | 1,0 | Monótono decreciente: 2 empata (`p` = 0,889), 4 → 0,9137 y 8 → 0,8780, los dos a `p` = 0,008. ⚠ Era **«la hipótesis más plausible de mejora grande sin probar»** y **no la hubo**: el cuello de botella de detección no se destapa desde el peso de la pérdida | [#13](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md) |
| **`scheduler`** | **`none`** | **cerrado** *(2 valores)* | `none` | `cosine` → 0,9329 contra 0,9341, `p` = 0,857. ⚠ **El empate es real, no un artefacto**: el tope se bajó a 100 a propósito para que `cosine` llegara a aplicar su bajada (con 150 habría medido «cosine casi sin aplicar») | [#13](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md) |
| **`monitor`** | **`val_loss`** | **cerrado** *(2 valores)* | `val_loss` | `val_f1` → 0,9346 (+0,0059, `p` = 0,214). ⚠ Y **el brazo `val_f1` partía con ventaja mecánica** —elige checkpoint con la misma métrica con que se le puntúa— y aun así no llega. La incoherencia declarada cuesta ~0,006 y **es indistinguible del ruido** | [#11](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2.md) |
| **`overlap_fovea_px`** | **2** → **7** ⚠ | **el óptimo es la PARED LEGAL** *(#14)* | **7** *(no aplicado)* | **Barrido ENTERO {0,1,2,4,5,6,7} sobre `r20260826`, y sube MONÓTONO**: 0 → 0,9236 · 1 → 0,9278 · 2 → 0,9332 · 4 → 0,9375 · 5 → 0,9400 · 6 → 0,9415 · **7 → 0,9433**. Con **10 semillas contra 8**: `p` < 0,001, Δ = +0,0124. **Cost-neutral: 167.852 parámetros en TODO el rango** (medido). ⚠ **El óptimo es el máximo LEGAL** (`overlap_fovea_range(16)` = [0..7]): no está acotado por evidencia sino por la geometría — apunta a que quien limita es el tamaño de fóvea. ⚠ El dato viejo decía otra cosa (4 con `p` = 0,270) | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) · [#13](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md) |
| `fovea_px` | 16 | **no barrible** | — | Atado por contrato al `window_size` del dataset. Cambiarlo exige **regenerar el dataset** | — |
| **`stride`** *(de extracción — B, no el de inferencia ni el de las convoluciones)* | **5** | **cerrado por un lado** *(25/25)* | **1** *(no aplicado)* | Monótono y sin rupturas: 16 → 0,8901 · 8 → 0,9159 · 4 → 0,9277 · 2 → 0,9328 · **1 → 0,9383**. 1 contra 16: `p` = **0,0079**. ⚠ **Gana el extremo**: dentro de δ = 0,0026 no queda ningún otro brazo, así que no hay saturación en el rango y el eje **no cierra por arriba** — con ventana 16 no se puede mirar más denso que 1 px. No es eje de `space`: es campo de **B**, un dataset por valor | [#16](reportes/estudios/2026/08-agosto/2026-08-27-stride-estudio.md) |
| `overlap_border_px`, `merge`, `pool_mode`, `pad_mode` | 0 · concat · avg · edge | **medidos** *(#14)* | los cuatro vigentes | Tanteo + verificación: **ninguno mueve el vigente**. `overlap_border_px` 2 daba +0,0158 con 1 semilla y +0,0055 con 4 (`p` = 0,314). ⚠ **`merge: sum` EMPATA con 0,54× de parámetros** (91.052 contra 167.852): no gana, pero que empate es la noticia. `pool_mode: max` pierde 0,023 | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) |
| **`dropout`** | **0,0** | **tanteo** *(pendiente `do-v`)* | **0,0** | Primera medida del parámetro. 0,0 → 0,9315 · 0,25 → 0,9282 · 0,5 → 0,9274 · **0,1 → 0,9129** (el peor: el eje **no es monótono**). Amplitud 0,0186, casi el doble del umbral de 0,010 — el eje mueve la aguja, **hacia abajo**. ⚠ **Lo que de verdad se midió es el mecanismo**: la brecha val/train pasa de **+29,5 %** (con 0,0) a **−2,6 %** ya con 0,1, o sea que regulariza como se diseñó, **y el f1 no sube**. Y el `train_loss` casi se dobla: no redistribuye capacidad, la pierde. Cierra —con `weight_decay`— los dos mandos de regularización, y con un mecanismo medido detrás. ⚠ **`patience` = 10 no es neutral en este eje** (34 a 82 épocas): parte de lo medido es eso | [#17](reportes/estudios/2026/08-agosto/2026-08-28-dropout-tanteo.md) |
| `k_periph`, `s_center`, `s_periph` | 3 · 1 · 1 | **NO BARRIBLES hoy** | — | ⚠ **No es que salieran mal: tienen UN SOLO valor legal** con la geometría vigente (comprobado con `build_search_space`). La banda periférica son 4 px, y un kernel/stride debe caber en ~la mitad. Serían barribles con el borde ancho — o sea que **`border_px` = 8 los acerca** | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) |
| `optimizer`, `weight_decay`, `lambda_pos`, `smooth_l1_beta` | adam · 0,0 · 1,0 · 0,08 | **medidos** *(#14)* | los cuatro vigentes | **Ninguno mueve el vigente.** ⚠ **`weight_decay` = 0 GANA y subirlo hunde** (0,001 → 0,8731). ⚠ **Y desde el [#17] esto ya no se lee como «la regularización no ayuda porque no hace falta»**: `dropout` **sí** cierra la brecha val/train y el f1 tampoco sube, así que lo medido es que **la brecha no es el cuello de botella**. `adam ≡ adamw` con `wd` = 0: el **control** salió bien | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) · [#17](reportes/estudios/2026/08-agosto/2026-08-28-dropout-tanteo.md) |
| **`patience`** | **10** | ⚠ **tanteo, y ABIERTO POR ARRIBA** *(no «medido»)* | **20** *(el borde del rango)* | ⚠ **Esta fila decía «medido, ninguno mueve el vigente» y escondía la mitad interesante; corregida el 2026-08-28 leyendo `pat-t` e `pat-v`.** **`20` gana LAS DOS VECES** y de forma consistente (+0,0028 y +0,0027 sobre el vigente), con el eje **monótono** — y **20 es el techo del rango: nadie ha mirado más allá**. Pero ninguna declara (`p` = 0,667 y 1,000; con 2 semillas el suelo es 0,333) y **+0,0027 está por debajo de δ**. `5` pierde las dos veces (−0,0105, −0,0041): confirma el mínimo de 8. ⚠ **`pat-v` nunca terminó (6/9: le falta la semilla 3 entera)** y sus semillas se diseñaron para SUMAR a las de `pat-t` — **nunca se sumaron**, porque el informe trabaja sobre un recorrido. Hay **4 semillas pagadas sin analizar juntas**, y sobre `r20260826`, que se perdió. ⚠ **Único eje NO cost-neutral por reloj**: más `patience` = más épocas = más dinero | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) |
| `momentum`, `epochs` | 0,9 · 100 | **no procede** | — | `momentum` es **inerte** con `adam`. `epochs` es **guarda, no ajuste, y hoy no ata**: medido sobre los 630 runs con curvas, la época más alta es **130** y **ninguno** llegó a 150, así que subir el tope daría runs idénticos | [#14](reportes/estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) |

### Red plana de control (`plana-24-single` · `regions: single` · ≈165.430 parámetros)

Existe para responder *la* pregunta del proyecto. Está afinada sólo a nivel de **tanteo**, y por eso
la comparación todavía no se puede hacer.

| Parámetro | Vigente | Estado | Óptimo del tanteo | Qué se sabe | Reporte |
|---|---:|---|---|---|---|
| **`lr`** | **0,0014** | **tanteo** | 0,0007 | Zona útil **0,00035–0,0014** (0,9615–0,9649, todo dentro del ruido). ⚠ En **0,0028 una semilla colapsó a f1 = 0,0000** mientras la otra dio 0,9442: ahí el entrenamiento **puede colapsar entero**. El óptimo de la foveada (0,0014) cae dentro de la zona buena pero **no es el mejor**: un óptimo **no se hereda** al cambiar de arquitectura | [#7](reportes/estudios/2026/08-agosto/2026-08-25-plana-tanteo.md) |
| **`batch_size`** | **85** | **tanteo, acotado** | **170** *(interior)* | 24 → 0,9510 · 43 → 0,9581 · 85 → 0,9626 · **170 → 0,9658** · 340 → 0,9601. Y 170 es además **más barato por época** que el vigente (50,2 s contra 85,2) — el mismo patrón que en la foveada. Réplica exacta con `bs-alto-pl`, otra flota | [#10](reportes/estudios/2026/08-agosto/2026-08-26-plana-tanteo-fase1.md) · [#8](reportes/estudios/2026/08-agosto/2026-08-25-bs-alto-tanteo.md) |
| **`n_layers`** | **4** | **tanteo, acotado** | **5** *(interior, pero al borde de lo fiable)* | 2 → 0,9196 · 3 → 0,9521 · 4 → 0,9615 · **5 → 0,9659** · 6 → **bimodal**. ⚠ **L6 dio 0,0000 y 0,9630 en sus dos semillas**: la media de 0,4815 es el promedio de una moneda y **no debe citarse**. El ganador 5 está justo en el borde de esa zona, y 2 semillas es **exactamente el número que no puede ver la bimodalidad** | [#10](reportes/estudios/2026/08-agosto/2026-08-26-plana-tanteo-fase1.md) |

### Inferencia — se ajustan **sin reentrenar** (dominio F)

Es el mejor ratio ganancia/coste del inventario, y está **deliberadamente sin aplicar**.

| Parámetro | Default vigente | Óptimo medido hoy | Nota |
|---|---:|---|---|
| **`threshold`** | 0,5 | **0,25 – 0,40** ⚠ *ya no coincide entre modelos* | El pico está medido como **plano entre 0,2 y 0,4**, así que la discrepancia puede ser ruido |
| **`stride`** | `n/2` (8 px) | **2 px** | Los dos runs válidos coinciden |
| **`nms_radius`** | `n/2` | **16 px** | Los dos runs válidos coinciden |
| `min_size` | 4,0 | sin medir | — |

**Ganancia medida con los pesos que ya hay: +0,053 a +0,071** de métrica de tarea.

⚠ **Dos avisos que pesan más que la ganancia**, los dos del reporte [#12](reportes/estudios/2026/08-agosto/2026-08-26-knobs-f.md):

1. **El óptimo ya NO es el mismo para todos los modelos, y en julio sí lo era.** Si el `threshold`
   óptimo depende del modelo, los knobs dejan de ser un ajuste global y pasan a ser **parte de la
   identidad de cada run** — y eso afecta a **cómo se comparan modelos**, no sólo a cuánto valen.
   ⚠ **Con dos runs válidos no se puede afirmar**; la re-corrida con tres estaba en marcha.
2. **La decisión F15 está CERRADA en NO** (del usuario, 2026-07-26): aplicarlos **re-escala todos los
   números publicados**, y hay un efecto medido de que **comprimen la separación entre modelos**.

### Lo que sigue abierto, en una pantalla

1. **La pregunta que da nombre al proyecto sigue sin contestar** — ¿gana la foveada a la plana? Está
   bloqueada por la **fase 2 de la plana** (5 semillas sobre `batch_size` ∈ {85, 170, 340} y
   `n_layers` ∈ {4, 5, 6}), **que no se ha corrido**. ⚠ Y **los 0,96 de la plana contra los 0,93 de
   la foveada NO son esa comparación**: son f1 de ventana y las dos redes ven áreas distintas.
2. **Ningún eje ha pasado por la métrica de tarea (R5).** En `k_center` no es opcional: es el eje
   donde proxy y tarea se contradicen **en el signo**.
3. **`border_px` = 8 contra 4 sigue sin resolverse al 5 %**, con `p` = 0,063 medida **dos veces**. Lo
   que lo cerraría es **más semillas en esos dos puntos** (10 contra 10 da `p` mínimo **1,08·10⁻⁵**), **no**
   un rango más ancho.
4. ✅ **`overlap_fovea_px` ya declara, y dice que el solape aporta** (`p` = 0,032 para el punto 0).
   El run que faltaba —`overlap`=4 semilla 3— lo corrió la propia flota antes de cerrar a las
   15:58:19 UTC, así que el recorrido está en **20/20**, igual que `channels`. ⚠ **Lo que sigue
   pendiente y es gratis: volver a pasar `estudio_informe.py`.** Las tablas de arriba se
   recalcularon a mano desde `runs/*/metrics.jsonl`; los `sweeps/*/informe.json` en disco son de
   las **15:13 UTC**, o sea de antes del último run, y aún dicen que `ov-fov` tiene 2 semillas en su
   punto ganador. **Cero alquileres, y quita la última fuente rancia de este inventario.**
   ⚠ Por arriba el eje **no queda cerrado**: 4 es el borde del rango. Cerrarlo pide un punto más
   allá, decidido **antes** —como en `borde-ancho`—. ⚠ **Y no hay coste que sopesar: corregido el
   2026-08-26, este eje es cost-neutral.** Aquí decía «contra su coste, porque el solape sube N», y
   es falso: `N = fovea_px + 2·(border_px // border_reduce)` **no contiene el solape**. Medido
   contando parámetros en los ocho valores legales: **167.852 en todos**. Lo que crece es la banda
   de la rama periférica, no el tensor. El tope de este eje lo pone la geometría —
   `overlap_fovea_range(16)` = [0…7]— y no el presupuesto.
5. **El confound de `border_reduce` sigue abierto**: capacidad contra resolución. Desconfundirlo pide
   un diseño que suba N sin subir el área, o que compare a parámetros igualados.
6. **`do-v` está creado y sin lanzar** — el estudio de 5 semillas de `dropout` sobre
   {0 · 0,05 · 0,1 · 0,2}. 20 runs, ≈1,1 $. El tanteo [#17] dejó el vigente `0,0` ganando, pero
   con 2 semillas no declara, y falta mirar **`0,05`**, que nadie ha visto.
7. ⚠ **`patience` NO estaba «medido»: está abierto por arriba, y hay 4 semillas pagadas sin
   analizar.** `20` gana a `10` en las **dos** medidas independientes (+0,0028 y +0,0027), el eje
   sube monótono y **20 es el borde del rango**. Ninguna declara (2 semillas cada una) y `pat-v`
   **nunca terminó** (6/9: falta la semilla 3 entera). Sus semillas se diseñaron para **sumar** a
   las de `pat-t` y **nunca se sumaron** — el informe trabaja sobre un recorrido. Y todo eso está
   sobre `r20260826`, que se perdió: hay que **re-medir**. El siguiente paso es un **tanteo**
   `{10 · 20 · 40}` con **`epochs` subido a 300** (con `patience` 20 los runs ya llegaban a 92
   épocas; con 40 pasarían del tope de 150 y **fallarían R1**), y con el criterio de **quedarse
   ahí** si la amplitud no llega a 0,010 o si `40` no mejora a `20`. Está escrito entero en
   [`telegram-coordinator/CLAUDE.md`](https://github.com/stalinbeltran/telegram-coordinator/blob/main/CLAUDE.md).
8. **Y `patience` es además un CONFOUND a lo largo de los demás ejes** — medido en el [#17]: las
   épocas fueron de 34 a 82 según el `dropout` (factor 2,4), o sea que **cada brazo de un eje
   recibe un presupuesto de entrenamiento distinto**. Eso es otro estudio (2-D, o cambiar la regla
   de comparación) y **no lo contesta un tanteo 1-D de `patience`**.

### Dónde está el detalle de cada cosa

Esta tabla resume; el detalle está en tres sitios, y cada uno contesta algo distinto:

| Si necesitas… | Mira en |
|---|---|
| **qué se corrió, cuándo, con cuántas máquinas y qué costó** | el reporte de la tabla cronológica de abajo |
| **el veredicto y el criterio escrito ANTES de mirar** | el documento de plan que enlaza cada reporte, en `foveal-vision/docs/` — [`plan-prioridades-2026-08-25.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-prioridades-2026-08-25.md) (prioridades 1 y 2), [`plan-tres-ejes.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-tres-ejes.md) (`batch_size`, `n_layers`, `border_px`), [`plan-lr-alto.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-lr-alto.md) (`lr`), [`plan-cnn-plana.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-cnn-plana.md) y [`plan-plana.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-plana.md) (la plana), [`metrica-de-tarea.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/metrica-de-tarea.md) y [`decisiones.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/decisiones.md) **F15** (los knobs) |
| **el número crudo, run a run** | `foveal-vision/sweeps/<recorrido>/informe.json` (grupos y contrastes) y `flota.json` (coste, reloj, máquinas); el libro de a bordo en `runs/`. ⚠ **Mira desde qué commit lo lees**: `informe.json` describe los runs que ese clon tenía a la vista, y un `push` roto hizo que `3636ccfa` marcara `interrupted` cuatro runs que sí habían terminado (corregido en `foveal-vision@199f10d4`; ver la corrección del [#13](reportes/estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md)). Un `n_seeds` bajo puede ser un recorrido incompleto **o** un clon desactualizado |
| **qué significa cada parámetro, en cristiano, y por qué se estudia en ese orden** | [`foveal-vision/reportes/2026/08-agosto/parametros-y-prioridad-de-estudios.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/reportes/2026/08-agosto/parametros-y-prioridad-de-estudios.md) — el inventario completo de los cuatro dominios (C/D/F/X) con la explicación de cada mando |
| **cómo se lee la geometría nueva** (`border_px`, `border_reduce`, `overlap_*`) y su traducción desde `N`/`c_frac`/`d` | [`foveal-vision/instructionsNewNN.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/instructionsNewNN.md) §2.1 |
| **los primeros resultados bajo la geometría nueva** | [`foveal-vision/reportes/2026/08-agosto/2026-08-26-geometria-nueva-primeros-resultados.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/reportes/2026/08-agosto/2026-08-26-geometria-nueva-primeros-resultados.md) |
| **cómo encajan los cinco repos entre sí**, dónde están los acoplamientos y qué convendría cambiar | [reportes/arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md](reportes/arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md) — análisis de diseño del sistema completo. ⚠ **No es un barrido**: no tiene fila en la tabla cronológica de abajo porque no alquiló nada ni costó nada, y una fila de «no aplica» ensuciaría la contabilidad |

⚠ **El inventario de `parametros-y-prioridad-de-estudios.md` es del 2026-08-25 y esta tabla es
posterior.** Los siete ejes de prioridad 2 que allí figuran como «NUNCA barrido» —`pos_weight`,
`scheduler`, `monitor`, `k_center`, `channels`, `border_reduce`, `overlap_fovea_px`— **se midieron el
26-ago** (reportes #11 y #13), y `border_px` ya **no** está «abierto por la derecha» (#9 lo cerró).
Para el estado de un eje manda esta tabla; para *qué es* cada parámetro, aquel documento.

