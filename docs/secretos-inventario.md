# Inventario de secretos y variables: todos los proyectos, en una tabla

**Fecha del barrido:** 2026-09-10. **Alcance:** los 62 repos de la cuenta
`stalinbeltran`, los 20 clones locales de `c:\Desarrollo` y el droplet `mini` vivo.
**Compañero:** [`secretos-desde-cero.md`](secretos-desde-cero.md) — cómo conseguir cada
uno si se pierde.

> ⚠ **Aquí no hay ni un valor.** Sólo nombres, para qué sirve cada uno y de dónde sale.
> Un inventario con valores es una filtración con índice.

---

## 0. Por qué este documento está en el repo CENTRAL

Extiende el charter a propósito y se declara, en vez de romperlo en silencio: hasta hoy
este repo gobernaba **los seis repos del sistema de redes neuronales**; esta tabla
gobierna **todos los proyectos**, incluidos los que no tienen nada que ver con redes
(`comercial-*`, `dashboard-sagj`). Pasa las dos preguntas de admisión de
[`docs/README.md`](README.md):

1. *¿Gobierna más de un repo?* — Gobierna todos.
2. *¿Sigue siendo cierto si se reescribe cualquier repo?* — El procedimiento para
   recuperar un token de DigitalOcean no depende de ningún repo.

Sigue sin haber aquí ni código ni datos: es prosa y una tabla.

---

## 1. La distinción que ordena todo: secreto contra configuración

De los **~90 nombres distintos** que hay repartidos por los `.env`, la inmensa mayoría
**no son secretos**: son puertos, rutas, fechas y modos. Se pueden volver a escribir
mirando el `.env.example`. Lo que hace falta proteger y saber recuperar es mucho menos.

| | qué es | si se pierde | cuántos hay |
|---|---|---|---|
| **Secreto** | prueba que eres tú ante un tercero | hay que ir al proveedor y **emitir otro** | **13** |
| **Identificador** | no abre nada, pero sin él la cosa no funciona | se vuelve a mirar dónde estaba | 3 |
| **Configuración** | puertos, rutas, modos, tamaños | se copia del `.env.example` | el resto |

**Sólo los 13 de la primera fila justifican un plan contra el peor caso.** Están todos en
[`secretos-desde-cero.md`](secretos-desde-cero.md), en el orden en que hay que
recuperarlos.

---

## 2. Los 13 secretos, y quién los usa

| # | Nombre en el llavero | Emisor | Lo usan | Caduca |
|---|---|---|---|---|
| 1 | `GITHUB_TOKEN` (=`GH_TOKEN`) | GitHub | **todo**: clonar privados, empujar, `gh` | sí, lo pones tú |
| 2 | `DO_TOKEN` | DigitalOcean | el lanzador: crear y destruir droplets | sí, lo pones tú |
| 3 | `CLAUDE_CODE_OAUTH_TOKEN` | Anthropic (`claude setup-token`) | Claude Code en el dev | largo |
| 4 | `VAST_AI_API_TOKEN` | Vast.ai | alquilar y **apagar** GPUs | no declarada |
| 5 | `TG_BOT_TOKEN` | @BotFather | bot **Coordinador** (en dev) | no |
| 6 | `TGL_BOT_TOKEN` | @BotFather | bot **Lanzador** (en mini) | no |
| 7 | `CWEB_TS_AUTHKEY` | Tailscale | web de lectura en el móvil | **sí, 90 días máx.** |
| 8 | `FVW_WEB_TOKEN` | *se genera solo* en `foveal-vision` | la web app de `:8010` | no |
| 9 | clave SSH `~/.ssh/do_droplet` | tú (`ssh-keygen`) | entrar en todos los droplets | no |
| 10 | `DB_PASSWORD` | tu PostgreSQL local | `comercial-DB`, `-desnormalizada`, `-resumen` | no |
| 11 | `DB_*_URL` (4) | PostgreSQL de `dashboard-sagj` | el dashboard | no |
| 12 | `SSH_PASSWORD` | servidor SAGJ | túnel del dashboard | no |
| 13 | `SSH_PKEY_PASSPHRASE` (+`SSH_PKEY_PATH`) | tú | túnel del dashboard | no |

**Identificadores** (no abren nada, pero sin ellos no funciona): `TG_ALLOWED_USER_IDS`,
`TGL_ALLOWED_USER_IDS` (tu id numérico de Telegram, se recupera con `/whoami` al bot) y
`GIT_USER_NAME`/`GIT_USER_EMAIL`.

⚠ **El #7 es el único con cuenta atrás.** 90 días es el máximo que Tailscale permite, no
es opcional, y el síntoma al caducar es **una máquina que arranca perfectamente y no
aparece en el tailnet**: sin error, sin log, sin nada.

⚠ **El #8 no se recupera: se regenera**, y eso cambia la dirección. Lo produce la propia
app; lo que hay que salvar no es el valor sino saber que existe, porque **sobrevive a
rehacer el dev sólo si viaja desde el llavero**.

---

## 3. El mapa: qué proyecto pide qué

### 3.1 La flota (`digital-ocean-dropplet-auto-launching`)

Es el único con `.env.example` completo y comentado. **36 nombres**, de los que 6 son
secretos y el resto configuración con valores por defecto razonables.

| grupo | nombres |
|---|---|
| secretos | `DO_TOKEN` `GITHUB_TOKEN` `CLAUDE_CODE_OAUTH_TOKEN` `VAST_AI_API_TOKEN` `TG_BOT_TOKEN` `TGL_BOT_TOKEN` `CWEB_TS_AUTHKEY` |
| puentes `TGL_` (van al bot Lanzador) | `TGL_ALLOWED_USER_IDS` `TGL_DO_TOKEN` `TGL_VAST_AI_API_TOKEN` `TGL_CLAUDE_PERMISSION_MODE` |
| puentes `TG_` (van al bot Coordinador) | `TG_ALLOWED_USER_IDS` `TG_CLAUDE_PERMISSION_MODE` |
| máquina | `DO_TYPE` `DO_SIZE` `DO_IMAGE` `DO_REGION` `DO_DROPLET_NAME` `DO_TAG` `DO_CLOUD_INIT` `DO_VOLUME` `DO_VOLUME_SIZE_GB` `DO_MAX_PRICE_MONTHLY` |
| acceso | `DO_SSH_KEY_FILE` `DO_SSH_KEYS` `DO_SSH_USER` `DO_SSH_PORTS` `DO_DEV_USER` |
| contenido | `DO_REPOS` `DO_SERVICES` `GIT_USER_NAME` `GIT_USER_EMAIL` |
| Vast | `VAST_MAX_PRICE_HOURLY` `VAST_IMAGE` `VAST_DISK_GB` `VAST_SSH_KEY_FILE` `BENCH_SRC` |

### 3.2 Los demás proyectos

| repo | `.env` local | `.env.example` en git | nombres | secretos |
|---|---|---|---|---|
| `telegram-coordinator` | sí | **incompleto** (2 de 4) | `BOT_TOKEN` `ALLOWED_USER_IDS` `CLAUDE_PERMISSION_MODE` `COMMAND_TIMEOUT_MS` | `BOT_TOKEN` |
| `claude-code-webapp-mobile` | no (lo genera el lanzador) | sí, **creado el 2026-09-10** | `TS_AUTHKEY` `CWEB_PORT` `CWEB_PUERTO_TS` `CWEB_SONDEO_MS` `CWEB_HOSTNAME` `CWEB_DATA_DIR` `COORD_HOME` | `TS_AUTHKEY` |
| `foveal-vision` | no | **NO EXISTE** | `WEB_TOKEN` (desde `FVW_WEB_TOKEN`), `FV_API_URL` | `WEB_TOKEN` |
| `claude-auto-retry` | sí | **incompleto** (1 de 6) | `CR_DETECTION_PRECISION` `CR_FALLBACK_HOURS` `CR_MARGIN_SECONDS` `CR_MAX_RETRIES` `CR_PERMISSION_MODE` `CR_TRANSCRIPT` | ninguno |
| `comercial-DB` | sí | sí (+2 de más) | `DB_HOST` `DB_PORT` `DB_NAME` `DB_USER` `DB_PASSWORD` · ejemplo añade `SEED_RANDOM` `SEED_SCALE` | `DB_PASSWORD` |
| `comercial-desnormalizada` | sí | sí, cuadra | `DB_*` + `DB_DESN_*` (5+5) | 2 contraseñas |
| `comercial-resumen` | sí | **incompleto** (le faltan los 6 de `DB_RESUMEN_*` y los `_TEST_NAME`) | `DB_*` `DB_DESN_*` `DB_RESUMEN_*` | 3 contraseñas |
| `comercial-demo` | no | sí, pero con una variable basura llamada `xx` | `DB_*` | `DB_PASSWORD` |
| `dashboard-sagj` | sí | sí, cuadra | `DB_ORIGINAL_URL` `DB_DENORMALIZED_URL` `DB_SUMMARY_URL` `DB_MOCK_URL` `USE_MOCK_DB` `SSH_TUNNEL_ENABLED` `SSH_HOST` `SSH_PORT` `SSH_USER` `SSH_PASSWORD` `SSH_PKEY_PATH` `SSH_PKEY_PASSPHRASE` `SSH_LOCAL_PORT` `SSH_REMOTE_BIND_HOST` `SSH_REMOTE_BIND_PORT` `ETL_*` (4) `SEED_*` (2) | 4 URL con contraseña + 2 SSH |
| `estudios-redes-neuronales` | — | no aplica | ninguno | — |
| `experimentos-cnn` (614 fich.) | — | ninguno | ninguno detectado | — |
| `image-text-sample-generator` | — | ninguno | ninguno detectado | — |

---

## 4. Los cinco agujeros que encontró el barrido

Esto no es una lista de mejoras: es lo que hoy hace que «rehacerlo desde cero» no
funcione tal cual.

### 4.1 ✅ `claude-code-webapp-mobile` no ignoraba `.env` — CORREGIDO el 2026-09-10

Su `.gitignore` **entero** es una línea: `node_modules/`. La app lee `TS_AUTHKEY` de un
`.env`, o sea que **el día que alguien cree ese fichero, git lo va a rastrear**, y el repo
es **público**.

Es literalmente el caso que la regla nueva evita: antes de escribir un `.env` hay que
preguntarle a git si lo ignora, y si no, no escribirlo.

**Arreglado el 2026-09-10** (commit `a322da5` de ese repo): `.env` y `.env.*` al
`.gitignore`, y de paso su `.env.example`, que tampoco tenía. **No hubo fuga**: el fichero
nunca llegó a existir. No era una filtración ocurrida, era una armada y esperando.

### 4.2 Tres `.env.example` mienten por defecto

`claude-auto-retry` (1 de 6), `telegram-coordinator` (2 de 4) y `comercial-resumen` (le
faltan los seis `DB_RESUMEN_*`). Un ejemplo incompleto es **peor que ninguno**: quien
rehaga el proyecto cree que ya está y descubre lo que falta cuando algo revienta.

### 4.3 Falta un `.env.example`: `foveal-vision`

Eran dos; `claude-code-webapp-mobile` se arregló el 2026-09-10 (§4.1). Queda
**`foveal-vision`**, cuya web app pide `WEB_TOKEN` (llega como `FVW_WEB_TOKEN`) y
`FV_API_URL`. Es la regla que pediste — **siempre tiene que haber `.env.example` en git
para que Claude sepa qué se pide** — y es el único activo que sigue sin cumplirla.

### 4.4 El nombre de la key de Tailscale estaba mal en el lanzador

Se puso como `TAILSCALE_AUTHKEY` y el nombre que de verdad funciona es
**`CWEB_TS_AUTHKEY`**, porque el puente de `env_prefix` es lo que la convierte en
`TS_AUTHKEY` dentro del `.env` del servicio. Lo fija el propio repo de la app
(`docs/decisiones.md`, P3). **Corregido el 2026-09-10** en el `.env` de la laptop, en el
`.env.example` y en el llavero del `mini`. Confirmado por el propio descriptor del
servicio, `services/claude-web.json`, que declara `"env_prefix": "CWEB_"`.

Deja una regla: **el nombre de una variable puente no lo elige quien la transporta, lo
declara quien la consume.**

### 4.5 `comercial-demo` tiene una variable llamada `xx`

En su `.env.example`. O sobra, o alguien la necesitaba y nadie sabe para qué.

---

## 5. Dónde vive cada copia, hoy

| sitio | qué tiene | sobrevive a |
|---|---|---|
| `c:\Desarrollo\*\.env` (laptop) | **todo**: los 13 secretos y toda la configuración | nada, si se pierde la laptop |
| `mini:~/.config/dev-secrets.env` | 20 variables de flota (medido 2026-09-10) | que se pierda la laptop |
| `mini:~/src/telegram-coordinator/.env` | 4, derivadas del llavero | ídem |
| GitHub | **sólo los `.env.example`**, sin un valor | todo |

⚠ **Los dos primeros pueden desaparecer el mismo día** — un portátil robado con el mini
caducado, o al revés. De ahí el pendiente del §6.

---

## 6. Pendiente declarado: un tercer sitio donde guardar los secretos

> **NO SE DESARROLLA hasta que el dueño lo pida explícitamente.** Anotado el 2026-09-10 a
> petición suya. Está aquí para que no se pierda, no para que alguien lo implemente.

**El problema:** hoy los secretos viven en dos sitios que pueden morir juntos (§5), y el
único plan contra el peor caso es el manual de
[`secretos-desde-cero.md`](secretos-desde-cero.md) — que funciona, pero cuesta una tarde y
obliga a rotar todo.

**Lo que ya se descartó:** cifrarlos y commitearlos en git. **Al dueño no le gusta**, y la
decisión está tomada — no volver a proponerlo.

**Lo que NO puede ser, y conviene dejarlo escrito para que nadie lo intente:** el repo de
datos (`foveal-vision-data`). Es privado, sí, pero «privado» no es «cifrado»: lo lee
cualquier token con acceso, lo clonan las máquinas de trabajo enteras, y es exactamente el
repo que más manos de máquina toca. **Ahí no van secretos, ni aunque quepan.**

**Direcciones que quedan por explorar, cuando haya tiempo libre:** un gestor de
contraseñas con CLI, un secret manager de proveedor, o algo físico fuera de línea. La
pregunta que decide es **cuántos sitios independientes tienen que fallar a la vez para
perderlo todo**: hoy son dos, y la respuesta buena es tres.

---

## 7. Cómo se rehace este inventario

No a mano. Los tres comandos que lo produjeron, para volver a pasarlos cuando cambie algo:

```sh
# 1. Qué .env y .env.example hay en cada clon local
for d in */; do [ -d "$d/.git" ] && { printf "%-42s" "${d%/}"; \
  find "$d" -maxdepth 2 -name ".env*" -not -path "*/node_modules/*" -printf "%P "; echo; }; done

# 2. Los NOMBRES de las variables de cada uno (nunca los valores)
grep -ohE '^[[:space:]]*(export[[:space:]]+)?[A-Za-z_][A-Za-z0-9_]*[[:space:]]*=' <fichero> \
  | tr -d ' \t' | sed 's/^export//;s/=$//' | sort -u

# 3. Qué variables usa un repo que NO tiene .env
grep -rhoE "process\.env\.[A-Z_]+|os\.environ\[?['\"][A-Z_]+|os\.getenv\(['\"][A-Z_]+" . \
  --include=*.py --include=*.js --include=*.mjs --include=*.ts | grep -oE "[A-Z][A-Z0-9_]{2,}" | sort -u
```
