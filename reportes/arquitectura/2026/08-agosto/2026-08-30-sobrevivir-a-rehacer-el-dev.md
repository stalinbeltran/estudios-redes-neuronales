# Sobrevivir a rehacer el dev: por qué las dos aplicaciones nuevas no funcionaron a la primera

*sin # — no es un barrido · tipo `arquitectura` · [índice de reportes](../../../README.md)*

| campo | valor |
|---|---|
| **Qué era** | Repaso de las dos aplicaciones construidas el 29 y el 30 de agosto de 2026 —la **web app de `foveal-vision`** como servicio y el **entrenador en Vast**— y de los cambios de aprovisionamiento que cada una necesitaba para **sobrevivir a que se destruya el dev**. La pregunta que contesta: *¿por qué casi ninguno funcionó al primer intento?* |
| **Lanzado con** | Nada: lectura del histórico de commits **más** un inventario ejecutado sobre un dev **recién nacido**. Los comandos, abajo en § 8 |
| **Inicio / Fin (UTC)** | 2026-08-30, sesión única. **No se registró la hora**: no es un barrido y no hay reloj que reconstruir |
| **Duración** | ⚠ **no aplica** — no hay trabajo cronometrado que medir |
| **Instancias** | **0.** No se alquiló ni una máquina |
| **Coste real** | **0,00 $** — y es un dato, no un hueco: este repaso no tocó ninguna nube |
| **Dataset** | ⚠ **no aplica** — no se entrenó nada. Los datasets aparecen como *sujeto* del inventario, no como insumo |
| **Estado** | **terminado** para las dos aplicaciones. ⚠ **Deja tres cosas abiertas**, en § 7 |

⚠ **No es un barrido y por eso no tiene fila en la tabla cronológica** de
[`reportes/README.md`](../../../README.md): esa tabla es contabilidad —instancias y coste real— y
una fila de cinco «no aplica» la ensuciaría. El puntero está en § «Los que no son barridos».

**Clasificación**, con las dos preguntas mecánicas: *¿se corrió algo?* **no** (no hay reloj ni
factura) → `sintesis` o `arquitectura`; *¿cuál es el sujeto?* **el sistema** —los repos, el
aprovisionamiento y la cadena que va de un commit a una máquina nueva— → **`arquitectura`**. No es
`sintesis` porque no relee estudios ya pagados: mide el estado de una máquina.

---

## 1. Por qué esta pregunta se puede contestar hoy, y no discutir

**El dev se rehízo mientras tanto.** Las dos aplicaciones se construyeron sobre la máquina anterior
—`157.230.221.199`, la IP que cita el commit `62424f9a` al verificar contra el servidor real— y esta
máquina es otra:

```
uptime -s     ->  2026-08-30 18:16:50 UTC   (nació hace minutos)
hostname -I   ->  142.93.1.59               (otra IP: no es la misma máquina)
```

Así que no hace falta razonar sobre qué *debería* sobrevivir. Se mira. **Todo lo que este documento
llama «no sobrevivió» está comprobado en el disco de un dev que nació limpio**, no deducido de los
commits.

Y el veredicto corto es el que da título a la § 4: **el código llegó entero; el estado no llegó
casi nada**. Las dos aplicaciones están de pie y las dos están **vacías**.

---

## 2. Las dos aplicaciones, y qué necesitaba cada una para sobrevivir

| | **A · la web app** (`foveal-vision-web`) | **B · el entrenador en Vast** (`entrenar_vast.py`) |
|---|---|---|
| **Qué es** | el API + la UI de `foveal-vision` en un solo proceso y un solo puerto (`:8010`), como unidad de systemd | alquila **una** máquina en Vast, entrena allí, se trae los pesos según se escriben, cambia de máquina si se vuelve lenta y la destruye |
| **Nace en** | `foveal-vision@8b3de67b` · 2026-08-29 16:06 UTC | `foveal-vision@d918da7e` · 2026-08-30 01:11 UTC |
| **Para estar en una máquina nueva necesita** | el servicio declarado en `types/dev.json`, su descriptor `services/foveal-vision-web.json`, el venv, `npm ci` + build, un token, el puerto abierto en `ufw` | el token de Vast, **una clave SSH que las instancias acepten**, el repo del lanzador, y el repo de datos donde escribir |
| **Para servir de algo necesita además** | **fuentes** (imágenes) y **runs con `best.pt`** | un **dataset con `windows.npz`** y, para continuar, un `last.pt` |
| **Commits hasta darla por buena** | **8**, en 7 h 22 min (16:06 → 23:28) | **5**, en 1 h 58 min (00:09 → 02:07) |
| **Tests añadidos por el camino** | la suite de `foveal-vision` pasa de **276** a **309** *(el último commit no anota su número)* | de **333** a **356** |

⚠ **Y aun así.** Ochenta tests nuevos en diez horas, y la lista de la § 3 sigue siendo larga. No es
que faltaran tests: es que **ninguno de ellos puede probar lo que este documento mide**, que es qué
sale de una máquina recién creada. Eso es la § 6.

---

## 3. La cuenta de los intentos, aplicación por aplicación

No es una impresión. Cada línea es un commit que arregla algo que el anterior daba por hecho.

### A · la web app — 8 commits, 7 h 22 min

| # | commit | qué se descubrió |
|---:|---|---|
| 1 | `foveal-vision@8b3de67b` 16:06 | la app como servicio. Dentro ya: el API tiene que ir en `/api` y no en la raíz, porque `/runs`, `/sweeps`, `/studies`, `/networks` y `/recipes` son **cada uno una pantalla Y un recurso** |
| 2 | `lanzador@769739c` 16:06 | **declarar el servicio en el tipo no lo instala en la máquina que lo declara.** `update` reinicia lo que ya existe; una unidad **nueva** sólo la escribía `provision`, desde la lanzadora. Hubo que escribir `install-service`. ⚠ Y su verificación terminó en *«sólo falló el bind»*: el `:8010` lo tenía un `fv-api` lanzado a mano |
| 3 | `coordinador@bd1acb9` 16:06 | **el freno no veía el trabajo que corre DENTRO de la app**: un entrenamiento lanzado desde el navegador es un hilo de `fv.api`, no hay proceso que casar, y `cerrable.mjs` decía 🟢 con trabajo vivo |
| 4 | `foveal-vision@0ee97aab` 18:57 | `arrancar` y `parar` no miraban si la unidad existe. El subcomando al que llegas **cuando quieres levantar la app** era justo el que perdía la distinción «no instalado» / «instalado y parado». Y el `log` gritaba tres líneas de *hint* de `journalctl` sobre cinco, en la pantalla más pequeña que hay |
| 5 | `foveal-vision@72743b8d` 21:49 | la pantalla `/review`. **Dos fallos encontrados MIRANDO la página, no leyendo el código** (la nav no colapsaba: `flex-direction: row` sin `display: flex`; el contador mezclaba dos edades) y uno que reprodujo un test (`write_json_atomic` no crea padres) |
| 6 | `foveal-vision@0416c2fc` 22:26 | **una máquina recién hecha tiene CERO fuentes.** `/data/sources/` está en el `.gitignore` del repo de código y el generador sólo trae `specs.jsonl`. Sin fuente no hay una sola imagen que mirar |
| 7 | `coordinador@e193870` 22:26 | el preflight crece con ese fallo: `fuentes (A): NINGUNA` |
| 8 | `foveal-vision@62424f9a` 23:28 | visto **en el servidor real**: el selector principal eran los **runs, 859 de ellos**; de 18 datasets sólo **2** traen `windows.npz`; y dos fallos más encontrados con Playwright contra el servicio de verdad (3 s de lote obsoleto al cambiar de dataset; la nav comiéndose 50 px por `z-index`) |

### B · el entrenador en Vast — 5 commits, 1 h 58 min

*(la tabla lista 4: el quinto, `987bcdcb`, es sólo documentación)*

| # | commit | qué se descubrió |
|---:|---|---|
| 1 | `foveal-vision@d0f1a5f0` 00:09 | continuar un run. El checkpoint tenía que guardar **los tres generadores**, y el del DataLoader es el que se olvida: sin él la época 4 de una reanudación recibe el orden que recibió la 1 y **el modelo repasa lo mismo creyendo que avanza**. Lo encontró un test |
| 2 | `foveal-vision@d918da7e` 01:11 | **«los cuatro intentos que costó»**, y son cuatro causas distintas: (a) `Permission denied (publickey)` **con la clave bien registrada** — se arregló con una clave dedicada, y ⚠ *la causa de que la de DigitalOcean no valiera NO está establecida*; (b) doce minutos de reintentos ciegos porque `esperar_ssh` comprueba **el banner**, que llega antes que la clave, y porque el destino SSH se leía una sola vez; (c) **`ssh_capture` descarta `stderr`**, así que «rechaza la clave» y «aún no levanta sshd» se ven idénticos (`rc=255` y nada más); (d) la ruta del run en la máquina no se puede cablear |
| 3 | `foveal-vision@53d24b7e` 01:41 | cambiar de máquina cuando se vuelve lenta, y techo de gasto. De paso: un `·` prohibido por la regla U8.6 que **el validador no cazó porque no se volvió a correr** tras el commit que lo introdujo |
| 4 | `foveal-vision@8d39210c` 02:07 | **un `pid` de OTRA máquina.** Desde que se entrena en Vast, el `pid` que llega en `status.json` es el de la máquina alquilada; `reconcile` lo leía como «el proceso murió», desarmaba el guard `run_is_running` y **dejaba arrancar un segundo entrenamiento sobre el mismo run** — dos escritores sobre el mismo `metrics.jsonl` y el mismo `last.pt`. Y `os.replace` a un temporal en `/tmp` sólo es atómico dentro del mismo sistema de ficheros: con `/tmp` en tmpfs daría `EXDEV`, y el `except OSError` se lo tragaba |

⚠ **Los apartados (b) y (c) del intento 2 estaban resueltos desde hacía días en `estudio_flota.py`**
—`sellar` reintenta 12 veces, `resolver_destino` re-resuelve— y el propio commit lo dice: *«las dos
lecciones ya estaban pagadas y no las reutilicé»*.

---

## 4. El inventario de hoy: qué llegó a la máquina nueva y qué no

Medido el 2026-08-30 sobre este dev, nacido a las 18:16:50 UTC. Los comandos están en § 8.

| Qué | ¿Llegó? | Por qué mecanismo | |
|---|---|---|---|
| Los **6 repos**, incluido `estudios-redes-neuronales` | ✅ | `types/dev.json` → `repos` | el cambio de `lanzador@0042a17` funcionó |
| El **servicio `foveal-vision-web`** | ✅ | `types/dev.json` → `services` | **activo a las 18:23:41**, o sea 6 min 51 s después del arranque |
| El **puerto, el front construido y el venv** | ✅ | el `install` del descriptor (`web_app.py preparar`) | ⚠ y de regalo: **el venv de `foveal-vision` ya no falta en un dev nuevo**, que era un fallo con nombre propio en `CLAUDE.md` |
| Los `windows.npz` de **2** datasets | ✅ | commiteados en `foveal-vision-data` | el preflight lo confirma: `2 con windows.npz commiteado (de 18 descritos)` |
| El **token de la web app** | ⚠ **es otro** | se generó uno nuevo en `~/.config/fv-web.env` | el descriptor promete que sobrevive **si la lanzadora tiene `FVW_WEB_TOKEN` en su `.env`**; no lo tiene, así que **la URL marcada en el móvil ya no vale** |
| Los **16 datasets restantes** | ❌ | manifest sin `npz` | son descripciones de datos que se perdieron |
| Las **fuentes** (`data/sources/`) | ❌ | `.gitignore` del repo de código | `discover_sources()` → **0** |
| Los **pesos** (`best.pt` / `last.pt`) | ❌ | `*.pt` en el `.gitignore` del repo de datos | `find foveal-vision-data -name '*.pt'` → **vacío** |
| La **clave dedicada de Vast** | ❌ | no está en el aprovisionamiento | `~/.ssh/` sólo tiene `do_droplet`, que es la que fue rechazada |
| El **coste** del entrenamiento en Vast | ❌ | `entrenar_vast.py` no lo escribe en disco | lo calcula, lo imprime y lo manda a Telegram; con la máquina se fue |

### Lo que esto costó, concretamente

**El modelo `fov-optimo-p20` ya no existe.** Sobrevive su descripción entera —`config.json`,
`metrics.jsonl`, `summary.json`, en git— y no sobrevive el modelo:

| | |
|---|---|
| red | `fov16-optimo` (los dos ejes cerrados al 5 % **aplicados**) · 167.852 parámetros |
| dataset | `dirty1000-80px-16px-r20260827` (`sha256:3df67624…`) |
| entrenamiento | **69 épocas**, parado por `patience` 20 · **5.066 s de reloj de entrenamiento** (84,4 min) a 73,4 s/época, **en Vast** |
| mejor época | 49 por `val_loss` (0,09125), **f1 = 0,9430**; el mejor f1 fue **0,9461** en la 67 |
| coste real | ⚠ **no registrado.** El script lo calcula y lo imprime; ningún fichero lo guarda |
| qué queda | las curvas. **Ni un byte de pesos** |

⚠ **Ese f1 no se compara con las tablas de estudios**, y está escrito así en el commit que lo
produjo (`987bcdcb`): el criterio de parada es más alto a propósito y aquí el objetivo era un
modelo, no una fila.

⚠ **Y el hueco del coste es exactamente el que este directorio existe para evitar.**
`estudio_flota.py` aprendió a escribir `cuando`, `reloj_min`, `usd` y `maquinas_alquiladas` en
`flota.json`; `entrenar_vast.py` nació sin eso. **La lección no viajó al script nuevo**, aunque
esté escrita en la cabecera del índice.

---

## 5. Las cinco causas, indexadas por la acción que las dispara

Están escritas por **lo que estabas haciendo cuando muerden**, no por su primera víctima: una
trampa contada como historia se lee una vez, y una contada como acción se lee cada vez que toca
hacerla.

### C1 · Al declarar algo en el tipo del lanzador — llega a las máquinas que NACEN después, no a ésta

Es la causa estructural, y explica por qué *nada* de esto se puede probar donde se escribe.
`types/dev.json` describe **el nacimiento** de una máquina. Un cambio ahí es correcto, se commitea,
se empuja… y **la máquina donde lo escribiste sigue sin tenerlo**. Ya estaba avisado en
`lanzador@0042a17` (*«sólo llega a las máquinas creadas DESPUÉS, y sólo si el mini tiene este repo
al día»*) y volvió a morder inmediatamente: declarar `foveal-vision-web` obligó a escribir
`install-service` en el mismo commit, porque si no, **la única forma de estrenar el servicio era
rehacer una máquina perfectamente viva**.

**El corolario incómodo:** la verificación que se puede hacer no es la que importa. Lo que se
comprueba es *«el comando funciona aquí»*; lo que hace falta saber es *«sale bien una máquina
nueva»*, y eso cuesta un lanzamiento entero.

### C2 · Al hacer que una app «sobreviva» — sobrevive el CÓDIGO, no lo que la app necesita para servir de algo

Medido hoy: el servicio está activo, el puerto abierto, el front construido… y **0 fuentes, 2 de 18
datasets, 0 pesos**. Las dos aplicaciones llegaron enteras y vacías. La web app arranca y no tiene
una sola imagen propia que enseñar; el entrenador arranca y no tiene un `last.pt` que continuar.

Es la **regla 1 de escritura** del proyecto —*«sobrevive» siempre lleva complemento*— fallando en el
título de un commit: `d918da7e`, *«entrenar en Vast trayéndose los pesos SEGÚN SE ESCRIBEN, no al
final»*. Es verdad y es útil: protege contra que **la máquina alquilada** se caiga en la última
época. **No protege contra que se caiga el dev**, que es a donde bajan los pesos. Nadie escribió el
segundo complemento, y por eso nadie vio que faltaba el segundo mecanismo.

### C3 · Al arreglar «el dato no viaja» — el arreglo se escribe en la máquina que ya perdió el dato

`0416c2fc` (29-ago 22:26) hizo bien las tres cosas difíciles: publicar la fuente **reducida** (2,01
MB medidos), **no re-renderizar** —está medido que no da el mismo dato—, y **comparar píxel a píxel**
contra el `windows.npz` antes de publicar. Tiene tests. Tiene su excepción en el `.gitignore` del
repo de datos (`foveal-vision-data@66249db`).

**Y no se ha publicado ni una fuente.** Comprobado hoy por los dos lados:

```
foveal-vision-data/sources/          ->  no existe
git log --all -- sources             ->  vacío
```

El motivo cierra el círculo: el arreglo se escribió **en una máquina que ya tenía cero fuentes** —el
preflight de esa misma hora lo dice—, así que no había nada que publicar. Y la pista que el propio
preflight imprime hoy es la prueba: *«Publica la reducida desde **una máquina que la tenga**»*. **Ya
no hay ninguna.**

⚠ **Y la fuente de `r20260827` es irrecuperable.** Re-renderizar no devuelve el mismo dato (medido
tres veces, la última el 2026-08-27, con `bench_dataset.py build` abortando solo por huella), y
`publish_source` **aborta** si los píxeles no coinciden con el `npz` — que es precisamente el guard
que hace que esto sea un módulo y no un `cp -r`. El guard es correcto; lo que ya no se puede es
cumplirlo para ese dataset.

### C4 · Al arreglar algo a mano en la máquina — la solución que funcionó no está en el aprovisionamiento

El caso limpio es la clave de Vast. Está **bien documentada**, con los comandos exactos, en
`foveal-vision/docs/entrenar.md` § *«Antes de nada: una clave de Vast PROPIA»*: generar
`~/.ssh/vast_ed25519`, registrarla, y entrenar con `VAST_SSH_KEY_FILE` apuntando ahí. Escrita donde
se dispara, como manda la regla.

Y en esta máquina:

```
~/.ssh/            ->  do_droplet, do_droplet.pub   (nada más)
VAST_SSH_KEY_FILE  ->  por defecto ~/.ssh/do_droplet
types/dev.json     ->  post: vast_instance.py register-key --comment dev
```

O sea que el dev nuevo registra en Vast **exactamente la clave que fue rechazada**, y la dedicada no
está. ⚠ **Esto no predice un fallo** —la causa de aquel rechazo nunca se estableció, y podría ser de
las instancias concretas—, pero sí dice algo seguro: **la configuración con la que funcionó no se ha
reproducido**, y si vuelve a fallar habrá que volver a descubrirlo.

Mismo patrón, más barato, con `FVW_WEB_TOKEN`: el descriptor explica que el token sobrevive si la
lanzadora lo tiene en su `.env`. No lo tiene. El token de hoy es nuevo.

**Documentado ≠ aprovisionado.** Es la regla que el propio lanzador ya tiene escrita —*«si para que
algo esté hay que acordarse de un `--repo`, tarde o temprano no está»*— aplicada a un paso manual de
un documento, que es un sitio donde nadie la buscó.

### C5 · Al leer un fallo — un silencio se lee como un cero

Cinco casos de las últimas veinticuatro horas, y son el mismo:

| dónde | «no pude mirar» se veía igual que… |
|---|---|
| `V.ssh_capture` descartando `stderr` | …«miré y la clave está mal». **Doce minutos** de reintentos ciegos |
| `except OSError` sobre `os.replace` entre sistemas de ficheros | …«se colocó bien». La descarga fallaría **en silencio para siempre** |
| `discover_sources()` devolviendo 0 | …«esta máquina no tiene fuentes, y es normal». No había forma de saberlo sin ir a buscarlo |
| un `pid` de otra máquina en `reconcile` | …«el proceso murió»: **dueño desconocido leído como dueño muerto**, y el guard se desarma |
| `StaticFiles` **lanzando** el 404 en vez de devolverlo | …una ruta de la SPA. Lo encontró un test |

El proyecto ya tiene la regla —*entre un fallo ruidoso y uno silencioso, el ruidoso*— y la aplica en
`cerrable.mjs` (`🟡 NO SÉ`) y en `workspace.mjs`. **Lo que no tiene es la costumbre de aplicarla al
escribir el código nuevo**, que es donde nace cada uno de estos cinco.

### Y una que no es causa sino su síntoma: se verifica leyendo, o en la máquina que ya tenía el estado

Cuatro veces en dos días: los dos fallos de `/review` que sólo aparecen **mirando la página**; los
dos más que aparecieron **con Playwright contra el servidor real** (y con ellos los 859 runs del
selector, que en una máquina de pruebas habrían sido tres); el `·` que el validador no cazó **porque
no se volvió a correr**; y —fuera de estas dos apps, la misma semana— el test del triage que había
elegido sin querer *la única redacción que pasaba*, de modo que el ejemplo del documento era falso
desde el primer día y sólo se supo **ejecutando el comando que el documento promete**.

---

## 6. La causa de fondo: no hay forma de probar «¿sale bien una máquina nueva?» sin hacer una máquina nueva

Las cinco causas de arriba son cinco caras de un mismo agujero. El sistema tiene tests —**359** recogidos en `foveal-vision` (356 pasan, 3 saltados) y **125** en el
coordinador, los dos contados hoy—, tiene validadores de UI, tiene un preflight y tiene
frenos. **Ninguno de ellos puede correr en la única máquina que interesa**, porque esa máquina
todavía no existe cuando se escribe el cambio.

Lo que hay hoy es un **sustituto**: `bench-preflight.mjs`, que *«comprueba estado utilizable, no
presencia»* y *«crece con cada fallo»*. Es la pieza correcta y funciona —hoy mismo ha cazado dos
cosas—:

```
[ ok  ] venv de foveal-vision      torch 2.13.0+cpu · fv importable
[ ok  ] datasets de estudio        2 con windows.npz commiteado (de 18 descritos)
[aviso] fuentes (A)                NINGUNA: no se puede revisar ni medir la métrica de tarea
```

⚠ **Pero un preflight que crece con cada fallo va, por construcción, un fallo por detrás.** Cada
línea suya es un incidente ya pagado. La línea de `fuentes` se escribió a las 22:26 del 29 y **está
avisando hoy de que el arreglo no llegó**: hace su trabajo, y su trabajo es contar la pérdida, no
evitarla.

Lo que cerraría el agujero no es más disciplina, son dos cosas concretas y ninguna es cara:

1. **Que el preflight pregunte por lo que las dos aplicaciones nuevas necesitan** —pesos, clave de
   Vast, token persistente—, y no sólo por lo que necesitaba el benchmark de vCPU. Es una línea por
   cosa, en el mismo commit que el arreglo, que es la regla que ya está escrita.
2. **Que lo que no se puede re-derivar viaje o se declare perdido de antemano.** Hoy hay tres
   categorías mezcladas: lo que viaja por git (código, `windows.npz`, curvas), lo que se regenera a
   un coste conocido (el venv, el build) y **lo que simplemente se pierde** (pesos, fuentes, la
   factura de una corrida). La tercera no está declarada en ningún sitio, y por eso sorprende cada
   vez.

---

## 7. Lo que quedó pendiente

Tres cosas abiertas, en orden de lo que cuesta perderlas otra vez.

1. **Los pesos no viajan y no está decidido si deben.** `*.pt` está en el `.gitignore` del repo de
   datos, y hoy se ha comprobado que eso significa que **un modelo entrenado y pagado desaparece con
   el dev**. `best.pt` son 680.793 B y `last.pt` 2.050.697 B (medidos en `d0f1a5f0`), así que el
   tamaño no es el argumento. Las salidas son declarar el modelo *regenerable* —y entonces registrar
   su coste, que hoy tampoco se guarda— o hacerlo viajar. **No hacer ninguna de las dos es la opción
   que se ha elegido sin elegirla.**
2. **La fuente de `r20260827` es irrecuperable, y `publish_source` no se ha usado nunca.** El
   mecanismo está construido y probado; lo que falta es una máquina que tenga una fuente. Mientras,
   `/review` sirve los píxeles desde el propio `windows.npz` —son verbatim, y hay test que lo
   compara—, así que **mirar imágenes sí funciona**; lo que no se puede es puntuar la métrica de
   tarea ni revisar nada fuera del `npz`.
3. **La clave dedicada de Vast no está en el aprovisionamiento.** Un dev nuevo registra
   `~/.ssh/do_droplet`. Cabe en el `post` de `types/dev.json`, al lado del `register-key` que ya
   está. ⚠ Y como la causa del rechazo original nunca se estableció, ponerla es barato y no ponerla
   sólo se nota cuando ya hay una máquina alquilada facturando.

⚠ **Y una cuarta que no es de este repaso pero sale de él:** `entrenar_vast.py` no escribe el coste
de la corrida en ningún fichero. `estudio_flota.py` sí (`flota.json`). Mientras no lo haga, cada
entrenamiento en Vast deja el mismo hueco que la cabecera de este índice lleva señalando desde el
#1.

---

## 8. De dónde salen los números

Todo lo de la § 4 se ejecutó el 2026-08-30 sobre este dev (`142.93.1.59`, nacido a las 18:16:50 UTC):

```sh
uptime -s; hostname -I                       # la máquina es otra que la de los commits
systemctl show foveal-vision-web -p ActiveEnterTimestamp
cd ~/src/foveal-vision && .venv/bin/python scripts/web_app.py estado
.venv/bin/python -c "import sys;sys.path.insert(0,'src');\
 from fv.datasets.loader import discover_sources; print(len(discover_sources()))"      # -> 0
ls ~/src/foveal-vision-data/window-datasets/*/windows.npz | wc -l                      # -> 2 (de 18)
find ~/src/foveal-vision-data -name '*.pt'                                             # -> vacío
git -C ~/src/foveal-vision-data log --all -- sources                                   # -> vacío
ls ~/.ssh/                                                                             # -> sólo do_droplet
node ~/src/telegram-coordinator/scripts/bench-preflight.mjs
```

Y el run perdido, leído de `foveal-vision-data/2026/08-agosto/runs/fov-optimo-p20/`
(`summary.json`, `metrics.jsonl`, `config.json`), que es lo único que queda de él.

El histórico de la § 3 sale de los mensajes de commit, que en este proyecto llevan escrito lo que
costó cada cosa:

```sh
git -C ~/src/foveal-vision log --since=2026-08-29T15:00 --until=2026-08-30T03:00 --pretty='%h %ad %s%n%b'
git -C ~/src/digital-ocean-dropplet-auto-launching log --since=2026-08-29 --pretty='%h %ad %s%n%b'
git -C ~/src/telegram-coordinator log --since=2026-08-29 --pretty='%h %ad %s%n%b'
```

---

## 9. Lo que este documento NO establece

- **No dice que el entrenamiento en Vast vaya a fallar en esta máquina.** Dice que la clave con la
  que funcionó no está aquí, y que la causa del rechazo original nunca se estableció.
- **No mide cuánto costó `fov-optimo-p20`.** Ese número no lo guardó nadie; sólo consta el reloj de
  entrenamiento (5.066 s) que se puede sumar de `metrics.jsonl`.
- **No revisa el código de las dos aplicaciones.** El sujeto es la cadena que va de un commit a una
  máquina nueva, no la calidad de lo que corre encima.
- **No comprueba el camino entero.** Lo verificado es el estado de **una** máquina recién nacida. Si
  el arreglo de la § 7 funciona, se sabrá en la siguiente — que es exactamente el problema que
  describe la § 6.
