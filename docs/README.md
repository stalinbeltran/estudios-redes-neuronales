# `docs/` — lo descriptivo del proyecto que no es de ningún repo

Aquí va la información que describe **el proyecto**, no una pieza. La prueba para saber si un
documento pertenece aquí es una pregunta, y tiene que salir «sí» a las dos:

1. **¿Gobierna o describe a más de un repo?** Si sólo aplica a uno, va en ese repo (R7).
2. **¿Sigue siendo cierto si mañana se reescribe cualquiera de los cinco repos?** Si el documento
   describe un mecanismo concreto —un script, un hook, un formato de fichero— va con el mecanismo,
   porque es lo único que se actualiza cuando el mecanismo cambia.

**Lo que NO va aquí, con nombre y apellido:**

| Documento | Dónde vive y por qué |
|---|---|
| `docs/agentes.md` | En `telegram-coordinator`: documenta `scripts/triage.mjs` y `.claude/settings.json`, que viven ahí. El documento y el mecanismo se actualizan juntos o no se actualizan |
| `docs/plan-*.md` | En `foveal-vision`: es el **criterio escrito antes de mirar**, y lo escribe quien va a correr el estudio |
| `docs/reparto-mini-dev.md` | En el lanzador: describe sus tipos de máquina |
| `instructionsNewNN.md` | En `foveal-vision`: describe la geometría de *su* red |

## Qué hay hoy

Nada todavía. El directorio existe con esta guía porque un sitio sin criterio de admisión se llena
de lo que no cabía en otro sitio, que es justo lo que el charter del repo intenta evitar.

## Un candidato que se MIRÓ y se dejó donde estaba

**`telegram-coordinator/docs/reglas-de-diseno.md`** — las 19 reglas de diseño. Por la primera
pregunta pasa sin discusión: gobiernan los cinco repos, y el propio documento dice de sí mismo
*«el ejemplo es de aquí, la regla no»*. Se planteó traerlo aquí en la centralización del
2026-08-29 y **se decidió que no**, por la segunda pregunta.

Motivo, comprobado con `grep` ese día: el documento **no es prosa suelta, está cableado a un
mecanismo** que vive en el coordinador y que se lee por **ruta relativa**:

| Quién lo nombra | Qué pasa si se muda |
|---|---|
| `scripts/triage.mjs:79` | Es el hook `UserPromptSubmit`: **dispara en cada mensaje** e inyecta `entra por § 0 de docs/reglas-de-diseno.md`. Pasaría a nombrar una ruta que no resuelve desde el repo donde corre |
| `.claude/agents/arquitecto.md:15` | *«Abre `docs/reglas-de-diseno.md` y usa su § 0»* — es la primera instrucción del agente |
| `.claude/agents/revisor.md:33` | *«¿Rompe una de las 19 reglas de `docs/reglas-de-diseno.md`?»* |

O sea: **el documento y las tres cosas que lo leen se actualizan juntos o no se actualizan**, que
es exactamente lo que la segunda pregunta de arriba busca. Traerlo aquí obligaría a que un hook que
corre en cada mensaje dependa de que otro repo esté clonado — y cuando no lo esté, fallaría
callándose, que es la forma cara.

⚠ **Se mueve el día que el mecanismo deje de leerlo por ruta relativa**, no antes. Mientras tanto,
desde aquí se **enlaza**:
[`telegram-coordinator/docs/reglas-de-diseno.md`](https://github.com/stalinbeltran/telegram-coordinator/blob/main/docs/reglas-de-diseno.md).
Y su evidencia sí está aquí: el
[análisis de arquitectura del 2026-08-28](../reportes/arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md),
de donde salen las 19 reglas.
