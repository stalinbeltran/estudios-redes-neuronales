# CLAUDE.md — el repo CENTRAL

**Este repo es el central del sistema**: aquí vive lo que **no es de ningún repo en concreto** —
los reportes de todos los estudios, el estado de los parámetros y la referencia de qué pieza hace
qué. Los otros cinco repos apuntan aquí; desde aquí no se depende de ninguno.

⚠ **Este fichero NO repite el [`README.md`](README.md).** Allí están el charter entero, la tabla de
los cinco repos, las dos excepciones declaradas y las comprobaciones. Aquí sólo va lo que una
sesión de Claude **puede hacer mal en los primeros cinco minutos** si nadie se lo dice.

---

## 1. ⚠ El charter: prosa y veredictos. NADA de código ni de dato

> Ni una línea de código, ni un byte de dato crudo, ni un `.json` de resultados, ni un peso.
> El dato vive donde lo dejó quien lo produjo; aquí se **resume y se enlaza**.

No es estilo: es la razón de existir del repo. Uno llamado «central» es exactamente el que acaba
siendo *el sitio donde va lo que no se sabe dónde poner*, y entonces deja de servir para lo único
que hace — poder mirar en un sitio qué se pagó ya y qué quedó decidido.

```sh
find . -path ./.git -prune -o \( -name '*.py' -o -name '*.json' -o -name '*.npz' -o -name '*.pt' \) -print
#  -> tiene que salir VACÍO
```

Si necesitas guardar un número, una tabla o un artefacto, **va al repo que lo produjo** y desde aquí
se enlaza. Si no sabes cuál lo produjo, ésa es la señal de que hay que decidirlo, no de que vaya
aquí.

## 2. Los dos ficheros tienen reglas de edición OPUESTAS (R8)

| | |
|---|---|
| [`ESTADO.md`](ESTADO.md) | *qué es verdad hoy*. **Se REESCRIBE.** Una fila por parámetro, con su veredicto y su reporte |
| [`reportes/README.md`](reportes/README.md) | *qué pasó*. **SÓLO SE AÑADE**, en orden cronológico. Reescribir una fila vieja es perder el histórico que lo hace útil |

Mezclarlos obliga a leerlo todo y ordenar por fecha para saber el presente — que es exactamente de
lo que se salió al crear este repo.

## 3. Un estudio no está cerrado hasta que su reporte está aquí

```
reportes/<tipo>/<año>/<mes>/<fecha>-<nombre>.md      + su fila al final de reportes/README.md
```

El **tipo** se elige con dos preguntas mecánicas, no por gusto: *¿se corrió algo (hay reloj y
factura)?* y *¿cuál es el sujeto medido?* — `estudios` si es **la red**, `infraestructura` si es
**la máquina o la cadena**, `sintesis` si **no se midió nada** y se relee lo ya pagado,
`arquitectura` si el sujeto es **el sistema de repos y procesos**.

⚠ **Quién lanza no clasifica.** El detalle, los campos obligatorios (inicio y fin en UTC,
instancias **alquiladas**, coste real, «lo que quedó pendiente») y por qué esos y no otros, en
[`reportes/README.md`](reportes/README.md).

## 4. ⚠ Aquí se trabaja en `main`, y es una excepción declarada

La convención del proyecto es rama propia por workspace. **Aquí no**: un índice sólo-añadir no
diverge de verdad, y un veredicto parado en una rama es invisible para la máquina siguiente — el
fallo que este proyecto ya pagó el 2026-08-14. El motivo entero, en el README.

## 5. Lo que NO está aquí, y dónde está

| Si tu duda es… | Vive en |
|---|---|
| cómo se **diseña** (las 19 reglas, por disparador) | [`telegram-coordinator/docs/reglas-de-diseno.md`](https://github.com/stalinbeltran/telegram-coordinator/blob/main/docs/reglas-de-diseno.md) |
| cómo se **escribe** (procedencia de los números, «sobrevive» con complemento) | [`telegram-coordinator/CLAUDE.md`](https://github.com/stalinbeltran/telegram-coordinator/blob/main/CLAUDE.md) |
| cómo se **entrena**, y el criterio de cada estudio | [`foveal-vision/CLAUDE.md`](https://github.com/stalinbeltran/foveal-vision/blob/main/CLAUDE.md) y sus `docs/plan-*.md` |
| **si se puede apagar el server** | `node ~/src/telegram-coordinator/scripts/cerrable.mjs` |

⚠ **Y esas reglas aplican aunque estés trabajando aquí.** Este repo no tiene reglas propias de
diseño ni de redacción: tiene un charter. Lo demás se hereda, y se enlaza en vez de copiarse —
copiarlo es como nacen las dos mitades desfasadas.
