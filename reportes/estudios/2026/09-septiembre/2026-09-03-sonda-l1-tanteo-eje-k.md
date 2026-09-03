# Sonda L1 — tanteo del eje `k`: ¿aprenden filtros genéricos los kernels de la primera capa?

| | |
|---|---|
| **Inicio (UTC)** | 2026-09-03 02:33:54 |
| **Fin (UTC)** | 2026-09-03 04:40:13 |
| **Reloj** | 2,1 h |
| **Instancias alquiladas** | **0** — corrió en el droplet dev, 2 vCPU |
| **Coste real** | **0 $**. Aquí el coste es **reloj**, no factura |
| **Runs** | 8 (4 valores de `k` × 2 de λ), 30 épocas, semilla **1** |
| **Dataset** | `dirty1000-80px-16px-r20260827` |
| **Artefactos** | [`foveal-vision-data/sondas/l1/`](https://github.com/stalinbeltran/foveal-vision-data/tree/main/sondas/l1) — `tabla.md`, `resumen.json`, y por run: `config.json`, `metrics.jsonl`, kernels `.npy`, hojas de contactos |
| **Criterio** | escrito **antes** en [`foveal-vision/docs/plan-sonda-l1-2026-09-02.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/docs/plan-sonda-l1-2026-09-02.md) §4.2 |

> Este reporte **resume y enlaza**. El veredicto contra el criterio, con su tabla completa y sus
> matices, vive en el §9 del plan, que es donde se escribió el criterio antes de mirar.

---

## Qué se preguntaba

En `fov16-optimo-mask` los 16 kernels de la primera capa **no aprendieron filtros genéricos**:
su energía en el subespacio clásico 6D es 0,688 contra 0,667 de un kernel aleatorio — **1,03×**,
o sea nada. La hipótesis era que **L1 no está bajo presión**: detrás hay una cabeza de 153.660
parámetros que extrae las esquinas de casi cualquier proyección.

La sonda quita esa red de seguridad: un autoencoder de **una capa por lado**, decodificador
lineal sin sesgo, nada en medio, sobre la **misma vista 20×20** que ve producción. El modelo
*son* los kernels. Este tanteo barre el eje que lleva la premisa —`k` ∈ {3,5,7,9}— a K=16 fijo,
con λ ∈ {0 (control), calibrada por celda a ~10 % de activación}.

## Qué salió: **ninguna de las 8 pasa**, y falla de forma estructurada

| brazo | métrica de forma | salud del código |
|---|---|---|
| **λ = 0** | Gabor Δ/margen **0,615–0,836**, sube con `k`, pasa en los cuatro | 1–7 canales **muertos**, 7–9 **saturados**; en `k5` **cero vivos** |
| **λ calibrada** | Gabor Δ/margen **0,197–0,360**, orientación ≤ 0,056: **por debajo del 0,40** | **0 muertos, 0 saturados, 16 vivos**, activación 5,0–6,7 % (en banda) |

**El brazo λ=0 aprende kernels DELTA**, o sea la solución identidad: kernel delta → `z` copia de
`x` → decodificador delta, y `R²` de reconstrucción exactamente **1,000**. Medido por la anchura
de la envolvente del Gabor ajustado — **σ 0,49–0,57 px** contra **1,41–1,49 px** de un kernel
aleatorio; en k=9 la mitad de los kernels están en el suelo de σ.

**Respuesta a la pregunta del encargo: NO.** Bajo presión real —código disperso y sano— los
kernels salen **localizados y no orientados**, no Gabors. Sin presión, el modelo coge el atajo de
la identidad. Y contesta de paso la tercera cláusula del encargo: **la esparsidad no aportó
estructura de Gabor**; λ=0 puntúa más alto, pero sólo degenerando.

## ⚠ Lo que este resultado le debe a las métricas sin plantilla

**Sin ellas el estudio habría concluido lo contrario.** El ajuste a Gabor —la métrica principal
que pedía el encargo— supera su nulo al 5 % en **las 8** configuraciones, y con el umbral
absoluto propuesto (0,25) los cuatro runs de λ=0 lo habrían pasado holgadamente. El veredicto
habría sido *«éxito: salen filtros tipo Gabor»* sobre **kernels delta**.

Las dos métricas espectrales —concentración en **banda** y en **orientación**, de la FFT 2D con
rejilla común— dijeron que no, porque un delta tiene espectro **plano e isótropo**. Las pidió el
dueño al revisar el código, tras detectar que la métrica del subespacio clásico estaba rota por
otra razón (ver abajo).

## Dos cosas medidas por el camino que valen fuera de este estudio

1. **La métrica del subespacio clásico no significa lo mismo con y sin normalización de
   contraste.** `enriq` sale 0,47–0,61 —por **debajo** de su nulo de 1,0— y la causa está
   medida: la base clásica es de **baja frecuencia** y la normalización obligatoria deja los
   kernels en **alta**. Sin normalizar, la misma celda da `enriq` **1,01**.
   ⚠ Y producción **no** normaliza, así que el **0,688** de `fov16-mask-p20` y el `enriq` de la
   sonda **nunca fueron comparables**.
2. **Lo que asienta la fracción de activación son los PASOS del optimizador**, no las épocas ni
   el tamaño del dataset. Con las mismas 8.000 ventanas: 64 pasos → 24,3 %, 256 → 4,3 %, 640 →
   3,6 %. Una calibración medida a 64 pasos sobreestima **seis veces** y llega a invertir su
   propia conclusión.

## Lo que quedó pendiente

1. **Una sola semilla.** El patrón es consistente en los cuatro `k`, pero **nada aquí declara al
   5 %**: acota, no afirma. Repetir con 3 semillas cuesta ~1 h.
2. **`K` no se barrió**: todo es K=16. La sobrecompletitud (K=32 con k=9 sería 32×) es justo el
   eje que el encargo asociaba a que emergiera estructura.
3. **La rejilla completa (12–15 h) NO se lanzó**, y la recomendación es no lanzarla: el eje `k`
   está barrido y el patrón es idéntico en los cuatro valores.
4. **El umbral de magnitud (0,40) sigue negociable** y es del dueño. Con 0,30 pasarían dos runs
   del brazo calibrado; con 0,20, seis de los ocho. **No se cambia después de mirar.**
5. **La fase 2** —congelar el codificador ganador como L1 de la rama central y reentrenar— pedía
   que la fase 1 tuviera éxito, así que **no se ejecuta**.
6. **El control λ=0 merece un diseño mejor.** «Sin esparsidad» acabó siendo «solución
   identidad»; separar *«¿ayuda la esparsidad?»* de *«¿evita el atajo?»* pide un control que
   bloquee la identidad de otro modo (p. ej. atar decodificador y codificador).
