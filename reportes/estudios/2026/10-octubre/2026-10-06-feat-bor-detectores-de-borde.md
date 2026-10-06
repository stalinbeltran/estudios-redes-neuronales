# `feat-bor` — los 13 detectores de `feat-ind32` viendo el BORDE de la tinta en vez de la tinta

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-06 00:53 → 02:02 (detectores en Vast, del libro); evaluación en el dev, 02:03 → 02:09 |
| **Instancias alquiladas** | **1** (Xeon E5-2680 v4, 28 vCPU, 62,8 GB, Texas), 26 detectores a la vez, rc 0, destruida |
| **Coste real** | **0,1098 $** (69 min a 0,0956 $/h, del libro `resultados/vast/detectores/`) |
| **Runs** | 2 representaciones (contorno · borde con signo) × 13 detectores × 1 semilla; compositores × 3 semillas |
| **Dataset / red** | entrenar: `feat-ind32-sinteticas-32px-r20261005` (2–4 px); prueba de grosor: `feat-bor-sinteticas-grueso-32px-r20261006` (2–12 px, publicado aquí); dígitos: `uci-optdigits-orig-32px-r20261005`. La red de `feat-ind32` (41.825 parámetros; 42.257 con 4 canales) |
| **Artefactos** | [`experimentos-cnn/2026-10-06-features-bordes/`](https://github.com/stalinbeltran/experimentos-cnn/tree/main/2026-10-06-features-bordes) (README con todas las tablas) |

**Resultado: el borde NO quita el problema del grosor.** Los dos aprenden (13/13 «a medias» o mejor; F1 medio 0,897 y 0,915
contra 0,916 de las líneas), pero con trazos de 6–12 px **pierden recall** (contorno 0,82 → 0,63; signo 0,84 → 0,73) donde las
líneas lo conservan (0,86 → 0,86) y lo que ganan es falsos positivos (0,03 → 0,13). Un trazo grueso tiene dos bordes
separados y el detector sólo vio bordes pegados. En los dígitos, el compositor cae de **0,949** a **0,805** (contorno) y
**0,865** (signo), y κ al engrosar/adelgazar no mejora. Lo único confirmado: con el borde con signo, los arcos dejan de
encenderse en los 1 (76 %/70 % → 2 %/8 %). H1 ✅ · H2 ❌ · H3 ✅ (signo) · H4 ❌ · H5 ❌.

**Lo que quedó pendiente:** (1) **¿se distinguen las curvas de las rectas?** — pedido por el dueño al terminar este
entrenamiento; la primera medida está en `feat-fallos` (finas sí, gruesas no); (2) bordes entrenados con trazos gruesos, que
sólo hace falta si `feat-fallos` S2 (líneas entrenadas con 2–12 px) no basta; (3) una sola semilla, y con 4 canales la
inicialización cambia: el efecto de la representación no se separa del de la inicialización. La estimación era 20–30 min y
0,02–0,05 $: 26 procesos en 28 vCPU fueron unas 4 veces más lentos por núcleo que el dev.
