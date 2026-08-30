# Reportes — el índice

Aquí queda **un reporte por cada barrido, estudio o medición que se termine, venga de donde venga**.
El repo desde el que se lanzó el trabajo, el repo que produjo el dato y el repo donde nació la
petición pueden ser tres distintos —y a menudo lo son—; el reporte va **siempre aquí**.

⚠ **Esto NO sustituye a la fuente de verdad, la resume y la enlaza.** El dato vive donde lo dejó
quien lo produjo —`sweeps/*/flota.json` e `informe.json` en `foveal-vision-data`, `results/*/` en el
lanzador— y el **veredicto** de un estudio vive en el documento de plan que escribió su criterio
*antes* de mirar, en `foveal-vision/docs/plan-*.md`. Copiar aquí el análisis es como nacen las dos
mitades desfasadas. Un reporte de este directorio contesta *«qué se corrió, cuándo, con cuántas
máquinas, qué costó y qué salió»*, y para lo demás apunta.

Y **el estado no vive aquí**: en qué quedó cada parámetro está en [`../ESTADO.md`](../ESTADO.md),
que **se reescribe**. Esta tabla de abajo es historial y **sólo se añade**. Están separados a
propósito.

## Dónde va cada reporte

```
reportes/<tipo>/<año>/<mes>/<fecha>-<nombre-del-estudio>.md
         └─ p. ej. reportes/estudios/2026/08-agosto/2026-08-25-bs-alto-tanteo.md
```

El mes lleva **número delante** (`08-agosto`, no `Agosto`) por una razón práctica: así `ls` y el
árbol de GitHub salen en orden cronológico en vez de alfabético, que con nombres de mes deja
`Agosto` antes que `Julio`. El nombre del fichero lleva la fecha por lo mismo, dentro del mes.

⚠ **Y por eso la fecha en el nombre no es cosmética: con `<tipo>/` en medio, el orden cronológico
del conjunto ya no sale de recorrer un directorio.** Se recupera así, y sólo funciona si todos los
ficheros empiezan por su fecha:

```sh
find reportes -name '*.md' ! -name README.md -printf '%f\n' | sort
```

### Los cuatro tipos, y cómo se elige uno

El discriminante es **mecánico**, y se contesta leyendo la cabecera del propio reporte. Dos
preguntas:

1. **¿Se corrió algo?** (¿hay reloj y factura?) → sí: `estudios` o `infraestructura`.
   No: `sintesis` o `arquitectura`.
2. **¿Cuál es el sujeto medido?**

| Tipo | El sujeto es… | Contesta a | Hoy |
|---|---|---|---:|
| **`estudios`** | **la red** — un eje del espacio de búsqueda, f1, una semilla | *¿qué le pasa a la red al mover este mando?* | 14 |
| **`infraestructura`** | **la máquina o la cadena** — s/época, $/unidad, una prueba de humo | *¿con qué se mide, cuánto cuesta y funciona el camino?* | 4 |
| **`sintesis`** | **conocimiento ya pagado**, releído y reordenado. No mide nada nuevo | *¿qué sabemos ya, junto y en orden?* | 2 |
| **`arquitectura`** | **el sistema** — repos, interfaces, procesos | *¿cómo encajan las piezas y qué habría que cambiar?* | 2 |

Dos casos de borde ya resueltos, para no volver a discutirlos:

- **`#12` (knobs de inferencia) es `estudios` aunque alquiló 0 máquinas y costó 0,00 $.** Corrió
  sobre `best.pt` que ya estaban en disco, pero **el sujeto medido es la red**. La primera pregunta
  ordena; la segunda decide.
- **`#1` y `#2` son `infraestructura` aunque los lanzó un script de `foveal-vision`.** Miden
  `seconds_per_epoch` por vCPU y $/unidad: el sujeto es la máquina. **Quién lanza no clasifica.**

### Qué NO va aquí

**Ni una línea de código, ni un byte de dato crudo.** Al repo central va el `.md`; los adjuntos de
datos se quedan con su productor y se enlazan por URL. El caso vivo:
`reportes/2026/08-agosto/datos/knobs-f-20260825.json` sigue en `foveal-vision`, y el `#12` lo enlaza.
Es lo que impide que esto se convierta en el sitio donde va lo que no se sabe dónde poner.

## Qué lleva un reporte, siempre

Una tabla de cabecera con estos campos, y ninguno se omite —si un dato no existe **se escribe que
no existe**, que es información y no un hueco—:

| campo | por qué |
|---|---|
| **Qué era** y **lanzado con** | para poder repetirlo |
| **Inicio** y **Fin** (UTC) | y si están *derivados* en vez de leídos, se dice |
| **Duración** | el reloj, que es lo que se nota |
| **Instancias** | las **alquiladas**, no las que trabajaron: son las que facturan |
| **Coste real** | el de la factura. Si sólo se conoce un suelo, se marca como suelo |
| **Dataset** | dos medidas sólo se comparan si coinciden en el dato |
| **Estado** | terminado / incompleto, con el recuento de lotes |

Y después: qué se midió, los hallazgos, **lo que quedó pendiente**, y de dónde salen los números.

Además, desde la centralización del 2026-08-29, **la segunda línea de cada reporte lleva su `#` y su
tipo**. No es adorno: el `#` se cita desde otros repos (`foveal-vision/docs/plan-dropout-2026-08-28.md`
enlaza al `#14`) y hasta ahora **sólo existía en la tabla de abajo** — un dato que no se puede
re-derivar de ningún artefacto. Los que no son barridos lo dicen: `sin # — no es un barrido`.

⚠ **El `#` no se reasigna nunca**, ni al reclasificar ni al reordenar. Un número reapuntado sigue
resolviendo a *un* reporte, sólo que al equivocado, y eso es peor que uno que falta.

## Los reportes, en orden cronológico

Se **añade al final**; las filas anteriores no se tocan.

| # | Reporte | Ubicación | Inicio (UTC) | Fin (UTC) | Instancias | Coste real | Hallazgo en una línea |
|---:|---|---|---|---|---:|---:|---|
| 1 | `bench-vcpu` — s/época por vCPU en droplets de DO | [infraestructura/2026/08-agosto/2026-08-20-bench-vcpu-droplets-do.md](infraestructura/2026/08-agosto/2026-08-20-bench-vcpu-droplets-do.md) | 2026-08-20 12:50:52 ⚠ sin zona | ⚠ no registrado | 2 | ⚠ **no registrado** | Doblar las vCPU dio 1,83×, no 2×. La corrida no guardó ni coste ni horas |
| 2 | `foveal-cpu` — barrido de vCPU en Vast.ai | [infraestructura/2026/08-agosto/2026-08-20-barrido-vast-foveal-cpu.md](infraestructura/2026/08-agosto/2026-08-20-barrido-vast-foveal-cpu.md) | 2026-08-20 21:18:28 | 2026-08-21 02:22:57 | ≥5 | **≥0,0293 $** | El precio por hora no ordena el rendimiento: 18 vCPU y 2 vCPU al mismo $/h, 4× de diferencia en $/unidad |
| 3 | `lr-alto-L4` — cerrar `lr` por la derecha | [estudios/2026/08-agosto/2026-08-23-lr-alto-L4.md](estudios/2026/08-agosto/2026-08-23-lr-alto-L4.md) | 2026-08-23 18:18:10 *(derivado)* | 2026-08-23 20:37:16 | 3 | **0,2952 $** | `lr` queda acotado por los dos lados; bandas disjuntas y `p = 0,100` es el suelo, no un empate |
| 4 | `lr-alto-L4-b` — el mismo, una máquina por run | [estudios/2026/08-agosto/2026-08-23-lr-alto-L4-b.md](estudios/2026/08-agosto/2026-08-23-lr-alto-L4-b.md) | 2026-08-23 22:40:55 *(derivado)* | 2026-08-23 23:36:19 | 9 | **0,3656 $** | 2,5× menos reloj por 7 céntimos. El coste del paralelismo fino no es el peaje: es agotar las ofertas baratas |
| 5 | `bs5-L4`+`nl5-L4`+`d5-L4` — tres ejes, pasada 1 | [estudios/2026/08-agosto/2026-08-24-tres-ejes-pasada1.md](estudios/2026/08-agosto/2026-08-24-tres-ejes-pasada1.md) | 2026-08-24 17:19:19 | 2026-08-24 23:28:00 | 22 (+43 en 2 abortos) | **2,6471 $** (+0,14 abortos) | `batch_size` es plano entre 57 y 192 —no lo que decían los 3 estudios viejos—; `n_layers=4` confirmado. Reproducibilidad bit a bit con 5 pares |
| 6 | `d5-L4` — pasada 2 (rehacer el eje `d`) | [estudios/2026/08-agosto/2026-08-24-d5-L4-pasada2.md](estudios/2026/08-agosto/2026-08-24-d5-L4-pasada2.md) | 2026-08-24 23:52:10 | 2026-08-25 02:22:37 | 11 | **0,7071 $** | `d` sube monótono y gana el extremo: sin acotar por arriba. Un estudio inválido tampoco vale para decidir dónde mirar. ⚠ Releído con la geometría de 2026-08-25: ese eje era **`border_px` a coste constante** (2→8 px) |
| 7 | `pl-t-lr`+`pl-t-bs` — tanteo de la red plana | [estudios/2026/08-agosto/2026-08-25-plana-tanteo.md](estudios/2026/08-agosto/2026-08-25-plana-tanteo.md) | 2026-08-25 03:01:05 | 2026-08-25 04:35:58 | 25 | **1,026 $** | ⚠ Incompleto (13/20). `lr` útil en 0,00035–0,0014; en 0,0028 **una semilla colapsó a f1 = 0** |
| 8 | `bs-alto-fov`+`bs-alto-pl` — `batch_size` por arriba | [estudios/2026/08-agosto/2026-08-25-bs-alto-tanteo.md](estudios/2026/08-agosto/2026-08-25-bs-alto-tanteo.md) | 2026-08-25 10:35:05 | 2026-08-25 13:08:32 | 24 | **1,6846 $** | En las dos redes el f1 **baja** monótono al subir el batch: la zona plana termina en el ancla, no hay nada arriba |
| 9 | `borde-ancho` — ¿más contexto a coste constante? | [estudios/2026/08-agosto/2026-08-26-borde-ancho.md](estudios/2026/08-agosto/2026-08-26-borde-ancho.md) | 2026-08-26 01:42:54 | 2026-08-26 04:40:15 | 18 *(flota compartida con #10)* | **1,0536 $** *(compartido con #10)* | **El eje queda CERRADO por los dos lados**: sube hasta 8 px y baja de ahí en adelante. El vigente se queda (p = 0,063, la misma que en `d5-L4`) |
| 10 | `pl-t-bs`+`pl-t-nl` — afinado de la plana (fase 1) | [estudios/2026/08-agosto/2026-08-26-plana-tanteo-fase1.md](estudios/2026/08-agosto/2026-08-26-plana-tanteo-fase1.md) | 2026-08-26 01:42:54 | 2026-08-26 08:32:39 | 18 *(compartida con #9)* + 4 | **compartido con #9** + **0,0571 $** | Los dos tanteos acotados: `batch_size` 170 y `n_layers` 5, los dos interiores. ⚠ En L6 **una semilla dio f1 = 0,0000** |
| 11 | Prioridad 2 — siete ejes nunca medidos | [estudios/2026/08-agosto/2026-08-26-prioridad2.md](estudios/2026/08-agosto/2026-08-26-prioridad2.md) | 2026-08-26 01:57:11 | 2026-08-26 06:53:57 | **101** *(para 35 lotes)* | **3,2996 $** | ⚠ Incompleto (3 de 7; los otros 4 se cierran en el **#13**). `border_reduce`=1 gana con p = 0,008 **pero no es cost-neutral**; `k_center` y `monitor` dejan el vigente donde estaba |
| 12 | Knobs de inferencia (F) re-medidos | [estudios/2026/08-agosto/2026-08-26-knobs-f.md](estudios/2026/08-agosto/2026-08-26-knobs-f.md) | 2026-08-26 ~01:30 | 2026-08-26 03:23 | **0** *(local)* | **0,00 $** | ⚠ 2 de 3 filas válidas. Los defaults dejan +0,053 a +0,071, y **el óptimo ya no es el mismo para todos los modelos** — en julio sí lo era |
| 13 | Prioridad 2 — relanzamiento de los cuatro a medias | [estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md](estudios/2026/08-agosto/2026-08-26-prioridad2-relanzamiento.md) | 2026-08-26 11:04:49 | 2026-08-26 15:58:19 | **34** *(18 lotes)* | **2,0283 $** | **Los cuatro cerrados, 20/20.** El vigente gana en los cuatro ejes; en `ov-fov` el punto **0** (ramas disjuntas) es **peor con p = 0,032**: el solape aporta. ⚠ `overlap` no queda acotado por arriba |
| 14 | Cierre de parámetros — `overlap_fovea_px`, `border_px` y los 9 ejes nunca medidos | [estudios/2026/08-agosto/2026-08-26-cierre-parametros.md](estudios/2026/08-agosto/2026-08-26-cierre-parametros.md) | 2026-08-26 22:13:08 | 2026-08-27 18:05 *(parado a mano)* | **≥160** *(4 flotas)* | **≥9,9367 $** ⚠ *el último tramo no quedó registrado* | ⚠ **El dataset de los estudios ya no se puede reconstruir** (`r20260826`, otro dato). Con el eje entero sobre un solo dataset: **`overlap_fovea_px` 2 → 7** (`p` < 0,001) y **`border_px` 4 → 8** (`p` = 0,006, y más rápido). Los 9 ejes nunca medidos: **ninguno mueve el vigente** |
| 15 | `stride` — infraestructura del barrido de stride de extracción y validación en Vast | [infraestructura/2026/08-agosto/2026-08-27-stride-infraestructura.md](infraestructura/2026/08-agosto/2026-08-27-stride-infraestructura.md) | 2026-08-27 02:29:43 | 2026-08-27 10:40:32 | **9** *(2 entrenaron)* | **0,0383 $** | **La infraestructura probada en máquina; el estudio SIN correr.** Hecho de la manera natural, un barrido de stride mide el examen y el presupuesto y no la densidad: hay **146,2×** más ventanas de train a stride 1 que a 16 |
| 16 | `stride` — el barrido de la densidad de la rejilla (1→16 px) | [estudios/2026/08-agosto/2026-08-27-stride-estudio.md](estudios/2026/08-agosto/2026-08-27-stride-estudio.md) | 2026-08-27 12:55:06 | 2026-08-27 17:11:55 | **97** *(2 flotas; 25 runs útiles)* | **1,6801 $** | **La rejilla más densa predice mejor y NO satura**: monótono de 0,8901 (stride 16) a 0,9383 (stride 1), `p` = 0,0079. Gana el extremo, así que el eje no queda cerrado por arriba. ⚠ R4 falló como estaba escrito —medía el reloj— y hubo que arreglarlo antes de declarar |
| 17 | `do-t` — tanteo de `dropout`: ¿regulariza, y ayuda? | [estudios/2026/08-agosto/2026-08-28-dropout-tanteo.md](estudios/2026/08-agosto/2026-08-28-dropout-tanteo.md) | 2026-08-28 01:30:11 | 2026-08-28 04:51:22 | **5** *(2 entrenaron)* | **0,3626 $** | `dropout` **cierra entera** la brecha val/train de +28 % —confirmada aquí en +29,5 %— **y el f1 baja igual**: el sobreajuste no era el cuello de botella. ⚠ El estudio de 5 semillas (`do-v`) queda **creado y sin lanzar** |

**Gastado y registrado hasta aquí: ≥25,35 $** (13,33 hasta el #13, más ≥9,94 del #14, más 0,04 de
la validación del #15, 1,68 del barrido de stride del #16 y 0,36 del tanteo de dropout del #17).

⚠ **Y el #14 vuelve a dejar el hueco que este directorio existe para evitar**: su última flota se
**paró a mano**, así que `estudio_flota.py` no llegó a escribir su `flota.json` y de ese tramo sólo
hay el **suelo** que el log alcanzó a anotar (2,1404 $ en 21 instancias). La lección de siempre, y
van dos: **cuando haya que parar una flota a mano, el número de la factura sólo queda en el panel
del proveedor.**

**Gastado y registrado hasta el #13: 13,33 $** (6,89 hasta el #8, más 6,44 en los estudios de
prioridad del 26-ago). No incluye lo que no quedó anotado —la flota de droplets de DO (#1), los
alquileres de Vast que fallaron antes de medir (#2) y **la corrida del 25-ago 22:20 que se mató a
mitad** (#9), cuyo `flota.json` no llegó a escribirse—, que es justamente lo que este directorio
existe para que no vuelva a pasar.

⚠ **Cuando falta el dato, falta por una razón: la corrida no llegó a su final.** El coste, el reloj
y las instancias los escribe `estudio_flota.py` **al terminar**; si la flota se mata o se apaga desde
fuera, las máquinas mueren pero **la contabilidad no se escribe**. El libro de a bordo salva los
*resultados* minuto a minuto, no el *cierre*. Cuando haya que apagar una flota a mano, el número de
la factura sólo queda en el panel del proveedor: **anótalo antes de perderlo.**

⚠ **Y el recíproco, que costó dos correcciones en el #13: que falte el dato AHORA no quiere decir
que vaya a faltar siempre.** Ese reporte declaró su coste irrecuperable mientras la flota seguía
viva; cerró sola 2 h 40 más tarde y escribió los 2,0283 $ que ahora están en la tabla. **Un
`flota.json` ausente sólo prueba que la flota no ha terminado —no que haya muerto.** Antes de dar
una corrida por perdida, comprobar que de verdad no queda nada corriendo.

## Los que no son barridos, y por eso no tienen fila arriba

La tabla de arriba es **contabilidad**: lleva instancias y coste real, y una fila de «no aplica» la
ensuciaría. Estos cinco no alquilaron nada y no costaron nada, así que van aquí — pero se listan,
porque un reporte que no está en ningún índice no existe.

| Reporte | Tipo | Qué es |
|---|---|---|
| [2026-08-25-parametros-y-prioridad-de-estudios.md](sintesis/2026/08-agosto/2026-08-25-parametros-y-prioridad-de-estudios.md) | `sintesis` | El inventario completo de los cuatro dominios (C/D/F/X): qué es cada mando, qué se ha barrido ya y en qué orden conviene seguir. ⚠ Es del **2026-08-25** y `ESTADO.md` es posterior: para el estado de un eje manda `ESTADO.md`; para *qué es* un parámetro, manda éste |
| [2026-08-26-geometria-nueva-primeros-resultados.md](sintesis/2026/08-agosto/2026-08-26-geometria-nueva-primeros-resultados.md) | `sintesis` | Los ocho recorridos del 25–26 de agosto leídos juntos bajo la geometría nueva. No midió nada: relee lo ya pagado |
| [2026-08-27-stride-validacion.md](infraestructura/2026/08-agosto/2026-08-27-stride-validacion.md) | `infraestructura` | ⚠ **El mismo evento que el `#15`**, escrito desde el otro repo. Se conserva por el detalle del arnés; el canónico es el `#15` |
| [2026-08-28-analisis-arquitectura.md](arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md) | `arquitectura` | Los cinco repos leídos como un solo sistema. De aquí salen las 19 reglas de diseño |
| [2026-08-30-sobrevivir-a-rehacer-el-dev.md](arquitectura/2026/08-agosto/2026-08-30-sobrevivir-a-rehacer-el-dev.md) | `arquitectura` | Por qué las dos aplicaciones del 29–30 de agosto —la web app como servicio y el entrenador en Vast— **no funcionaron a la primera**. Inventariado sobre un dev recién nacido: el código llegó entero y el estado casi nada (0 fuentes, 0 pesos, 2 de 18 datasets). ⚠ El modelo `fov-optimo-p20` —69 épocas en Vast— **se perdió** |
