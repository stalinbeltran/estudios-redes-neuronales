# `feat-ind32` — curvas de CNN entrenadas de punta a punta contra los detectores independientes

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-05 18:03 → 18:08 (libro `resultados/vast/cnn/` del experimento) |
| **Instancias alquiladas** | **1** (Xeon E5-2680 v4, 28 vCPU, 31 GB, Vietnam), 120 entrenamientos a la vez, rc 0, destruida |
| **Coste real** | **0,0056 $** (5,1 min a 0,0707 $/h, del libro) |
| **Runs** | 120: 2 CNN × 60 particiones (4 tamaños de dataset × 5 fracciones de train × 3 semillas) |
| **Dataset / redes** | `uci-optdigits-orig-32px-r20261005` (5620 dígitos, 43 escritores); las 60 particiones de la ganancia de `feat-ind32` (comprobado por huella). CNN de 3 capas del repo (la de `ruido-nist`, 1.338 parámetros, 8×8) y LeNet-5 sobre 32×32 (61.706); 3996 pasos de 20, Adam, sin selección ni aumento |
| **Artefactos** | [`experimentos-cnn/2026-10-05-features-independientes-32px/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-05-features-independientes-32px) (README § «Curvas de CNN», `resultados/curvas-cnn.png`, `nn/curvas-cnn/`) |

Pedido del dueño: comparar las curvas de aprendizaje de los detectores independientes (`feat-ind`, `feat-ind32`) con
modelos tradicionales. En el repo sólo había CNN medidas en N = 180; esto da las curvas enteras, sobre los mismos datos.

**Resultado:** los detectores 8×8 + compositor lineal ganan a LeNet-5 en todo N hasta **~1440 muestras** y necesitan
**~3× menos muestras** para el 90–95 % (N(95 %) = 133 contra 412). Con 2000 muestras LeNet-5 los pasa por poco (98,5 contra
98,1 %): sigue subiendo mientras los detectores se quedan en el techo de su compositor. La CNN de 3 capas del repo es la peor
de las seis en todo el rango y queda por debajo de una regresión logística sobre píxeles hasta N ≈ 1550. Del criterio escrito
antes: 2 de 4 predicciones se cumplen; los dos cruces llegan más tarde de lo previsto.

**Pendiente:** LeNet-5 con aumento de datos o con selección por validación; los mismos pasos (3996) para todo N es una
decisión que favorece a los N pequeños en épocas.
