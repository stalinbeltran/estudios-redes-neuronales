# `rect-lin` — el detector de rectas como un kernel LINEAL (célula simple de V1)

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-08 13:44 (criterio commiteado) → 17:52 (análisis); rejilla en Vast 14:25 → 14:57 (del libro) |
| **Instancias alquiladas** | **1** (Xeon E5-2682 v4, 32 vCPU, 62,6 GB, Noruega, 0,1298 $/h), 14 procesos × 2 hilos, rc 0, destruida. La misma rejilla corrió en el dev hasta 381/432 y se paró a mano |
| **Coste real** | **0,0688 $** (31,8 min, del libro `resultados/vast/rejilla/`); el dev, 0 $ |
| **Runs** | 432 (kernel 5/7/9 × 1/2/3 escalas × entrenamiento continuo/punteado × N 4–1000 × 3 semillas); referencias: 9 Gabor a mano y la CNN de `feat-ind32` |
| **Dataset / red** | `rect-lin-entreno-r20261008` y `rect-lin-banco-r20261008` (publicados aquí). 2 kernels k×k aprendidos + rot90 = 4 orientaciones, pirámide de 1–3 escalas, max: 52–164 parámetros |
| **Artefactos** | [`experimentos-cnn/2026-10-08-rectas-kernel-lineal/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-08-rectas-kernel-lineal) (README con tablas y 4 figuras; criterio en `instrucciones/02-criterio.md`, commit `b8590bb`, antes de medir) |

**Resultado:** para rectas **finas**, un kernel lineal de 9×9 (164 parámetros) da recall **0,991** en cualquier ángulo con
**0,07 %** de falsos positivos, y llega a 0,95 con **64** ejemplos. La CNN de `feat-ind32` (167.300 parámetros) da **0,674**,
y cae a 0,30 con rectas a 15–22,5° de sus cuatro direcciones. Pero aprender aporta poco sobre un **Gabor a mano** (0,96–0,98
sin entrenar; +0,033, por debajo de las 0,05 del criterio). Un kernel aprendido con varias escalas **empeora** en lo fino
(0,94) y no gana grosor; el Gabor con 2 escalas sí (0,88 en 6–8 px). Las **punteadas no se generalizan**: entrenado con
continuas sólo ve puntos casi pegados; entrenado con punteadas no ve continuas y se enciende con puntos sueltos (FP
0,47–0,61). Una **curva** de 20 px se detecta como recta el 96–100 % de las veces con trazo fino, y sólo responde más
débil que una recta si R ≤ 9. Contra el criterio: **2 de 9** (H1 ✅, H9 a medias: fino ✅, grueso ❌).

**Velocidad (pregunta del dueño):** alquilar acelera unas **8×** en reloj (24,6 min contra ~4 h 20 min estimadas en el
dev) por 0,07 $. Es por paralelismo: cada proceso va 1,21× más lento que en el dev. Los 381 brazos hechos en las dos
máquinas dan métricas y kernels **idénticos bit a bit**.

**Lo que quedó pendiente:** (1) usar el **mapa** de detecciones (dónde y con qué orientación), no su max, como base de la
segunda etapa: curvas como cadenas de rectas cortas y agrupación de puntos separados; (2) el grosor vía filtro de bordes
(el dueño); (3) el Gabor de 2 escalas como detector de serie, ajustando su umbral; (4) por qué el kernel aprendido no se
parece a una línea (sin comprobar); (5) un N equilibrado por orientación para la parte baja de la curva de aprendizaje.
