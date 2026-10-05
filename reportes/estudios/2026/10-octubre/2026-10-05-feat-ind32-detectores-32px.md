# `feat-ind32` — detectores independientes por feature, con la entrada a 32×32 en vez de 8×8

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-05 12:44 → 13:00 (detectores en Vast); compositores en el dev, 14:05 → 14:07 |
| **Instancias alquiladas** | **1** (Xeon E5-2680 v4, 28 vCPU, 31 GB, Vietnam), 13 detectores a la vez, rc 0, destruida |
| **Coste real** | **0,0172 $** (15,5 min a 0,066 $/h, del libro `resultados/vast/detectores/`) |
| **Runs** | 13 detectores × 1 semilla; compositores × 3 semillas |
| **Dataset / red** | `feat-ind32-sinteticas-32px-r20261005` y `uci-optdigits-orig-32px-r20261005` (los de `feat-ind` sin reducir, comprobado bit a bit); detector 32×32 → mapa 8×8, 41.825 parámetros |
| **Artefactos** | [`experimentos-cnn/2026-10-05-features-independientes-32px/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-05-features-independientes-32px) (README con todas las tablas) |

**Resultado:** a 32×32 los detectores mejoran mucho —arcos y esquinas pasan de «no aprendió» a «a medias», las rectas a
«aprendió», la precisión de 0,61–0,67 a 0,88–0,98— y el par 1↔8 baja de 22 a 5. **Pero el compositor no mejora**:
posicional 0,949 contra 0,959 a 8×8 (empate en el borde inferior del criterio), y la curva llega a 0,975 con 900
dígitos contra 0,992 del banco fino+grueso a 8×8. El máximo 3×3 sube el desplazado +0,125. Veredicto por hipótesis y
lectura, en el README del experimento.

**Pendiente:** si el hueco es de transferencia sintético → manuscrito (lo más probable) no está medido; 1 semilla de
detectores; la opción de mapa 32×32, sin correr.

## Añadido el mismo día: 13 contra 13

Con sólo los 13 detectores del banco fino a 8×8 (corrida 14 de `feat-ind`, 0 $, mismo test de 717), **8×8 gana en
todos los N**: 0,831 / 0,964 / 0,985 con N = 36 / 180 / 1080, contra 0,792 / 0,942 / 0,974 a 32×32; y con B,
desplazado 0,774 contra 0,733. Tabla en el README de `feat-ind32`.
