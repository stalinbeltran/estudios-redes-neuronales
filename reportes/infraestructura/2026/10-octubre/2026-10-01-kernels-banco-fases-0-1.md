# Plan de kernels del banco, fases 0 y 1 — el modo `trabajo` en Vast, su primera corrida y el dataset a /4

| | |
|---|---|
| **Inicio (UTC)** | 2026-10-01 22:34:42 (primer alquiler de la fase 1). El trabajo del dev empezó antes: el primer render del dataset, a las 21:45:26 |
| **Fin (UTC)** | 2026-10-01 22:46:10 (la última máquina destruida). El segundo render del dataset terminó a las 22:54 |
| **Instancias alquiladas** | **3** en Vast.ai *(1 trabajó; 2 murieron antes de trabajar, las dos en el **mismo** host 152600, Core i7-4790, que nunca aceptó la clave SSH)*. La que trabajó: Xeon E5-2660 v4, 4 vCPU, 0,0489 $/h |
| **Coste real** | **0,0063 $** *(del libro: 0,0007 + 0,0031 los dos intentos fallidos + 0,0025 la que trabajó)*. El dev: 0 $ (dos renders, 41 y 24 min) |
| **Runs** | **1** en Vast: `banco-k`, `identidad` semilla 3, ya conocido (el del dev del 2026-09-08). Más una réplica en el dev, el mismo día, como referencia |
| **Artefactos** | el libro de la fase 1 y lo traído: [`experimentos-cnn/…/banco-kernels/resultados/vast/fase1/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-09-08-banco-kernels/resultados/vast/fase1) · el modo `trabajo`: [`vast_instance.py`](https://github.com/stalinbeltran/digital-ocean-dropplet-auto-launching/blob/main/scripts/vast_instance.py) (`b43e782`, `0a55a98`) · el dataset: `foveal-vision-data/experimentos-cnn/parrafos1000-pagina1024-r4-r20261001/` (almacén, `4e0f1535`) |
| **Plan y criterio** | [`experimentos-cnn/docs/plan-kernels-banco-2026-10-01.md`](https://github.com/stalinbeltran/experimentos-cnn/blob/main/docs/plan-kernels-banco-2026-10-01.md); la regla de deriva de la fase 1 estaba escrita en su §2 **antes** de alquilar |

> Este reporte **resume y enlaza**. Es de la **cadena** —el lanzador, el peaje, la deriva entre
> máquinas— y del **dato**; ninguna red se ha entrenado todavía. El estudio de kernels (fases 2–4)
> sigue abierto y tendrá su propio reporte.

---

## Qué se preguntaba

El dueño pidió (2026-10-01) ejecutar el plan para generar kernels de **todos** los tamaños que
espera el banco `banco-k` (k = 3…19), en máquinas alquiladas en paralelo. Decisiones suyas: escala
del banco (/4), **un** kernel para los cuatro bordes, **3 semillas**, los **tres** productores
(`bor-k`, `bor-ae`, `bor-pca`), sin importar los kernels viejos con fuga. Y sus sugerencias
aceptadas: el `λ` de `bor-ae` se calibra antes en el dev; las máquinas viven como mucho 3 h.

Antes de alquilar nueve máquinas, el plan alquila **una** y le hace correr algo cuyo resultado ya
está en git, para contestar tres cosas que nadie sabía de este trabajo: si la cadena entera
funciona, cuánto cuesta el arranque y cuánto se desvía el número de una máquina a otra.

## Qué salió

### 1. La cadena funciona — al tercer intento, y los dos primeros enseñaron algo

| | |
|---|---|
| del alquiler a la máquina lista | **1 min 48 s** (sello al primer intento, payload con sha256 comprobado en destino, instalación de torch 2.14.1 en 51 s). El plan suponía **8,4 min** (`estudio_estimar.py`, otro trabajo, agosto). **Una sola muestra** |
| el run | **36,0 s** en el E5-2660 v4, contra 41,5 s del dev el 2026-09-08 |
| vida total de la máquina | 3,1 min, 0,0025 $ |

**Los dos primeros intentos murieron con `Permission denied (publickey)`**, y la red de seguridad
funcionó las dos veces: el libro apuntó el fallo y la máquina se destruyó en 1 y 4,5 min. Eran
**dos lecciones que `foveal-vision/scripts/estudio_flota.py` ya había pagado** (2026-08-24) y que el
modo nuevo no traía: el banner de sshd llega antes que la clave, y la API puede publicar el puerto
de **otra** instancia. Se portaron (`0a55a98`: destino propio y estable, y un sello que se escribe
y se relee). Con eso el segundo intento reintentó 12 × 20 s… **en el mismo host**, que nunca aceptó
la clave: la oferta más barata era siempre esa. Se **bloqueó** el host 152600 (`16abc0d`) y el
tercero, en otro, entró al primer intento — o sea que la clave de flota sirve en Vast y el problema
era el host. ⚠ El `#24` (`dim-gen`, la misma madrugada) vio lo mismo: «clave SSH rechazada ×1».

### 2. La deriva Vast ↔ dev pasa del umbral escrito antes

| época | IoU eval dev | IoU eval Vast | Δ |
|---:|---:|---:|---:|
| 25 | 0,758441 | 0,758418 | −0,000023 |
| 50 | 0,787213 | 0,787735 | +0,000523 |
| 100 | 0,795962 | 0,793338 | −0,002623 |
| **200** | **0,809848** | **0,808662** | **−0,001186** |

**|Δ IoU eval| final = 0,0012 > 0,001.** La regla del plan, escrita antes de medir, manda entonces
que en la fase 3 **cada máquina corra también `identidad` y el aleatorio de su `k`**, aunque ya
existan en el dev: todo lo que se compara sale de la misma máquina.

**La deriva es de la máquina, no de la versión**: el mismo día, este dev reprodujo **bit a bit** el
`identidad-s3` del 2026-09-08 (diferencia máxima 0,00e+00 en todas las columnas) con el torch que
se instaló en Vast. El dev es «DO-Regular»; la de Vast, un E5-2660 v4.

⚠ **Fijar la familia de CPU no sale gratis aquí**: medido el 2026-10-01, sólo hay **9** máquinas
E5-26 distintas con 4–8 vCPU, ≥ 8 GB y ≤ 0,12 $/h (la novena a 0,119 $/h). No se fija: los
controles por máquina ya hacen comparable cada `k` consigo mismo, y el banco no compara entre `k`.

### 3. El dataset a /4 salió sesgado la primera vez, y no se publicó

`bor-p4` rinde las páginas de `bor-p` con la geometría ×4 y las guarda reducidas /4 (la escala a la
que el banco aplica el kernel). **El primer render dio 977 párrafos en vez de 1000** (6 páginas
perdidas enteras, 91 descartes) y el cuarto de cuerpo más grande **un 11 % por debajo** del
uniforme (`[270, 251, 237, 217]` por cuartos de [11, 30], ~244 cada uno): con la geometría ×4 hay
columnas de 120 px donde un cuerpo de 30 no cabe, y el descarte hacía de **sortear-y-rechazar**
sobre el cuerpo. Se emparejaron los cuerpos con las ranuras (el mayor a la más ancha: ninguna
marginal cambia) y se subieron los intentos por página de 3 a 10.

**El segundo render**: 331 páginas, **1000 párrafos**, 40 descartes, 0 páginas perdidas, cuerpo
`[253, 255, 239, 253]` (χ² = 0,66 con 3 g.l.), tinta fuera de las cajas **0**, ventana limpia
mínima 79 px. Publicado como `parrafos1000-pagina1024-r4-r20261001`; el primero quedó en el almacén
como evidencia (`temporal/dev/2026-10-01/bor-p4-primer-render/`).

### 4. Lo construido (fase 0), sin gastar

- **El modo `trabajo`** de `vast_instance.py`: N trabajos en N máquinas, una unidad de systemd por
  trabajo, ofertas repartidas antes, libro en disco paso a paso, lo traído nunca pisa lo local,
  destrucción en `finally`. 11 tests. **Su freno**, en el mismo commit que el acelerador: el
  `cerrable.mjs` del coordinador lo ve (`ef3ccca`, 3 tests que fallan con el freno anterior), y
  desde Telegram `/use exp-vast` → `estado` · `apagar <prefijo>`. ✅ **Ya lo usó otra sesión**: el
  `#24` corrió sus 30 brazos con él.
- `banco-k/nn/importar_kernel.py` lee la reserva §3.7 del **manifiesto** del dataset cuando el
  experimento de origen no tiene receta (antes asumía fuga siempre). 8 casos.
- `bor-k`, `bor-ae`, `bor-pca`: código comprobado, reglas y **criterio escrito antes de medir**.

## Lo que quedó pendiente

- **Antes de la fase 2, todo en el dev y a 0 $**: los suelos y el `LAMBDA_COORD` de `bor-k`
  (`--suelos`), el ensayo de mecanismo de su `lr`, **el tanteo de `λ` de `bor-ae`** —con la regla
  ya escrita: si ningún `λ` aparta el filtro de la identidad, `bor-ae` no va al banco— y las
  componentes de `bor-pca`. **Nada de esto se ha corrido.**
- **Fase 2** (entrenar): 9 máquinas **por experimento** (`bor-k`, y `bor-ae` si el tanteo da un
  `λ`), no 9 en total: juntar dos experimentos en una máquina haría que uno ejecutase el código del
  otro (Regla 0). ≈ +0,07 $ de arranques *(estimado)*. **Espera el visto bueno del dueño**: era el
  punto de parada acordado tras la fase 1.
- **Fase 3** (evaluar en el banco): con `identidad` y el aleatorio de su `k` **en cada máquina**,
  por la regla de deriva de arriba.
- **Fase 4**: informe del banco y, si algún kernel declara, este estudio sube a `ESTADO.md`.
- ⚠ El peaje de 1 min 48 s es **una** muestra; los costes del plan se rehacen con él cuando haya
  más (la fase 2 dará nueve).
