# Los pesos de un entrenamiento sobreviven a la máquina que los produjo

**Tipo:** `infraestructura` — el sujeto medido no es la red, es **la cadena** que lleva un
artefacto desde una máquina alquilada hasta git.

| | |
|---|---|
| **Inicio (UTC)** | 2026-08-31 00:10:47 *(del log; `cuando − reloj_min` da lo mismo)* |
| **Fin (UTC)** | 2026-08-31 00:24:41 |
| **Reloj** | 13,9 min |
| **Instancias alquiladas** | **5** *(1 entrenó; 4 fallaron antes de estar listas y se repusieron)* |
| **Coste real** | **0,0295 $** — de los cuales 0,0173 $ en las 4 que no entrenaron |
| **Recorrido** | `dt-pesos` (1 punto, 3 épocas). No mide nada de la red |
| **Rama** | `dataTests` de `foveal-vision-data`, a petición del dueño |
| **Prefijo** | `dt-` |

## Qué se comprobaba, y por qué existía la duda

La noche del 29 al 30 de agosto se entrenó `fov-optimo-p20` (69 épocas en Vast, f1 0,9430). Se
commitearon sus cuatro ficheros de descripción —`config.json`, `metrics.jsonl`, `status.json`,
`summary.json`— y **ningún peso**. Al rehacer el dev, la red entrenada dejó de existir: no estaba en
ninguna rama de ninguno de los cinco repos (`git rev-list --objects --all`). Hubo que reentrenarla
entera, ~2 h 22 min. El detalle del inventario está en
[`2026-08-30-sobrevivir-a-rehacer-el-dev.md`](../../../arquitectura/2026/08-agosto/2026-08-30-sobrevivir-a-rehacer-el-dev.md).

**No fue un fallo: estaba diseñado así**, y escrito en tres sitios. La razón para cambiarlo la puso
el dueño: *hay que poder probar a mano un entrenamiento*. Sin pesos la web app enseña las imágenes
sin cajas, la métrica de tarea no se puede puntuar, y «la red detecta mal» no se distingue de «no
hay red».

**La cadena tenía TRES puertas, y bastaba una cerrada para perderlos. Ninguna daba error:**

| # | Dónde | Qué hacía |
|---|---|---|
| 1 | `PULL`, el `find` que corre **en la máquina alquilada** | nombraba sólo los 4 ficheros pequeños: los pesos no subían ni al tar |
| 2 | `LIBRO`, lo que se extrae de un tar traído | los mismos 4 nombres |
| 3 | el `.gitignore` del repo de datos | `*.pt` |

## El criterio, escrito antes de mirar

**Pasa** si y sólo si, en la rama `dataTests`, aparecen `best.pt` y `last.pt` del run de la flota,
los dos cargan con `torch.load`, y `last.pt` corresponde a la última época. **Falla** si llega sólo
la descripción, que es lo que pasó con `fov-optimo-p20`.

## Resultado: PASA por los dos caminos

Verificado **desde un clon limpio** de la rama, no desde el árbol de trabajo:

| | `best.pt` | `last.pt` | `load_model` |
|---|---:|---:|---|
| **Vast** (`sweeps/dt-pesos/runs/dt-pesos-0000-lr0p0014`) | 680.793 B · época 3 | 2.050.697 B · época 3 | red de 168.652 parámetros |
| **Dev** (`runs/dt-dev-pesos`) | 680.793 B · época 2 | 2.050.697 B · época 2 | red de 168.652 parámetros |

`best.pt` lleva sólo pesos; `last.pt` lleva además el optimizador, y por eso es 3× más grande.

**La cadencia también quedó probada.** Con `FV_EPOCAS_POR_PESOS=2` sobre un run de 3 épocas, cada
`.pt` aparece en **2 commits** de la rama: la recogida intermedia y el tirón final. No es un detalle:
git guarda todas las versiones commiteadas, así que subir `last.pt` en cada época serían 2 MB ×
época × run —140 MB en un run de 70 épocas—, que es exactamente por lo que estaba excluido.

## Lo que quedó pendiente

1. **En el dev, guardar sigue siendo manual.** `fv-train` imprime el comando al terminar, pero no
   commitea. En esta prueba los pesos del run local **sí** quedaron commiteados, pero **por
   accidente**: el libro de a bordo de la flota hace `git add -A` sobre todo el repo de datos y se
   los llevó de paso. Sin una flota corriendo a la vez, se habrían quedado en disco. El hueco sigue
   abierto y está dicho en `foveal-vision/docs/entrenar.md`.
2. **4 de 5 máquinas fallaron antes de estar listas** (0,0173 $ de 0,0295 $, el **59 %** del gasto).
   Con un barrido de un solo punto no se puede distinguir mala suerte de un problema del catálogo.
3. **La rama `dataTests` no se fusiona.** Es de prueba: contiene un recorrido que no mide nada.

## Cómo reproducirlo

```bash
cd ~/src/foveal-vision
FV_EPOCAS_POR_PESOS=2 .venv/bin/python scripts/estudio_flota.py \
    --sweep dt-pesos --cpu E5-26 --criba 2 --git --horas-max 1 --prefijo dt- --yes
```

Los cuatro tests que fijan las tres puertas y la cadencia están en
`foveal-vision/tests/test_pesos_a_git.py`; los cuatro fallan con el código anterior.
