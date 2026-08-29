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

## Candidatos declarados, y por qué NO se han movido aún

**`telegram-coordinator/docs/reglas-de-diseno.md`** — las 19 reglas de diseño. Pasan la prueba de
las dos preguntas sin discusión: gobiernan los cinco repos y el propio documento dice de sí mismo
*«el ejemplo es de aquí, la regla no»*. Su sitio es éste.

**No se movió en la centralización del 2026-08-29 a propósito.** Ese día el repo central no se pudo
crear en GitHub (el token no tiene permiso), así que sólo existía en local — y en un servidor
efímero, mover un documento desde un repo empujado a uno sin remoto es **borrarlo**. Se mueve
cuando el remoto exista, en el mismo commit que lo demás; está anotado en
[`migracion-2026-08-29.md`](migracion-2026-08-29.md).

⚠ Vale la regla general, y es la que hay que respetar aquí: **nada se saca de un repo empujado para
meterlo en uno que no lo está.**
