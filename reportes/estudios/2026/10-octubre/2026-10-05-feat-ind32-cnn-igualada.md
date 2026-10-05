# `feat-ind32` — la CNN del repo con el terreno igualado (+ padding, + cabeza densa, + capacidad)

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-05 20:11 → 20:16 (libros `resultados/vast/igualada/` del experimento) |
| **Instancias alquiladas** | **2** a la vez (AMD EPYC 7C13, 37 vCPU · EPYC 7B13, 32 vCPU), 4,5 y 4,2 min; rc 0, destruidas |
| **Coste real** | **0,0174 $** (0,0087 + 0,0087, de los libros) |
| **Runs** | 240: 4 variantes × 60 particiones (4 tamaños de dataset × 5 fracciones de train × 3 semillas) |
| **Dataset / redes** | `uci-optdigits-orig-32px-r20261005`, las particiones de la ganancia de `feat-ind32` (huella). La CNN de 3 capas del repo (`ruido-nist`) con: 1 padding · 2 + cabeza densa (aplanar → lineal) · 3 + capacidad (186.538 parámetros, lr 1e-3) · 3b ídem con lr 3e-3. 3996 pasos de 20, Adam, sin selección ni aumento |
| **Artefactos** | [`experimentos-cnn/2026-10-05-features-independientes-32px/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-05-features-independientes-32px) (README § «La CNN del repo con el terreno igualado», `resultados/igualada.png`) |

Observación del dueño: la CNN del repo tenía un hándicap serio por la reducción de sus salidas (8 → 2 sin padding). Se le
quitaron las desventajas una por peldaño.

**Resultado:** **el padding solo no ayuda** (+0,3 puntos en N ≈ 180; −20 con 10 muestras y −3 con ≥ 800): el hándicap era la
**cabeza** —el promedio global tira la posición—, no la reducción. La cabeza densa da el salto (+4,5 puntos en N ≈ 180, +12
con 40) y la capacidad añade poco (+0,3 a +1,2). Con el terreno igualado, la CNN tradicional llega a LeNet-5 (N(95 %)
357–583 contra 412) y a la arquitectura de los detectores aprendida desde cero con muchos datos, pero **C —detectores
sintéticos + ajuste fino— sigue ganando en todo N** y necesita **3,3–5,4× menos muestras** para el 95 % (108). Criterio
escrito antes: 1 de 4 predicciones enteras, 2 a medias (la cabeza da algo menos de lo previsto; el lr explica parte del
peldaño 3).

**Pendiente:** BatchNorm, aumento de datos y selección por validación en la CNN igualada (aquí no, para no cambiar el
protocolo de las demás).
