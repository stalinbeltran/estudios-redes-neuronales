# `rect-bor` — el grosor de las rectas con un filtro de bordes antes del kernel lineal

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-08 18:22 (criterio commiteado) → 18:55 (análisis); rejilla en Vast 18:31 → 18:40 (del libro) |
| **Instancias alquiladas** | **1** (AMD EPYC 7702, 32 vCPU, 31,5 GB, Noruega, 0,110 $/h), 12 procesos × 2 hilos, rc 0, destruida |
| **Coste real** | **0,0162 $** (8,8 min, del libro `resultados/vast/rejilla/`) |
| **Runs** | 144 (filtro contorno/Sobel × kernel 5/7/9 × N 4–1000 × 3 semillas), 1 escala, trazo continuo; control: los 72 brazos equivalentes de `rect-lin` (#38), re-evaluados con sus kernels |
| **Dataset / red** | los de `rect-lin` (`rect-lin-entreno-r20261008`, `rect-lin-banco-r20261008`); el detector de `rect-lin` con un filtro de bordes como único pre-proceso, aplicado igual a entrenamiento, banco y negativos |
| **Artefactos** | [`experimentos-cnn/2026-10-08-rectas-grosor-bordes/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-08-rectas-grosor-bordes) (README con tablas y 3 figuras; criterio en `instrucciones/02-criterio.md`, commit `580f96e`, antes de medir) |

**Resultado:** **Sobel ayuda mucho pero no llega al umbral.** Con el kernel 9×9 entrenado sólo con trazos finos, las rectas
gruesas (10–14 px, largo ≥ 16) pasan de **0,12** a **0,70** de recall (el criterio pedía 0,80). Las de 6–8 px pasan de
0,16 a **0,88**. Las finas apenas pierden (0,99 → 0,97) y los falsos positivos se quedan en **0,3 %**. A cambio, hacen
falta el doble de ejemplos (N90 128 contra 64). El recall cae con el grosor (0,69 / 0,54 / 0,44 a 10 / 12 / 14 px). La
hipótesis, sin comprobar, es que aprendió «dos bordes juntos», no «un borde». El **contorno** (x − erosión), que se predijo
mejor, queda por debajo de Sobel con 7×7 y 9×9. El Gabor sin entrenar no sirve con bordes. Con Sobel, las curvas gruesas
también se detectan todas como rectas. **4 de 7** hipótesis (G2, G3, G5 ✅).

**Lo que quedó pendiente:** (1) entrenar con algo de grosor (2–8 px) con Sobel delante, que es la prueba directa de la
hipótesis; (2) el mapa de detecciones en vez del max (una recta gruesa debería dar dos detecciones paralelas); (3) los
bordes de un solo lado, que necesitan un detector de varios canales; (4) el control re-evaluado difiere hasta 0,028 de lo
medido en #38 (kernels guardados con 4 decimales).
