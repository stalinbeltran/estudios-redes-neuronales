# `feat-1lado` — detectores de features sobre 8 bordes de UN SOLO LADO, y un compositor entrenado con la entrada desplazada

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-07 16:53 → 19:31 (detectores en Vast, de los libros); compositor y evaluación en el dev hasta las 21:52 |
| **Instancias alquiladas** | **2**, una por brazo, las dos Xeon E5-2660 v4, 28 vCPU, 31 GB, Vietnam, 0,0948 $/h; 13 procesos × 2 hilos; rc 0, destruidas |
| **Coste real** | **0,1973 $** — control 16,3 min / 0,0257 $; compartido 108,6 min / 0,1716 $ (de los libros `resultados/vast/`) |
| **Runs** | 2 brazos (control · compartido) × 13 detectores × 1 semilla; compositores × 3 semillas, 15 regímenes de desplazamiento por banco |
| **Dataset / red** | entrenar: `feat-ind32-sinteticas-32px-r20261005` (2–4 px); grosor: `feat-bor-sinteticas-grueso-32px-r20261006`; dígitos: `uci-optdigits-orig-32px-r20261005`. La red de `feat-ind32` detrás de una capa FIJA de 8 kernels Sobel orientados + ReLU (42.833 / 41.825 parámetros) |
| **Artefactos** | [`experimentos-cnn/2026-10-07-features-un-lado/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-07-features-un-lado) (README con todas las tablas) y el boceto previo, [`docs/bocetos/2026-10-07-borde-de-un-lado/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/docs/bocetos/2026-10-07-borde-de-un-lado) |

**Resultado: mirar un borde de un solo lado cada vez NO arregla el grosor, y cuesta acierto en los dígitos.** Los dos brazos
aprenden (F1 medio 0,924 control, 0,872 compartido, contra 0,916 de las líneas), pero con trazos de 6–12 px el compartido es el
que más recall pierde de todos los medidos (0,81 → 0,68), y su F1 grueso (0,560) no mejora el de las líneas (0,567). El control
—los 8 canales juntos— da el mejor F1 grueso medido (0,627). Los dos apagan los arcos en los 1 (compartido 16 % / 4 %). En los
dígitos el compositor cae de **0,949** (líneas) a **0,889** (control) y **0,796** (compartido). H1 ✅✅ · H2 ❌❌ · H3 ❌✅ ·
H5 ❌❌ · H6 ❌❌ · H7 ❌❌.

**La curva de desplazamiento** (pedida por el dueño): un compositor entrenado con desplazamientos 0–*s* px aguanta hasta ~*s*
px sin perder en el centro (líneas, 0–4 px: 0,960 en d = 0 y 0,839 en d = 4, contra 0,633 sin desplazar); entrenado SÓLO con
*s* px aprende esa posición y falla en el centro. El compositor sin desplazar cae al azar a 8 px con el dígito casi entero en el
lienzo: la caída es de posición no vista, mucho antes que de información perdida (en el boceto, «sólo 16 px» aún acierta 0,77).

**Lo que quedó pendiente:** (1) por qué el compartido pierde recall con el grosor — reparte entre dos canales OPUESTOS, o sea los
dos lados del trazo; la explicación del ancla en la línea media no está comprobada; (2) elegir el compositor de 0–4 px en vez de
0–2 (relectura, 0 $); (3) una semilla, e inicializaciones distintas entre brazos. **Coste contra estimación:** se estimó primero
~0,1 $ / 1 h, luego 0,6–2,1 $ / 3–6 h (revisor); el control midió 8,1 s/época (más rápido que el dev) y con eso el compartido pasó
a 13 × 2 hilos en 28 vCPU: tardó 108 min, no los ~60 recalculados.
