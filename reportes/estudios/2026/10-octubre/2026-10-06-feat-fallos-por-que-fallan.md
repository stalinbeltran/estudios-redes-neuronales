# `feat-fallos` — por qué fallan los detectores de features al leer dígitos, y qué lo arregla (por iteraciones)

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-06 01:25 → 03:58 (estudio en el dev: la hora de la iteración 1 y la del último resultado); S2 en Vast 02:14 → 02:33 (del libro) |
| **Instancias alquiladas** | **1** (Xeon E5-2680 v4, 28 vCPU, 62,8 GB, Texas), los 13 detectores de S2 a la vez, rc 0, destruida |
| **Coste real** | **0,0303 $** (19,0 min a 0,0956 $/h, del libro `resultados/vast/lineas-grueso/`); el resto, 0 $ en el dev |
| **Runs** | 13 detectores × 1 semilla (S2); 21 combinaciones banco × preprocesado, cada una con compositor × 3 semillas, curva 36/180/1080, κ y prueba gruesa; 3 con compositor tolerante; 6 en la confirmación ciega |
| **Dataset / red** | dígitos `uci-optdigits-orig-32px-r20261005`; prueba gruesa `feat-bor-sinteticas-grueso-32px-r20261006`; S2 entrena con `feat-fallos-sinteticas-grueso-32px-r20261006` (publicado aquí, 2–12 px; reducido a 8×8 es bit a bit el de la corrida 4 de `feat-ind`); bancos leídos por id y huella de `feat-ind32`, `feat-bor` y `feat-cortas` |
| **Artefactos** | [`experimentos-cnn/2026-10-06-por-que-fallan/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-06-por-que-fallan) (README con todas las tablas y figuras; cada iteración, con su criterio escrito antes, en `instrucciones/02-criterio.md`) |

**Resultado:** la causa principal es el **grosor**: el trazo de los dígitos es ~2,8 veces el de las features de entrenamiento
(mediana 5,8 px contra 2,1; los 1, 9,8). Con trazo grueso, un detector entrenado fino **ve de más** (algún arco se enciende en
el 45 % de las rectas gruesas y en el 89 % de los 1), y el error se concentra en el tercil más grueso. Las curvas **sí** se
distinguen de las rectas cuando el detector ha visto ese grosor (0,7 % con el banco entrenado con 2–12 px). Quitar el grosor
**de la imagen** no sirve (esqueleto 0,876; erosión según grosor 0,689: uno pierde lo relleno, otro lo fino), y los bordes
tampoco (#34). Lo que funciona es dárselo todo al compositor: **finos + gruesos + cortas, crudo + normalizado: 0,9716** (contra
0,949; −44 % de errores; S6b), y además mucho más estable al cambiar el grosor (κ al adelgazar 0,706 contra 0,345). Desinclinar
no ayuda con 180 ejemplos; el compositor tolerante a la posición ayuda con 36 (S6b + tolerante: 0,893 con 36, contra 0,792).
**Confirmado a ciegas** (iteración 7, criterio escrito antes): en 3823 dígitos de otros escritores que ningún paso usó para
elegir, S6b lee **0,966** contra **0,925** de la referencia (−54 % de errores, más que en val).

**Lo que quedó pendiente:** lo que queda de errores son dígitos con **zonas macizas** (4 cerrados con el triángulo relleno,
8 gruesos) y ninguna transformación de la imagen lo arregla sin romper otra cosa; el compositor no lineal (no probado); una
sola semilla por detector en los tres bancos entrenados; y el coste de inferencia de S6b (68 pasadas de detector contra 13,
no medido).
