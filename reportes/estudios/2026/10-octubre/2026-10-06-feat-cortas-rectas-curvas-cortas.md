# `feat-cortas` — detectores de rectas y curvas CORTAS, y leer los dígitos con ellos

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-06 02:15 → 02:31 (detectores en Vast, del libro); evaluación y diagnóstico en el dev, 02:31 → 02:48 |
| **Instancias alquiladas** | **1** (Xeon E5-2686 v4, 18 vCPU, 31,4 GB, Nevada), 8 detectores a la vez, rc 0, destruida |
| **Coste real** | **0,0145 $** (15,8 min a 0,0551 $/h, del libro `resultados/vast/detectores/`; 9 de esos minutos, arrancando la máquina) |
| **Runs** | 8 detectores × 1 semilla; compositores × 3 semillas |
| **Dataset / red** | `feat-cortas-sinteticas-32px-r20261006` (publicado aquí: 4 rectas de 6–12 px y 4 curvas de radio 5–10 px y 60–100°, 2–4 px de grosor); dígitos `uci-optdigits-orig-32px-r20261005`; la red y la receta de `feat-ind32` sin tocar |
| **Artefactos** | [`experimentos-cnn/2026-10-06-rectas-curvas-cortas/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-06-rectas-curvas-cortas) (README con todas las tablas) |

**Resultado:** los 8 se aprenden (2 «aprendió», 6 «a medias») y **distinguen la curva corta de la recta corta** (1,2 % y
1,5 % de falsos positivos cruzados, contra el 10–25 % que esperaba). Para leer dígitos, **solas pierden** contra las 13 largas
de `feat-ind32` en las tres vistas (crudo 0,880 contra 0,949), y el diagnóstico escrito antes de medirlo dice **por qué**: no
es el largo —como trazos, las cortas ganan a las largas, 0,880 contra 0,843— sino el **vocabulario**: les faltan el lazo y
las esquinas, que un compositor lineal no puede construir juntando trozos; con ellos prestados, 0,945 (≈ 0,949). Juntas con
las largas, **0,9606**, la combinación más alta medida en ese momento (H3 pedía 0,966: ❌ por 0,006).

**Lo que quedó pendiente:** las cortas en las dos vistas y junto a todo lo demás (la S6 de `feat-fallos`, #35); cortas
entrenadas con trazo grueso (una recta de 8 px enciende curvas cortas); una sola semilla.
