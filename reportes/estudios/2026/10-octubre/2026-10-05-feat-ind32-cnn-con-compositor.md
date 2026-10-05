# `feat-ind32` — CNN con el compositor de los detectores: A, B y C

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-05 19:31 → 19:38 (libros `resultados/vast/compositor/` del experimento) |
| **Instancias alquiladas** | **2** a la vez: B en AMD EPYC 7502 (32 vCPU, 6,8 min) y C + A en EPYC 7763 (32 vCPU, 4,8 min); rc 0, destruidas |
| **Coste real** | **0,0268 $** (0,0157 + 0,0111, de los libros) |
| **Runs** | 180: 3 variantes × 60 particiones (4 tamaños de dataset × 5 fracciones de train × 3 semillas) |
| **Dataset / redes** | `uci-optdigits-orig-32px-r20261005`, las particiones de la ganancia de `feat-ind32` (huella). A: CNN de 3 capas + compositor (9.943 parámetros); B: los 13 detectores como convoluciones agrupadas + compositor, desde cero (191.383); C: B arrancando de los detectores sintéticos de `feat-ind`, fase congelada + ajuste fino. 3996 pasos de 20, Adam, sin selección ni aumento |
| **Artefactos** | [`experimentos-cnn/2026-10-05-features-independientes-32px/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-05-features-independientes-32px) (README § «CNN con el compositor de los detectores», `resultados/compositor.png`) |

Pregunta del dueño: ¿se puede entrenar la CNN de 3 capas con el compositor de las redes de features definidas? Sí (es
derivable), y se probaron tres variantes.

**Resultado (criterio escrito antes: 4 de 4):** **C —detectores sintéticos + ajuste fino— es la mejor en todo N** y supera
a LeNet-5 también con 2000 muestras (98,9 contra 98,5 %); para el 95 % necesita 108 muestras, **3,8× menos que LeNet-5**.
B, la misma arquitectura aprendida desde cero, queda por debajo de los detectores sintéticos hasta N ≈ 800 (necesita 230
muestras para el 95 %, contra 133). A, la CNN de 3 capas con el compositor en vez del promedio global, mejora a la del repo
en todo N (+5 puntos en N ≈ 180, hasta +15). Y la fase congelada de C muestra que parte de la ventaja era del protocolo del
compositor (Adam en minilotes sin L2): hasta +4,8 puntos sobre el de siempre, con los mismos detectores.

**Pendiente:** C sin fase congelada; separar el efecto del L2 y del minilote en el compositor; aumento de datos.
