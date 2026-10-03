# Plan de kernels del banco, fases 2–4 — 49 kernels aprendidos para `banco-k`, y cuáles sirven

| | |
|---|---|
| **Inicio (UTC)** | 2026-10-02 10:49 (fase 2 de `bor-k`) |
| **Fin (UTC)** | 2026-10-02 23:54 (la última máquina de la fase 3) |
| **Instancias alquiladas** | **33** en Vast: fase 2 `bor-k` 13 (9 + 4 reintentos), fase 2 `bor-ae` 10 (9 + 1), fase 3 10 (9 + 1). Los 6 reintentos: hosts cuyo sshd no contestó o que no aceptaron la clave (bloqueados) |
| **Coste real** | **0,7021 $** *(del libro: 0,1393 + 0,3665 + 0,1963)*. Con las fases 0–1 (#25, 0,0063 $): **0,7084 $** el estudio entero |
| **Artefactos** | [`experimentos-cnn/…/banco-kernels/resultados/KERNELS.md`](https://github.com/stalinbeltran/experimentos-cnn/blob/main/2026-09-08-banco-kernels/resultados/KERNELS.md) (el veredicto de cada kernel) · informes de [`bor-k`](https://github.com/stalinbeltran/experimentos-cnn/blob/main/2026-10-01-bordes-kernel/resultados/INFORME.md) y [`bor-ae`](https://github.com/stalinbeltran/experimentos-cnn/blob/main/2026-10-01-bordes-autoencoder/resultados/INFORME.md) · libros en `resultados/vast/` de cada uno |
| **Criterios** | escritos antes de medir: `02-criterio.md` de cada productor y el §2 de la especificación del banco |

> Resume y enlaza. El detalle por kernel vive en `KERNELS.md`.

## Qué se preguntaba

Producir kernels de **todos** los tamaños que acepta el banco (k = 3…19), con tres
procedimientos —`bor-k` (supervisado, un kernel para los cuatro bordes), `bor-ae` (autoencoder
de un filtro) y `bor-pca` (PC1 de parches de borde)—, 3 semillas donde se entrena, y medir en el
banco cuáles mejoran a un kernel aleatorio de igual norma y `k` (§2.1) y cuáles además reducen la
brecha train–eval respecto de la identidad (§2.2). Fase 1 y su regla de deriva: #25.

## Qué salió

| productor | k ≤ 11 | k = 13 | k = 15 | k = 17 | k = 19 |
|---|---|---|---|---|---|
| `bor-k` (3 semillas) | no declara (15/15) | **útil 3/3** | **útil 3/3** (1 generaliza) | **útil 3/3** (1 generaliza) | **útil 3/3** |
| `bor-pca` | k=11 útil; k ≤ 9 no | **útil** | **útil** | **útil** | **útil y generaliza** |
| `bor-ae` (13 no-delta) | no declara | no declara | no declara | — (delta) | — (delta) |

**49 kernels: 17 útiles (§2.1), 3 de ellos además generalizan (§2.2); 32 no declaran.**

1. **El tamaño manda más que el método**: ningún kernel de `k ≤ 9` es útil, y desde `k = 13` lo
   son todos los de `bor-k` y `bor-pca`. Ganan al aleatorio de su `k` por +0,03–0,05 de IoU
   `eval` (márgenes ~0,025).
2. **`bor-k` aprende sus cuatro bordes desde `k = 7`**, pero eso **no** basta para ser útil en el
   banco hasta `k = 13`: aprender la tarea del productor no es lo mismo que servir de
   preproceso.
3. **`bor-pca`, sin entrenar y a 0 $, iguala casi a `bor-k`** y da uno de los tres que
   generalizan (`borpca-k19`).
4. **`bor-ae` no aporta**: con dispersión el filtro se vuelve la identidad (tanteo), con
   `λ = 0` los extremos también, y los 13 que no son delta no declaran.
5. Los tres que generalizan (`bork-k15-s1`, `bork-k17-s2`, `borpca-k19`) son **uno por
   productor/k suelto, no tres semillas de lo mismo**: la §2.2 aquí es frágil.

## Lo que quedó pendiente

- **Ningún reporte a `ESTADO.md`**: el banco mide kernels como preproceso de su CNN de
  referencia, no un parámetro de `foveal-vision`.
- La §2.2 con 3/17 y sin repetirse entre semillas pide más semillas antes de creérsela.
- `bor-ae` con otra presión anti-identidad (ruido en la entrada, código submuestreado) sería
  otro experimento.
- Lecciones de infraestructura ya aplicadas en el lanzador: `TRABAJO_VCPU` (nproc miente en
  Vast), 10 min de espera a sshd, y bloqueo de hosts que no aceptan la clave (6 de 33).
