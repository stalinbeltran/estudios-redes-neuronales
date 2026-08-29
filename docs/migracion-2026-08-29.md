# La centralización de los reportes — 2026-08-29

> ## ✅ COMPLETADA el 2026-08-29
>
> Los 21 reportes viven aquí y **en ningún otro sitio**. Los dos repos de origen conservan un
> puntero y nada más; `foveal-vision` se queda además con el **dato crudo** que produjo
> (`reportes/2026/08-agosto/datos/*.json`), que es de quien lo produjo y no del reporte.

Se conserva porque explica **por qué** las cosas están donde están, y porque la siguiente
centralización —la habrá— puede leerse aquí en vez de volver a razonarla.

## Qué se movió, y de dónde

| Origen | Cuántos | Ahora |
|---|---:|---|
| `telegram-coordinator/reportes/2026/08-agosto/` | 18 | clasificados en `reportes/<tipo>/2026/08-agosto/` |
| `foveal-vision/reportes/2026/08-agosto/` | 3 | íd. |

```
reportes/estudios/         14   el sujeto medido es LA RED
reportes/infraestructura/   4   el sujeto es LA MÁQUINA o LA CADENA
reportes/sintesis/          2   no se midió nada: se relee lo ya pagado
reportes/arquitectura/      1   el sujeto es el SISTEMA de repos y procesos
```

## Por qué salieron de donde estaban

**`telegram-coordinator` es el transporte**: dispara los estudios, no los produce. Guardar allí sus
reportes era el incumplimiento de la R7 que el propio
[análisis de arquitectura](../reportes/arquitectura/2026/08-agosto/2026-08-28-analisis-arquitectura.md)
señaló.

**Y `foveal-vision` tampoco era la respuesta**, aunque la R7 literal lo pidiera:

1. **R1 pesa más.** En agosto de 2026 `foveal-vision` recibió **2.420** commits y los reportes
   **29** — 83× — y el 92 % de los suyos los escribe una **máquina** a mitad de una flota, no una
   persona al terminar un estudio. Son dos relojes y dos autores.
2. **R7 no sabía colocar 4 de los 21.** El `#1` mide droplets del lanzador con un script de
   `foveal-vision`; el `#2` se lanza con `vast_instance.py` (lanzador) envuelto en `vast-sweep.sh`
   (coordinador); el análisis de arquitectura tiene por sujeto los cinco repos y por productor a
   ninguno. Una regla ambigua para el 19 % de los casos no puede ser el único criterio.
3. **La mitad del encargo no cabía en ningún repo**: la información descriptiva del proyecto.

⚠ **Lo que esto NO arregla**, y conviene no creérselo: el ciclo de dependencias **de código** entre
repos sigue **intacto** — esto no tocó un solo `import` ni un solo `ROOT.parent`. Lo que desapareció
es la arista rara de la **documentación**: hasta ahora el repo que mide dependía documentalmente
del repo del transporte.

## Lo que se aprovechó, porque mover era el único momento barato

- **El README índice era estado E historial en el mismo fichero** (R8). Partido en
  [`ESTADO.md`](../ESTADO.md), que **se reescribe**, y [`reportes/README.md`](../reportes/README.md),
  que **sólo se añade**. Mezclados, la fila de `patience` corregida el 28-ago convivía con su
  versión vieja en el mismo sitio.
- **El `#` de cada reporte sólo existía en la tabla del índice** (R9) — y se cita desde otros repos
  (`foveal-vision/docs/plan-dropout-2026-08-28.md` enlaza al `#14`). Ahora va **dentro** del
  fichero, en su segunda línea. ⚠ **No se reasigna nunca**: un número reapuntado sigue resolviendo
  a *un* reporte, sólo que al equivocado.
- **El `#15` y `stride-validacion` son el MISMO evento** —mismas 9 instancias, mismos 0,0383 $,
  mismos 37,1 min— escrito desde los dos repos el mismo día. Eran las «dos mitades» que la regla
  del proyecto teme, y **eran invisibles porque vivían en repos distintos**. Al quedar uno al lado
  del otro deja de serlo. Se marcan mutuamente y **no se fusionan**: fundir dos textos es perder el
  encuadre de uno de los dos.

## Lo que se tocó en el mismo día, y por qué no podía esperar

**Las cuatro listas de repos**, que estaban cableadas a cinco:

| Fichero | Qué pasaba si no |
|---|---|
| `telegram-coordinator/scripts/cerrable.mjs` | **el freno daba falso verde** con la tabla de veredictos sin empujar → permiso para destruir el server (R11) |
| `telegram-coordinator/scripts/workspace.mjs` | ningún workspace nuevo traía el repo → el revisor ciego en todos ellos |
| `telegram-coordinator/scripts/bench-preflight.mjs` | `--fix` no lo clonaba en una máquina nueva |
| `digital-ocean-dropplet-auto-launching/types/dev.json` | **un `dev` recién lanzado nacía sin la tabla de veredictos**. ⚠ Y hace falta `actualizar` en el Lanzador, o el mini sigue leyendo el tipo viejo |

**Y `.claude/agents/revisor.md`**, que no es un enlace sino **una capacidad**: ese agente tiene
instrucción de mirar el índice para ver si una petición **repite trabajo ya pagado**. Ahora, si el
repo central no está clonado, tiene que **decirlo en voz alta y bajar a RESERVAS** (R2). Un barrido
repetido no falla: cuesta lo mismo que el primero y sale igual de bien, así que nadie se entera.

## Las dos cosas que se decidieron NO hacer

1. **Mover `telegram-coordinator/docs/reglas-de-diseno.md`.** Estaba en el plan y se descartó al
   comprobar que lo leen por **ruta relativa** un hook que dispara en cada mensaje
   (`scripts/triage.mjs`) y dos definiciones de agente. El motivo largo, en
   [`README.md` de este directorio](README.md).
2. **Aprovechar para partir `foveal-vision/CLAUDE.md`** en estado + bitácora, que también le hace
   falta (recomendación #5 del análisis). «Ya que estamos» es como nacen las migraciones a medias.

## El bloqueo que hubo, para la próxima vez

El repo **no se pudo crear desde la máquina**: los dos tokens son PAT *fine-grained* y los dos dan
403 en `createRepository`, por GraphQL y por REST.

```
gh repo create …   → GraphQL: Resource not accessible by personal access token (createRepository)
POST /user/repos   → 403 {"message":"Resource not accessible by personal access token"}
```

Lo creó el usuario a mano. Si hiciera falta poder crearlos desde aquí: el token necesita
*Repository permissions* → **Administration: Read and write** con alcance *All repositories*
(<https://github.com/settings/personal-access-tokens>), o un PAT clásico con scope `repo`.

⚠ **Y la regla que salió de ahí, que vale para cualquier migración futura:** mientras el destino no
tenga remoto, **nada se saca de un repo empujado para meterlo en uno que no lo está**. Estos
servidores son efímeros; hacerlo no habría sido migrar, habría sido borrar 21 reportes. Durante las
horas que duró el bloqueo hubo dos copias, declaradas y con banner en los dos sitios — feo, pero
seguro, y es el estado de tránsito correcto.
