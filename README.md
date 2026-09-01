# estudios-redes-neuronales — el repo central del proyecto

**Qué es:** el sitio donde vive lo que **no es de ningún repo en concreto** — los reportes de todos
los estudios y mediciones, el estado de los parámetros, y la referencia a las piezas que forman el
sistema.

**Empieza por aquí:**

| Si quieres saber… | Mira en |
|---|---|
| **las reglas de este repo** (lo que no se puede hacer mal) | [`CLAUDE.md`](CLAUDE.md) |
| **qué está fijado hoy** y qué sigue abierto | [`ESTADO.md`](ESTADO.md) — se reescribe |
| **qué se corrió, cuándo, con cuántas máquinas y qué costó** | [`reportes/README.md`](reportes/README.md) — sólo se añade |
| **qué repo hace qué** | la tabla de aquí abajo |
| **cómo se diseña aquí** | [`docs/`](docs/) |

---

## El charter, y no es decorativo

> Aquí va **prosa y veredictos**. Ni una línea de código, ni un byte de dato crudo, ni un `.json` de
> resultados, ni un peso. El dato vive donde lo dejó quien lo produjo; aquí se **resume y se enlaza**.

Existe porque un repo llamado «central» es exactamente el que acaba siendo *el sitio donde va lo que
no se sabe dónde poner*. Se comprueba en un comando:

```sh
find . -path ./.git -prune -o \( -name '*.py' -o -name '*.json' -o -name '*.npz' -o -name '*.pt' \) -print
#  -> VACÍO
```

## Los cinco repos que forman el sistema

Se **declaran** aquí, no se anidan. **Nada de submódulos**: un submódulo fija *un commit*, y
`foveal-vision` recibe del orden de cien commits de máquina al día (medido 2026-08-28: 195 de sus
últimos 212 los escribió una flota, no una persona), así que el pin nacería rancio — y chocaría de
frente con la convención de «rama propia por workspace», que apunta a una rama y no a un commit.

| Repo | Para qué existe | Qué produce | Qué necesita al lado | Su doc |
|---|---|---|---|---|
| [`foveal-vision`](https://github.com/stalinbeltran/foveal-vision) | **El experimento.** Entrena las dos redes (foveada y plana), define el espacio de búsqueda y lanza las flotas | `runs/`, `sweeps/`, `informe.json`; y el **criterio** de cada estudio, escrito antes de mirar, en `docs/plan-*.md` | `foveal-vision-data`, `image-text-sample-generator` y el lanzador, **como hermanos de directorio** | [`CLAUDE.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/CLAUDE.md) · [`docs/`](https://github.com/stalinbeltran/foveal-vision/tree/main/docs) |
| [`foveal-vision-data`](https://github.com/stalinbeltran/foveal-vision-data) | **Dónde se guarda lo medido.** Separado del código porque el código cambia por personas y esto lo escriben las máquinas | `runs/`, `sweeps/`, `studies/` por año-mes, y los `window-datasets/*/windows.npz` — que **no se pueden re-derivar** | `foveal-vision` lo busca como hermano; si falta, **cae al repo de código y no commitea nada, sin un solo error** | [`README.md`](https://github.com/stalinbeltran/foveal-vision-data/blob/main/README.md) |
| [`telegram-coordinator`](https://github.com/stalinbeltran/telegram-coordinator) | **El transporte.** Operar la máquina desde Telegram: enruta mensajes a ejecutores, y es quien dispara los estudios | Nada del experimento. Ejecutores, sesiones, workspaces | Nada. Descubre los ejecutores de los demás repos si están clonados | [`CLAUDE.md`](https://github.com/stalinbeltran/telegram-coordinator/blob/main/CLAUDE.md) |
| [`digital-ocean-dropplet-auto-launching`](https://github.com/stalinbeltran/digital-ocean-dropplet-auto-launching) | **El lanzador.** Crea y destruye máquinas en DigitalOcean y Vast.ai | `results/`, los tipos de máquina (`types/*.json`) | `DO_TOKEN` y `VAST_AI_API_TOKEN`; y la clave SSH **registrada antes** de crear nada | [`docs/reparto-mini-dev.md`](https://github.com/stalinbeltran/digital-ocean-dropplet-auto-launching/blob/main/docs/reparto-mini-dev.md) |
| [`image-text-sample-generator`](https://github.com/stalinbeltran/image-text-sample-generator) | **El generador del dato.** Renderiza las imágenes de texto sobre las que se entrena | Los renders (reproducibles byte a byte desde `specs.jsonl`) | Google Chrome de verdad, **no** el Chromium de Playwright | [`CLAUDE.md`](https://github.com/stalinbeltran/image-text-sample-generator/blob/main/CLAUDE.md) |

⚠ **Y este repo es el sexto**, con su [`CLAUDE.md`](CLAUDE.md) desde el 2026-09-01: una sesión
abierta aquí no recibía ninguna instrucción, y lo primero que puede hacer mal es **romper el
charter** escribiendo código o datos. Quien monte un workspace tiene que traérselo: sin él no hay forma de
comprobar si un estudio que se va a pagar ya se pagó una vez. Está en las listas de
`telegram-coordinator/scripts/workspace.mjs`, `scripts/cerrable.mjs`, `scripts/bench-preflight.mjs` y
`digital-ocean-dropplet-auto-launching/types/dev.json`.

## Dos decisiones de estructura, con su motivo

Las dos son **excepciones declaradas** a reglas del proyecto. Se anotan aquí, que es donde se
aplican, en vez de romperlas en silencio.

### 1. Excepción a la R7 — «un artefacto vive en la pieza de quien lo produce»

R7 pediría que el reporte de un barrido de `dropout` viviera en `foveal-vision`. Aquí no vive ahí, y
el precio es real: **quien clone sólo `foveal-vision` no tiene los veredictos**, sólo un puntero.

Se paga a propósito, por tres motivos medidos:

1. **R1 pesa más aquí.** En agosto de 2026, `foveal-vision` recibió 2.420 commits y los reportes 29
   — 83× de diferencia — y el 92 % de los de `foveal-vision` los escribe una máquina a mitad de una
   flota, no una persona al terminar un estudio. Son dos relojes y dos autores.
2. **R7 no sabe colocar 4 de los 21.** El `#1` mide droplets del lanzador con un script de
   `foveal-vision`; el `#2` se lanza con `vast_instance.py` (lanzador) envuelto en `vast-sweep.sh`
   (coordinador); el análisis de arquitectura tiene por sujeto los cinco repos y por productor a
   ninguno. Una regla que da respuesta ambigua para el 19 % de los casos no puede ser el único
   criterio.
3. **La mitad del encargo no cabe en ningún repo**: la información descriptiva del proyecto que no
   es de una pieza concreta.

⚠ **Lo que esto NO arregla:** el ciclo de dependencias entre repos que denunció el
[análisis de arquitectura](reportes/arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md)
§ 3.1 sigue **intacto** — esto no toca un solo `import` ni un solo `ROOT.parent`. Lo que sí
desaparece es la arista rara de la documentación: hasta ahora el repo que mide dependía
documentalmente del repo del **transporte**.

### 2. Aquí se trabaja en `main` — también desde un workspace

⚠ **Reformulado el 2026-09-01**: esto se llamaba «excepción a rama propia por workspace», lo que
daba a entender que la convención del proyecto eran las ramas. No lo es. **La regla general de los
seis repos es `main`**, porque un server nuevo hace un clon limpio; la rama propia es lo excepcional
y sólo aplica a **trabajos paralelos en workspaces separados del mismo dev**.

Lo que este repo añade es que **ni siquiera en ese caso** se separa: **se clona y se empuja en
`main` desde todos los workspaces.**

Motivo: un índice sólo-añadir no diverge de verdad (conflicta en la última línea: ruidoso y barato),
y **un veredicto parado en una rama es invisible para la máquina siguiente**, que es exactamente el
fallo que este proyecto ya pagó el 2026-08-14. Aquí el riesgo de no publicar supera al de chocar.

## Cómo se comprueba que esto sigue sano

```sh
# el charter (que no se convierta en el vertedero)
find . -path ./.git -prune -o \( -name '*.py' -o -name '*.json' -o -name '*.npz' \) -print   # VACÍO

# ningún reporte vive en dos sitios
find ~/src/telegram-coordinator/reportes ~/src/foveal-vision/reportes -name '*.md' | wc -l   # 2 (los punteros)

# el índice cuadra con el disco
find reportes -name '*.md' ! -name README.md | wc -l                                          # 21

# ESTADO.md no lleva historial dentro
grep -n "desactualizado\|ya no aplica\|~~" ESTADO.md                                          # VACÍO

# se clona solo y se lee solo
git clone https://github.com/stalinbeltran/estudios-redes-neuronales /tmp/solo && ls /tmp/solo/ESTADO.md
```

⚠ **Las cinco son comprobaciones que hay que acordarse de correr, o sea que todavía no existen.**
Convertirlas en CI es lo pendiente número uno, y coincide con la recomendación #1 del análisis de
arquitectura.
