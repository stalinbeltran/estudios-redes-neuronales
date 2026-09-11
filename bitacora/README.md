# Bitácora — qué se pidió y qué se descubrió, día a día

Aquí queda **lo que se pide y lo que se averigua, según ocurre**, venga del repo que venga. Una
fila por cosa. Se escribe **en el momento**, no al cerrar el trabajo.

⚠ **SÓLO SE AÑADE. Una fila escrita no se toca nunca más.** Ni para corregirla, ni para pulir la
redacción, ni para tacharla cuando resulta que estaba equivocada. Si una fila vieja resultó falsa,
se escribe una fila **nueva** que lo diga y enlace a la anterior. Ése es justamente el valor de
esto: se ve lo que se creyó en cada momento, no sólo lo que acabó siendo verdad.

El motivo no es de estilo. El 2026-09-11 costó una mañana entera reconstruir qué se había pedido
la tarde anterior y con qué pruebas se había concluido cada cosa, porque lo único que quedaba eran
los commits (que cuentan el resultado, no el camino) y un log de conversación que nadie recordaba
dónde estaba. Un registro que se reescribe no habría servido: lo que hacía falta era **la creencia
equivocada del momento**, no la conclusión final.

## Qué va aquí, y qué no

| | |
|---|---|
| **Aquí** | lo que el usuario pide, los hallazgos según aparecen, las creencias que luego se corrigen, las decisiones y su porqué. De **todos** los repos |
| [`../ESTADO.md`](../ESTADO.md) | *qué es verdad hoy*. **Se reescribe**. Un veredicto por parámetro |
| [`../reportes/`](../reportes/) | *qué se corrió, cuándo y qué costó*. Un documento por estudio o medición terminada, con su fila en el índice |

La diferencia con `reportes/` es el **grano y el momento**: allí va un trabajo terminado, con su
reloj y su factura; aquí va cada petición y cada hallazgo suelto, según pasa, aunque no lleve a
ningún estudio. Un hallazgo de la bitácora que acaba siendo un estudio se enlaza desde su fila; no
se mueve ni se copia.

Sigue valiendo el charter del repo: **prosa y veredictos, ni una línea de código ni un byte de dato
crudo**. Lo que produzca un hallazgo vive en el repo que lo produjo, y aquí se enlaza.

## Dónde van las filas

Un fichero por mes, para que esto no acabe siendo un solo documento imposible de abrir:

```
bitacora/<año>/<mes>.md      p. ej. bitacora/2026/09-septiembre.md
```

El mes lleva **número delante** (`09-septiembre`, no `Septiembre`) por lo mismo que en
[`reportes/`](../reportes/README.md): así `ls` y el árbol de GitHub salen en orden cronológico y no
alfabético, que con nombres de mes deja `Agosto` antes que `Julio`.

Abrir un mes nuevo es **añadir** un fichero y **añadir** su fila a la tabla de aquí abajo. Nada se
mueve nunca.

| Mes | Fichero |
|---|---|
| septiembre de 2026 | [`2026/09-septiembre.md`](2026/09-septiembre.md) |
