# Conseguir todos los tokens desde cero

**Para cuando se pierda todo.** Este documento se sigue de arriba abajo, por un humano o
por Claude, y al final de él hay una flota funcionando otra vez.

**Fecha:** 2026-09-10. **Compañero:** [`secretos-inventario.md`](secretos-inventario.md) —
qué hay y quién lo usa.

> ⚠ **Ningún paso de aquí necesita un secreto anterior**, salvo donde se diga. Ése es el
> punto: si necesitara uno, no serviría para el peor caso.

---

## Antes de empezar: qué se ha verificado y qué no

| | |
|---|---|
| **Verificado el 2026-09-10** | los comandos de comprobación de los pasos 1, 2, 4 y 5 (se ejecutaron contra las APIs reales y contestaron) |
| **NO verificado** | los pasos de *emisión* (entrar al panel y crear el token). No se puede probar sin destruir los que funcionan hoy |
| **Sin forma de verificar desde aquí** | el paso 0 (recuperar cuentas) y el 7 (Tailscale, cuya validez sólo se ve al unir una máquina) |

Está dicho así a propósito: presentar un paso no probado como probado es lo que hace que
un manual de emergencia falle **el día de la emergencia**.

---

## Paso 0 — Lo que de verdad hay que proteger no es ningún token

**Todos los tokens de este documento se pueden volver a emitir en diez minutos, siempre
que puedas entrar en la cuenta.** Lo que NO se puede volver a emitir es el acceso a la
cuenta. Son seis:

| cuenta | dónde se recupera | qué pierdes si no entras |
|---|---|---|
| **GitHub** (`stalin.beltran2006@gmail.com`) | github.com/login → recuperación | **todo el código**: 62 repos |
| **DigitalOcean** | cloud.digitalocean.com | las máquinas y la facturación |
| **Anthropic / Claude** | claude.ai | Claude Code en las máquinas |
| **Telegram** | la propia app, por SMS | los dos bots |
| **Vast.ai** | cloud.vast.ai | el saldo y las GPU |
| **Tailscale** | login.tailscale.com (entra con Google/GitHub) | la web del móvil |

⚠ **Las tres primeras cuelgan de la misma cuenta de Google y del mismo 2FA.** Si pierdes
el segundo factor, pierdes las tres a la vez, y ningún token de este documento sirve de
nada.

> **Lo único que hay que guardar fuera de línea son los códigos de recuperación de Google
> y de GitHub.** Imprímelos. Es la única parte de todo este sistema que no tiene arreglo
> por software, y es la parte que el manual no puede hacer por ti.

---

## Paso 1 — `GITHUB_TOKEN` · el primero, porque sin él no hay código

Sin este token no puedes clonar los repos privados ni empujar nada, y **el lanzador se
niega a arrancar** si el que tiene está revocado (comprobación añadida tras la avería del
2026-09-06).

1. Entra en **<https://github.com/settings/personal-access-tokens>** → *Generate new
   token* → **Fine-grained**.
2. **Repository access:** *All repositories* (o, si prefieres apretar, los del sistema:
   `foveal-vision`, `foveal-vision-data`, `image-text-sample-generator`,
   `telegram-coordinator`, `digital-ocean-dropplet-auto-launching`,
   `estudios-redes-neuronales`, `claude-code-webapp-mobile`).
3. **Permisos mínimos:** `Contents` = **Read and write**. Nada más hace falta para clonar
   y empujar. Añade `Metadata` = Read si el formulario lo exige (suele ir implícito).
4. **Caducidad:** la que quieras, pero apúntala. Un token caducado **no da ningún síntoma
   inmediato**: los repos privados no se clonan y ya está.
5. Guárdalo como `GITHUB_TOKEN=` en el `.env` de
   `digital-ocean-dropplet-auto-launching`.

**Comprobación** (verificada el 2026-09-10):

```powershell
python -c "import urllib.request,json;r=urllib.request.Request('https://api.github.com/user',headers={'Authorization':'Bearer PEGA_AQUI','User-Agent':'x'});print(json.load(urllib.request.urlopen(r))['login'])"
```

Tiene que imprimir tu usuario. Si contesta **401**, el token no sirve — y ojo con la pista
falsa: `~/.git-credentials` se queda en **0 bytes** cuando GitHub rechaza una credencial,
lo que se lee como «nunca llegó el token», que es lo contrario de lo que pasó.

**Y ahora clona lo mínimo para seguir:**

```powershell
cd c:\Desarrollo
git clone https://github.com/stalinbeltran/digital-ocean-dropplet-auto-launching.git
git clone https://github.com/stalinbeltran/estudios-redes-neuronales.git
```

---

## Paso 2 — `DO_TOKEN` · sin esto no hay máquinas

1. **<https://cloud.digitalocean.com/account/api/tokens>** → *Generate New Token*.
2. **Scopes:** necesita escritura sobre `droplet`, `ssh_key`, `tag` y (si usas volúmenes)
   `block_storage`. El token completo *Read and Write* es lo simple y lo que se ha usado
   siempre.
3. Empieza por `dop_v1_`. Guárdalo como `DO_TOKEN=` en el `.env` del lanzador.

**Comprobación** (verificada el 2026-09-10):

```powershell
cd c:\Desarrollo\digital-ocean-dropplet-auto-launching
python scripts/do_droplet.py list
```

Si contesta con la tabla de droplets (aunque esté vacía), el token sirve.

⚠ **Los scopes engañan**: aceptan `volume create` y luego dan 403 al listar. Si algo falla
sólo en volúmenes, es eso y no el token entero.

---

## Paso 3 — La clave SSH · el token deja CREAR, no ENTRAR

Es el paso que más se olvida y el que deja máquinas que existen, facturan y no sirven. Un
droplet acepta las claves registradas en la cuenta **en el momento de crearlo**.

```powershell
cd c:\Desarrollo\digital-ocean-dropplet-auto-launching
python scripts/do_droplet.py keygen          # crea ~/.ssh/do_droplet si no existe
python scripts/do_droplet.py register-key    # la sube a la cuenta de DigitalOcean
python scripts/do_droplet.py keys            # comprobacion: tiene que aparecer
```

⚠ **La privada no viaja nunca.** Si trabajas desde otra máquina, repite `keygen` +
`register-key` allí: cada portátil con su clave, y `DO_SSH_KEYS=` vacío hace que todos los
droplets nuevos acepten todas.

⚠ **Las máquinas que ya existían no aceptarán la clave nueva.** Para ésas hay que entrar
por la consola web de DigitalOcean y añadirla a mano, o rehacerlas. Es la razón de ser del
diseño de la *clave de flota*
([`digital-ocean-dropplet-auto-launching/docs/flota-simetrica.md`](https://github.com/stalinbeltran/digital-ocean-dropplet-auto-launching/blob/main/docs/flota-simetrica.md),
paso 2).

---

## Paso 4 — `CLAUDE_CODE_OAUTH_TOKEN`

Se genera **una vez**, en tu máquina, y no hay que repetirlo por droplet:

```powershell
claude setup-token
```

Pega lo que salga en `CLAUDE_CODE_OAUTH_TOKEN=` del `.env` del lanzador.

**Comprobación** (verificada el 2026-09-10, en la máquina donde Claude Code está
instalado):

```powershell
claude auth status
```

**Alternativa:** `ANTHROPIC_API_KEY` desde <https://console.anthropic.com>. Funciona
igual, pero **factura aparte por uso** en vez de ir contra la suscripción.

---

## Paso 5 — Los DOS bots de Telegram

Son **dos** y no uno, y no es comodidad: Telegram sólo admite un proceso haciendo long
polling por token, y el segundo recibe un **409**. Con un solo bot, una de las dos
máquinas se queda muda.

| bot | variable | vive en | para qué |
|---|---|---|---|
| **Lanzador** | `TGL_BOT_TOKEN` | el `mini` | pedirle máquinas |
| **Coordinador** | `TG_BOT_TOKEN` | el `dev` | pedirle trabajo |

1. En Telegram, habla con **@BotFather** → `/newbot` → nombre y usuario. Repítelo **dos
   veces**, con nombres que distingas (p. ej. `…-lanzador` y `…-coordinador`).
2. Te da un token con la forma `123456789:AA...`. Uno a `TGL_BOT_TOKEN`, otro a
   `TG_BOT_TOKEN`.
3. **Tu id numérico** va en `TG_ALLOWED_USER_IDS` y `TGL_ALLOWED_USER_IDS`. Es la **única**
   protección de un bot que ejecuta comandos en la máquina: sin él no atiende a nadie, y
   con el id equivocado atiende a otro. Si no lo sabes, escríbele `/whoami` al bot ya
   arrancado, o habla con **@userinfobot**.

**Comprobación** (verificada el 2026-09-10):

```powershell
python -c "import urllib.request,json;print(json.load(urllib.request.urlopen('https://api.telegram.org/botPEGA_AQUI/getMe'))['result']['username'])"
```

Imprime el usuario del bot. Hazlo con los dos y confirma que son **distintos**.
Medido el 2026-09-10 con los tokens actuales: `@SBDevLanzador_BOT` (el `TGL_`) y
`@SBDevCoordinador_BOT` (el `TG_`). Si los dos imprimieran lo mismo, uno de los dos
bots se quedaria mudo con un 409 y el sintoma no diria por que.

---

## Paso 6 — `VAST_AI_API_TOKEN`

Sólo hace falta para medir GPU. La flota arranca sin él, pero el botón de **apagar** lo
necesita, y ése es el que cuesta dinero si falta.

1. **<https://cloud.vast.ai/manage-keys/>** (Account → Keys) → crear una key.
2. ⚠ **Que NO sea de sólo lectura.** Una key recortada autentica, lista el catálogo y
   parece correcta: falla **sólo al alquilar**, y no hay forma de distinguirlo antes.
3. ⚠ **Comprueba que hay saldo.** Una cuenta a cero pasa todas las comprobaciones de token
   y el primer alquiler falla por algo que no se parece en nada a «no tienes dinero».
4. Guárdalo en `VAST_AI_API_TOKEN=` **y** en `TGL_VAST_AI_API_TOKEN=` (el segundo es el
   que llega al bot Lanzador).

**Comprobación:**

```powershell
python scripts/vast_check.py
```

Comprueba identidad, saldo, catálogo, instancias vivas y claves SSH **sin alquilar nada**,
y sale 0/1 para poder encadenarlo.

⚠ **NO verificado el 2026-09-10.** Se intentó tres veces y el comando **no imprimió nada en
100 segundos** antes de que lo cortara el `timeout`. Ese mismo día la red de la laptop dio
lecturas caídas repetidas contra `api.github.com` y `raw.githubusercontent.com`, así que lo
más probable es que sea la red y no el script — pero **no está comprobado**, y decirlo es
más útil que suponerlo. Si al seguir este manual tampoco contesta, comprueba a mano que el
catálogo responde (no necesita token):

```powershell
python -c "import urllib.request;print(urllib.request.urlopen('https://cloud.vast.ai/api/v0/bundles/',timeout=30).status)"
```

Si eso contesta 200 y `vast_check.py` no, el problema es el token o el script; si tampoco
contesta, es la red.

---

## Paso 7 — `CWEB_TS_AUTHKEY` (Tailscale) · el único con cuenta atrás

Es la auth key con la que el `dev` se une al tailnet para servir la web de lectura en el
móvil.

1. **<https://login.tailscale.com/admin/settings/keys>** → *Generate auth key*.
2. **Los tres ajustes que deciden si sirve para una flota**, y los tres fallan tarde:
   - **Reusable** — sin esto vale para **un** nodo, y la flota son varios. La segunda
     máquina falla con un error que no dice «ya la usaste».
   - **Ephemeral** — sin esto, cada `dev` destruido deja un **nodo muerto** en el tailnet.
     Y `dev` es desechable por diseño, así que se acumulan y el nombre estable deja de
     resolver.
   - **Caducidad** — **90 días es el máximo y no es opcional**. Al caducar, la máquina
     nueva arranca perfectamente y **no aparece en el tailnet**: sin error y sin log.
3. Guárdala como **`CWEB_TS_AUTHKEY=`** (no `TAILSCALE_AUTHKEY`, ni `TS_AUTHKEY`). El
   prefijo `CWEB_` es el puente: el lanzador le quita el prefijo y escribe `TS_AUTHKEY` en
   el `.env` del servicio, dentro del `dev`. **El nombre lo declara quien la consume**
   (`claude-code-webapp-mobile/docs/decisiones.md`, P3), no quien la transporta.

**Comprobación:** no hay ninguna que se pueda hacer desde fuera — una auth key no se puede
consultar sin un token de API aparte. La única prueba real es lanzar un `dev` y ver si
aparece en <https://login.tailscale.com/admin/machines>.

**Alternativa que quita el problema de raíz:** un **OAuth client** en vez de una auth key.
No caduca. Está apuntado como remedio en `docs/decisiones.md` de la app y **no está
implementado**.

---

## Paso 8 — Los que NO se recuperan: se regeneran

| variable | qué hacer |
|---|---|
| `FVW_WEB_TOKEN` | lo genera la propia web app de `foveal-vision` al arrancar. **No lo busques**: pon uno nuevo tú (cualquier cadena larga y aleatoria) en el `.env` del lanzador, y con eso sobrevive a rehacer el `dev` |
| clave de flota / `lanzador-*` | se regeneran con `keygen` (paso 3). Las viejas de la cuenta se pueden borrar |

---

## Paso 9 — Las bases de datos (proyectos `comercial-*` y `dashboard-sagj`)

Éstas **no vienen de ningún proveedor**: son tuyas.

| variable | de dónde sale |
|---|---|
| `DB_PASSWORD`, `DB_DESN_PASSWORD`, `DB_RESUMEN_PASSWORD` | la contraseña del usuario de **tu PostgreSQL local**. Si se pierde, se cambia con `ALTER USER … PASSWORD …` desde una sesión de superusuario |
| `DB_ORIGINAL_URL` y las otras tres de `dashboard-sagj` | la cadena entera `postgresql://usuario:clave@host:puerto/base`. Lleva la contraseña dentro |
| `SSH_HOST` `SSH_PORT` `SSH_USER` `SSH_PASSWORD` | credenciales del **servidor SAGJ**, para el túnel. Si se pierden, las da quien administre ese servidor — no hay autoservicio |
| `SSH_PKEY_PATH` `SSH_PKEY_PASSPHRASE` | tu clave privada y su frase. Si se pierde la frase, **la clave no se recupera**: se genera otra y se registra en el servidor |

⚠ **`SSH_PASSWORD` y la del servidor SAGJ son las únicas de todo el inventario que
dependen de un tercero.** Todo lo demás lo puedes reemitir tú solo.

---

## Paso 10 — Reconstruir la flota

Con los pasos 1-7 hechos, el `.env` está completo. Ahora:

```powershell
cd c:\Desarrollo\digital-ocean-dropplet-auto-launching
python scripts/do_droplet.py launch mini --type mini
# ...y desde el bot Lanzador, o desde aqui:
python scripts/do_droplet.py launch dev --type dev
```

**Comprobación final, que es la que de verdad cuenta:**

1. El bot Lanzador contesta en Telegram.
2. `estado` enseña las dos nubes.
3. `lanzar launch dev --type dev` crea la máquina de trabajo con sus repos.
4. El bot Coordinador contesta.

Si los cuatro salen, está recuperado.

---

## Y la regla que evita repetir esto: `.env.example` SIEMPRE en git

> **Todo proyecto lleva su `.env.example` commiteado, con todas las variables que pide y
> ningún valor.** Es lo que permite que Claude —o tú dentro de seis meses— sepa qué se
> pide sin adivinarlo leyendo el código.

Tres condiciones, y las tres se incumplen hoy en algún repo
([inventario §4](secretos-inventario.md)):

1. **Existe.** Falta en `claude-code-webapp-mobile` y en `foveal-vision`.
2. **Está completo.** `claude-auto-retry` declara 1 de 6; `telegram-coordinator`, 2 de 4;
   `comercial-resumen` se deja seis variables.
3. **El `.gitignore` cubre `.env`.** ⚠ `claude-code-webapp-mobile` **no lo cubre**, y es
   un repo **público**: el día que alguien cree el fichero, git lo rastrea.

La comprobación es una línea y se puede meter donde se quiera:

```sh
git -C <repo> check-ignore -q .env || echo "PELIGRO: $repo no ignora .env"
```
