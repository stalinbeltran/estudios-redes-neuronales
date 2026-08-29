# La centralización de los reportes — 2026-08-29

> ## ⚠ ESTADO: A MEDIAS, Y A PROPÓSITO
>
> Este repo **existe sólo en local**: el token de GitHub de esta máquina **no tiene permiso para
> crear repositorios**, así que no hay remoto al que empujar. Mientras eso siga así, **los 21
> reportes siguen también en sus repos de origen** y hay dos copias de cada uno.
>
> Sí, eso es exactamente lo que la R19 llama *«una migración a medias son dos sistemas»*. Se acepta
> a sabiendas porque la alternativa es peor: **estos servidores son efímeros y lo que no está
> empujado no existe**. Borrar los originales —que sí están empujados— para dejar la única copia en
> un repo local sin remoto no es migrar, es **borrar 21 reportes**.
>
> La regla que se respeta por encima de la R19: **nada se saca de un repo empujado para meterlo en
> uno que no lo está.**

## Qué se hizo, y qué falta

| | |
|---|---|
| ✅ | Repo `estudios-redes-neuronales` construido en `~/src/estudios-redes-neuronales`, con historia de git propia |
| ✅ | Los **21** reportes clasificados en `reportes/<tipo>/<año>/<mes>/` |
| ✅ | Los enlaces internos rotos por el movimiento, reescritos y **verificados uno a uno** |
| ✅ | El `#` de cada reporte, escrito **dentro** del propio fichero (antes sólo existía en el índice) |
| ✅ | El README índice partido en `ESTADO.md` (se reescribe) + `reportes/README.md` (sólo se añade) |
| ✅ | Los dos reportes duplicados del mismo evento (`#15` y `stride-validacion`), marcados el uno desde el otro |
| ✅ | `telegram-coordinator/scripts/cerrable.mjs` cuenta el repo nuevo — el freno, en el mismo commit |
| ⏳ | **Crear el remoto en GitHub** ← el bloqueo, y no se puede resolver desde esta máquina |
| ⏳ | Borrar los originales y dejar un puntero en su lugar |
| ⏳ | Reapuntar los ~40 enlaces de los otros repos |
| ⏳ | `workspace.mjs`, `bench-preflight.mjs` y `types/dev.json` |
| ⏳ | Mover `docs/reglas-de-diseno.md` desde `telegram-coordinator` |

## El bloqueo, con su evidencia

```
$ gh repo create estudios-redes-neuronales --public
GraphQL: Resource not accessible by personal access token (createRepository)

$ gh api -X POST /user/repos -f name=estudios-redes-neuronales
{"message":"Resource not accessible by personal access token","status":"403"}
```

Los **dos** tokens de la máquina (el de `GH_TOKEN` y el de `~/.config/gh/hosts.yml`) son PAT
*fine-grained* y los dos dan 403, por GraphQL y por REST. Autenticar y leer sí pueden; crear, no.

**Se arregla de una de estas tres formas, y cualquiera vale:**

1. **Crearlo a mano** en <https://github.com/new> · nombre `estudios-redes-neuronales` · **público**
   (como sus hermanos; los droplets clonan con `https://` sin token) · **sin** README, `.gitignore`
   ni licencia — el repo local ya los trae y un commit inicial ajeno obligaría a un merge.
2. **Dar permiso al token** en <https://github.com/settings/personal-access-tokens>: el token →
   *Repository permissions* → **Administration: Read and write**, con el alcance en *All repositories*.
   Es lo que pide `POST /user/repos` para un PAT fine-grained.
3. **Un PAT clásico** con el scope `repo`.

## Cómo se termina, una vez exista el remoto

Los tres pasos van **seguidos y el mismo día**. Si no caben seguidos, no se empiezan: es
literalmente la R19.

### Paso 1 — empujar (después de esto, el trabajo ya existe)

```sh
cd ~/src/estudios-redes-neuronales
git remote add origin https://github.com/stalinbeltran/estudios-redes-neuronales.git
git push -u origin main
```

⚠ **Hasta aquí no se borra nada.** Éste es el único paso que cambia el mundo: a partir de él hay
dos copias *empujadas*, que es un estado feo pero seguro. Los siguientes lo limpian.

### Paso 2 — vaciar los orígenes y dejar un puntero

```sh
cd ~/src/telegram-coordinator
git rm -r -q reportes/2026
cat > reportes/README.md <<'FIN'
# Los reportes se mudaron

Viven en **[`estudios-redes-neuronales`](https://github.com/stalinbeltran/estudios-redes-neuronales)**,
clasificados por tipo: `reportes/<tipo>/<año>/<mes>/`.

- El índice cronológico, con instancias y coste real: [`reportes/README.md`](https://github.com/stalinbeltran/estudios-redes-neuronales/blob/main/reportes/README.md)
- En qué quedó cada parámetro: [`ESTADO.md`](https://github.com/stalinbeltran/estudios-redes-neuronales/blob/main/ESTADO.md)

Se movieron el 2026-08-29 porque cambian 83× más despacio que el repo que mide y los escribe otra
mano; y porque 4 de los 21 no son de ningún repo en concreto. El motivo largo, en el
[README del repo central](https://github.com/stalinbeltran/estudios-redes-neuronales#1-excepción-a-la-r7--un-artefacto-vive-en-la-pieza-de-quien-lo-produce).
FIN
git add reportes/README.md

cd ~/src/foveal-vision
git rm -r -q reportes/2026/08-agosto/*.md          # ⚠ el .json de datos NO: se queda
mkdir -p reportes && cat > reportes/README.md <<'FIN'
# Los reportes se mudaron

Los `.md` viven en **[`estudios-redes-neuronales`](https://github.com/stalinbeltran/estudios-redes-neuronales)**.
Aquí sólo se queda el **dato crudo** que producen (`2026/08-agosto/datos/*.json`): el reporte se
lleva la prosa, el dato se queda con quien lo produjo.

El criterio de cada estudio —escrito **antes** de mirar— sigue donde estaba: `docs/plan-*.md`.
FIN
git add reportes/README.md
```

### Paso 3 — las referencias, todas de una vez

**Las cuatro listas de repos**, que hoy están cableadas a cinco y a las que hay que añadir
`estudios-redes-neuronales`:

| Fichero | Si no se toca |
|---|---|
| `telegram-coordinator/scripts/cerrable.mjs:86` | ✅ **ya hecho** el 2026-08-29 |
| `telegram-coordinator/scripts/workspace.mjs:41` | ningún workspace nuevo trae el repo → nadie puede comprobar si un estudio ya se pagó. ⚠ **No se tocó antes porque `--nuevo` CLONA**: con el remoto inexistente, `git clone` falla y **aborta el montaje entero** |
| `telegram-coordinator/scripts/bench-preflight.mjs:195` | `--fix` no lo clona en una máquina nueva. ⚠ Mismo motivo para no tocarlo antes: clona, y un repo que no existe dejaría el preflight en fallo permanente — bloqueando el estudio `do-v`, que está pendiente |
| `digital-ocean-dropplet-auto-launching/types/dev.json` (`"repos"`) | **un `dev` recién lanzado nace sin la tabla de veredictos.** Y después, `actualizar` en el Lanzador, o el mini sigue leyendo el tipo viejo |

**Los enlaces en prosa**, localizados con
`grep -rn "reportes/" --include='*.md' --include='*.py' --include='*.mjs' ~/src`:

- `telegram-coordinator/CLAUDE.md` — líneas 168, 502-517, 584, 793, 799, 841, 1101
- `telegram-coordinator/docs/reglas-de-diseno.md` — 27 y 370
- `telegram-coordinator/.claude/agents/revisor.md` — 19 y 22. ⚠ **No es un enlace, es una
  capacidad**: ese agente tiene instrucción literal de mirar el índice para ver si una petición
  **repite trabajo ya pagado**. Sin él debe **decirlo en voz alta**, no callar (R2). Un revisor que
  no encuentra el índice y se calla aprueba pagar dos veces un barrido
- `foveal-vision/CLAUDE.md` — 70, 127, 918, 1294, 1376-1377. ⚠ Las 918 y 1376 **no son enlaces: son
  la regla de dónde va un reporte**. Se reescriben, no se repuntan
- `foveal-vision/docs/plan-prioridades-2026-08-25.md:7`, `plan-dropout-2026-08-28.md:28,40`,
  `plan-cierre-2026-08-26.md:6,10`
- `foveal-vision/scripts/knobs_f.py:31`, `scripts/vigilante_avance.py:559`,
  `scripts/vigilante_prioridades.py:10` — rutas y mensajes al usuario **dentro de código**
- `foveal-vision-data/README.md:38,122`

Y **`telegram-coordinator/docs/reglas-de-diseno.md` se mueve a `docs/` de este repo** en este mismo
paso: gobierna los cinco repos, y el propio documento dice de sí mismo *«el ejemplo es de aquí, la
regla no»*.

### Paso 4 — comprobar

```sh
# ningún reporte vive en dos sitios (esto es la R19)
find ~/src/telegram-coordinator/reportes ~/src/foveal-vision/reportes -name '*.md' | wc -l   # -> 2

# cero rutas viejas vivas
grep -rn "reportes/2026/" ~/src --include='*.md' --include='*.py' --include='*.mjs' \
  | grep -v estudios-redes-neuronales | grep -v "/datos/"                                        # -> VACÍO

# el freno cuenta el repo nuevo (el que cuesta dinero si falla)
touch ~/src/estudios-redes-neuronales/BORRAME.md
node ~/src/telegram-coordinator/scripts/cerrable.mjs      # -> 🔴 y NOMBRA estudios-redes-neuronales
rm ~/src/estudios-redes-neuronales/BORRAME.md

# un workspace nuevo trae seis repos
node ~/src/telegram-coordinator/scripts/workspace.mjs --nuevo prueba-central && ls ~/ws/prueba-central   # -> 6
```

## Lo que se decidió NO hacer, y por qué

- **Copiar los reportes y dejar los originales como estado final.** Dos mitades desfasadas,
  garantizado. Lo de hoy es un estado *de tránsito* declarado y con su fecha, no un diseño.
- **Renumerar el `#` al reclasificar.** El `#` ya se cita desde otros repos; un número reasignado
  sigue resolviendo a *un* reporte, sólo que al equivocado — y eso es peor que uno que falta.
- **Fusionar el `#15` con `stride-validacion`**, que son el mismo evento. Fundir dos textos es
  perder el encuadre de uno de los dos. Se marcan el uno desde el otro y ya no son invisibles.
- **Submódulos de git para «referenciar los otros repos».** Un submódulo fija *un commit* y
  `foveal-vision` recibe ~100 commits de máquina al día: el pin nace rancio. Referenciar no es
  anidar; se hace con la tabla declarada del `README.md`.
- **Aprovechar para partir `foveal-vision/CLAUDE.md`**, que también le hace falta. «Ya que estamos»
  es como nacen las migraciones a medias.
