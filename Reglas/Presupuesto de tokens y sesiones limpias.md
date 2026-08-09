<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: presupuesto de tokens y sesiones limpias

Cómo trabajar iniciativas grandes sobre monorepos grandes sin quemar el
presupuesto. Complementa a [[Planes paraguas y replaneo]] (que define el
troceo) con la disciplina de ejecución.

## El principio

**Mantén el contexto principal pequeño y desechable.** El contexto de una
sesión no se borra entre turnos: cada turno reenvía todo lo acumulado. Sobre
un monorepo grande, cualquier lectura o `grep` sin acotar se queda en el
contexto el resto de la sesión y encarece cada turno siguiente. El costo
crece de forma **cuadrática** con la longitud de la sesión.

## La ventana dura: 160k tokens

El entorno fija `autoCompactWindow: 160000` en `.claude/settings.json` y
`CLAUDE_CODE_DISABLE_1M_CONTEXT=1` (en el `env` de settings y en la terminal
que lanza el cascarón). La sesión auto-compacta al llegar a 160k en vez de
crecer hasta ~900k.

**Por qué 160k y no la ventana de 1M**, con números medidos sobre 111
sesiones reales de un cliente:

- El **72% del gasto era relectura de caché**, no lectura de archivos. Todos
  los `tool_result` de todas las sesiones sumaban 28 MB (~7M tokens leídos
  **una vez**), pero se facturaron **5,049M tokens** de cache read:
  amplificación de ~700×. Lo caro no es leer, es **cargar**.
- El contexto por turno tenía mediana de 195k y p90 de **619k**, con picos de
  934k. El **49% de los turnos superaba 200k**, y por encima de ese umbral la
  tarifa se **duplica**: el 81% de los tokens se pagaba al doble.
- Con la ventana de 1M el auto-compact deja de protegerte: solo hubo **9
  compactaciones en 111 sesiones**, con sesiones de 1,759 turnos.

**Cómo presupuestarlo:** el preámbulo fijo (sistema + CLAUDE.md + skills +
esquemas de MCP) ocupa ~40k, así que te quedan **~120k de espacio real de
trabajo**. Un archivo grande puede costarte 60k de un golpe: la mitad de tu
presupuesto en una sola lectura. Por eso las reglas de exploración de abajo
no son consejos, son lo que hace que 120k alcancen.

Si ves que la sesión compacta cada pocos turnos, el problema no es la
ventana: es que algo demasiado grande está entrando al contexto (un archivo
completo, un volcado de Playwright). Delégalo a un subagente. Claude Code
detecta ese caso solo y avisa de que "un archivo o salida de herramienta es
demasiado grande".

Para ajustar la ventana: comando `/autocompact`. **No** uses la variable
`CLAUDE_CODE_AUTO_COMPACT_WINDOW`: tiene prioridad sobre el setting y deja
`/autocompact` bloqueado.

## Los dos avisos: statusline y guardián

La ventana dura evita el desastre, pero llega tarde: cuando compacta, ya
pagaste los turnos caros. Estas dos piezas avisan antes.

**La statusline** (`.claude/statusline.js`) muestra
`Opus 5 | ctx 87k/160k 54% | 12 prompts | $3.21`. Tokens absolutos y no solo
el porcentaje, porque "54%" no te dice si el archivo que vas a abrir cabe y
"87k/160k" sí. El costo acumulado va siempre visible: es lo que convierte
"llevo muchos turnos" en una cifra.

**El guardián** (`.claude/context-guard.js`) mete el mismo dato *dentro* de
la conversación, en los dos momentos en que cambia una decisión:

| Contexto | Statusline | Qué hace el guardián |
|---|---|---|
| < 50% | verde | nada (callarse es parte del diseño) |
| ≥ 50% | amarillo | al empezar cada turno, recuerda delegar a subagente |
| ≥ 75% | rojo | además avisa antes de cada `Read`/`Grep`/`Glob`/`Bash` |
| ≥ 92% | rojo intenso | **bloquea la búsqueda sin acotar** |

Solo se bloquea la **exploración**, nunca `Read`: bloquear `Read` dejaría a
Claude sin poder editar (`Edit` exige haber leído el archivo) y la sesión se
atascaría. Si el guardián bloquea una búsqueda, la salida no es buscar por
otro camino, es `graphify query` o un subagente — y si de verdad hace falta
explorar a fondo, `/sync` + `/clear` y seguir en sesión limpia.

**Qué cuenta como "búsqueda sin acotar".** La primera versión bloqueaba
`Grep`/`Glob`, y era un bloqueo decorativo: medido sobre 441 transcripciones,
esas dos herramientas se usaron **0 veces** (el CLI ni siquiera las expone
aquí) mientras que el **60% de las 9.424 llamadas a `Bash`** eran búsquedas.
Ahora el guardián mira el comando: bloquea cuando la línea **abre** con un
buscador (`rg`, `grep`, `find`, `fd`, `tree`, `ls -R`) y no lleva techo de
salida. Con techo pasa: `| head -30`, `--count`, `-l`. Y un `grep` que
*filtra* la salida de otro comando (`git diff | grep foo`) nunca se bloquea:
eso no es explorar, es lo contrario. `Bash` no se puede bloquear entero
—hace falta para git, tests y para poder cerrar la sesión—, así que se
bloquea justo la forma que vuelca miles de líneas de golpe.

Los umbrales son fracciones de `autoCompactWindow`, no números fijos: si
cambias la ventana con `/autocompact`, los avisos se mueven solos.

## La skill es el camino por defecto, no el atajo

La ventana y el guardián atacan **cuánto pesa** cada turno. Esta regla ataca
**dónde ocurre el trabajo**, que resultó ser la palanca más grande. Medido
sobre las mismas 111 sesiones:

| | Contexto por turno | % del gasto total |
|---|---|---|
| Conversación suelta (sin skill) | **294k** | **88%** |
| Sesión de `/implementa` | 73k | 12% |

No es que la skill use un modelo más barato: es que **sin plan escrito, la
memoria del trabajo es la propia conversación**. Todo lo explorado tiene que
seguir cargado para que el siguiente turno tenga sentido, así que la sesión
solo puede crecer. Con plan, la memoria vive en el baúl y la sesión puede
ser corta y desechable — que es justo lo que hace barata a `/implementa`.

De ahí dos piezas:

**El ruteador** (`.claude/skill-router.js`, hook `UserPromptSubmit`). Cuando
llega un prompt que parece requerimiento nuevo —encargo o reporte de bug— y
la sesión no viene de ninguna skill, inyecta la ruta por defecto: pasa por
`/plan`. Es conservador a propósito: calla ante preguntas, inspección
(`revisa`, `analiza`), continuaciones (`sigue con la tarea 2`), cualquier
mención de un plan o ticket en curso, e incidencias de git/deploy — y como
mucho habla **una vez por sesión**. Medido contra 1.114 prompts reales,
dispara en el 2% de ellos y en el 18% de las sesiones. El umbral de qué
lleva plan no cambia: es el de [[Umbral micro-fix y plan expres]], solo lo
cosmético va sin plan.

**El corte obligatorio de `/implementa`.** Cada tarea del plan es una
sesión: al terminar, la skill deja el estado escrito en el plan y manda a
`/clear` + `/implementa` para la siguiente, en vez de encadenarlas. Antes
esto era un "si arrastras mucho contexto, considera…" y por eso no ocurría
nunca: encadenar tres tareas es el camino corto de 73k a 294k. La revisión
del diff sí se atiende en la misma sesión — es parte de la tarea, no una
tarea nueva.

## Regla dura: un sub-plan = una sesión limpia

- Cada sub-plan hijo de un plan paraguas se implementa en **su propia
  sesión** y se cierra con `/sync` + `/clear` antes de abrir el siguiente.
- **NUNCA** encadenes varios sub-planes en la misma sesión ni ejecutes
  "todos los planes de corrido". Es el error más caro posible y anula el
  diseño mismo de los sub-planes (que existen justo para caber en sesiones
  limpias).

## Reglas de exploración

- **Delega la exploración a subagentes** (`Explore` / `Task`). El subagente
  lee muchos archivos, quema *sus* tokens y devuelve solo la conclusión; el
  contexto principal nunca ve los volcados. Usa el hilo principal para
  editar y decidir, no para leer decenas de archivos.
- **graphify primero, acotado.** Orienta con `graphify query` / `explain` /
  `path` (subgrafo pequeño). **No leas `graphify-out/GRAPH_REPORT.md` ni
  `graphify-out/*graph*.json`**: pesan cientos de KB / decenas de MB y
  revientan el presupuesto en una sola lectura. Están bloqueados por
  permisos en `.claude/settings.json` para evitar accidentes.
- **No corras `graphify update` tras cada edición**: re-escanea todo el
  repo. Se hace una sola vez al cerrar el sub-plan (lo hace `/sync`).
- **El navegador vive en el subagente `verificador-front`.** Ya no es una
  regla que puedas saltarte: `mcp__playwright__*` está denegado en el hilo
  principal por `.claude/playwright-guard.js`, y ese agente es el único con
  esas tools en su allowlist. Usa `/verify`, o lanza el agente directamente.
  Ver la sección de abajo.

## Round-trips: no releas, y agrupa lo independiente

Cada llamada a una herramienta es un turno, y cada turno reenvía la ventana
entera. Medido sobre 118 sesiones reales (jul–ago 2026): **9.508 llamadas
para 973 mensajes del usuario**, ~9,8 por mensaje. No está repartido: la
mediana es 3 y **el 10% de turnos más pesados concentra el 52% de las
llamadas**. El gasto está en la cola, no en el uso diario.

De ahí salen dos reglas, una automática y otra tuya:

- **No releas un archivo que no ha cambiado.** De 1.635 `Read`, 869 (53%)
  eran un archivo ya leído en esa misma sesión; uno se releyó 75 veces. Lo
  vigila `.claude/reread-guard.js`, que **deniega** la relectura cuando el
  archivo sigue igual que cuando lo leíste: su contenido ya está más arriba
  en la conversación, así que basta con mirar ahí. Tiene tres puertas de
  escape, y por eso no estorba: pasa si el archivo cambió (el caso normal
  tras un `Edit`), pasa si hubo compactación (lo de antes ya no está en la
  ventana) y pasa si repites la llamada tras un bloqueo. Filtradas esas
  puertas, el hook habría bloqueado 40 relecturas de verdad redundantes en
  el mes medido: **el 53% bruto engaña, el ahorro real ronda 400k tokens.**
  Si necesitas otro trozo del archivo, pide un rango con `offset`/`limit`
  en vez de volver a volcarlo entero.
- **Agrupa las llamadas independientes en un solo turno.** El **98,9%** de
  las respuestas medidas llevaba **una sola herramienta**: el paralelismo
  prácticamente no se usó. Cuando dos comandos no dependen uno del otro
  —`lint` y `tests`, leer dos archivos distintos, `git status` y
  `git diff`— van en la misma respuesta y cuestan un turno, no dos. Esto no
  lo puede forzar ningún hook: es criterio al llamar. La cadena
  editar → compilar → leer el error → editar sí es secuencial y ahí no hay
  nada que agrupar.

## Qué modelo corre cada skill

El gasto por skill está **muy concentrado**. Medido sobre las mismas 118
sesiones: `/implementa` es el **76%** del gasto y `/verify` otro **11,5%**;
`/hoy`, `/sync` y `/reunion` juntas no llegan al **4,1%**. Cualquier ahorro
por modelo que no toque las dos primeras es ruido.

Por eso **todas** las skills traen `model:` fijo en su frontmatter:

- **`/implementa` y `/verify` → `sonnet`.** No es para bajarlas de modelo:
  ya corrían en sonnet. Es para que **no hereden un `opus` global** —puesto
  con `/model` y fácil de olvidar— y se lleven el 87% del gasto a ~5× la
  tarifa sin que nadie lo haya decidido. Dentro de esas skills `/model` ya
  no manda; para subirlas hay que editar la línea del `SKILL.md`.
- **`/sync`, `/reunion` y `/tickets` → `sonnet`.** Mismo motivo, y con
  respaldo: las sesiones de `/sync` medidas corrieron enteras en sonnet, y
  `/reunion` ya iba mayoritariamente en sonnet y en haiku. No es bajarlas de
  modelo, es impedir que hereden opus. Proyectado al volumen medido, la
  diferencia entre opus y sonnet en estas tres ronda **$1.650/mes**.
- **`/hoy` → `haiku`.** Listar las tareas del día y traer su contexto no
  necesita más.

- **`/plan` e `/implementa-paraguas` → `sonnet`.** Las dos heredaban hasta
  ahora. `/plan` por decisión —es la skill que produce diseño en vez de seguir
  un procedimiento, y ahí un plan mediocre cuesta mucho más que la diferencia
  de tarifa; son ~2% del gasto—, pero heredar no significa "elegir opus para
  planificar": significa que el modelo lo decide lo que quedó puesto con
  `/model`, que es otra cosa. Fijarlo no cierra la puerta, la vuelve explícita:
  si una iniciativa amerita subirlo, se edita esa línea del `SKILL.md`.
  `/implementa-paraguas` es un orquestador que lanza subagentes, y ninguno
  declara modelo: heredaban del padre, así que un opus global ahí se
  multiplicaba por cada hijo.

**El criterio, que es lo reutilizable:** fija `model:` en toda skill que siga
un procedimiento escrito en su `SKILL.md`. En la práctica ninguna quedó
heredando — ni siquiera `/plan`, la única de juicio abierto: heredar es un
default silencioso, no una decisión, y un default de ~5× la tarifa conviene
tomarlo a mano cuando toque.

Dato técnico que costó verificar: **`model:` sí es frontmatter válido de una
skill**, y acepta alias simples (`sonnet`, `opus`, `haiku`, `fable`,
`inherit`). Verificado empíricamente: una skill con `model: haiku` lanzada
con `claude -p --model opus` registró `claude-haiku-4-5` en todos los turnos.
El frontmatter **gana** sobre el modelo global y se aplica solo, sin tocar
`/model`.

## La amplificación: un token no cuesta lo que pesa

Es el número más importante de toda esta regla. Un token no cuesta su peso:
cuesta **peso × turnos que sobrevive en el contexto**, porque cada turno
reenvía todo lo acumulado.

Medido sobre `/implementa` (25 sesiones de `sigac-3`, 2026-08-08): entraron
**2,85M de tokens** por resultados de herramienta, y se releyeron hasta sumar
**1.130 millones**. Amplificación media: **×398**. Un resultado de `Read` se
relee 420 veces de media; uno de `Bash`, 454.

Por eso el ranking por "lo que pesa una llamada" engaña, y el que manda es
peso × relecturas:

| Fuente | Entra | Amplificado | % del coste |
|---|---|---|---|
| `Read` | 1,23M | 517M | 45,5% |
| `Bash` | 671k | 319M | 28,2% |
| `browser_snapshot` | 610k | 188M | 16,6% |

**Las tres son el 90,3%.** Todo lo demás es ruido.

Dos consecuencias prácticas:

- **Lo que no entra, no se paga 400 veces.** Acotar una salida en origen
  (un `grep` en vez de un `Read` entero, `--stat` en vez del diff completo,
  `head` en un `Bash` ruidoso) vale mucho más de lo que sugiere su tamaño.
- **La sesión corta es la palanca, no la llamada barata.** La amplificación
  la fija la longitud de la sesión: en una sesión de 40 turnos ese mismo
  `Read` se relee 20 veces, no 420. Por eso la regla de "un sub-plan = una
  sesión limpia" no es higiene, es la partida más grande del presupuesto.

Ojo con el orden de magnitud del tope de 160k: recorta ~13% de esta
amplificación, no la elimina. Ayuda, pero no sustituye a cerrar la sesión.

## El navegador: el segundo mayor gasto, y no es el que parecía

La intuición dice que lo caro de un MCP es su **esquema** (los ~24 tools de
Playwright en el preámbulo). Se midió, y es falso en este entorno: el CLI
**difiere** los esquemas MCP (los carga bajo demanda vía `ToolSearch`) y en
el preámbulo solo quedan los nombres. Arrancar en `sigac-3` con y sin MCP:

| | Preámbulo |
|---|---|
| con Playwright + Jira | 29.380 tok |
| sin ningún MCP | 28.855 tok |

**525 tokens de diferencia**: 0,3% de la ventana. Apagar servidores MCP no
es una palanca de presupuesto, y no vale la pena la fricción de andar
encendiéndolos y apagándolos. (Si el CLI dejara de diferir esquemas, esto
cambia; vuelve a medirlo antes de creerlo.)

**Lo caro es lo que devuelven.** Sobre 133 transcripciones de este entorno:

- `browser_snapshot`: 321 llamadas, **1,17M tokens** (~3.600 por llamada).
- `browser_take_screenshot`: 28 llamadas, **477k tokens** (~17.000 cada una).
- Total Playwright: **1,83M tokens** volcados en contextos principales. Es
  el segundo consumidor del entorno, solo por detrás de `Read`.
- Peor sesión: **621k tokens** de navegador en una sola conversación, casi
  cuatro veces la ventana entera.

Y recuerda la amplificación: cada token que entra al contexto se reenvía en
todos los turnos siguientes. Un screenshot de 17k en el turno 10 de una
sesión de 100 turnos no cuesta 17k, cuesta 17k × 90 relecturas.

Por eso `/verify` no abre el navegador: lanza el subagente `verificador-front`,
que lo abre, quema *su* contexto y devuelve un veredicto de quince líneas con
la ruta de la evidencia en disco. Dentro del subagente, además, se acota
(`target`, `depth`) y se vuelca a archivo (`filename`) en vez de a la
respuesta.

Durante un tiempo esto se pidió solo en prosa, y no bastó: en 25 sesiones de
`/implementa` entraron igual 201 `browser_snapshot` al hilo principal, 188M de
tokens al reenviarse, el 16,6% del costo de la skill. Desde entonces son dos
mecanismos que aplica el harness, no el modelo:

- `.claude/agents/verificador-front.md` declara `mcp__playwright__*` en su
  `tools`; `.claude/agents/implementador-subplan.md` no, así que un
  implementador no puede abrir el navegador ni queriendo.
- `.claude/playwright-guard.js` deniega esas tools cuando la llamada no nace
  en un subagente (lo distingue por la marca `isSidechain` del transcript).

Si de verdad hace falta abrir el navegador en el hilo principal, el escape
deliberado es `MI_IDE_ALLOW_PLAYWRIGHT=1` en el entorno.

Origen: incidente 2026-07-11 — se corrieron los 18 sub-planes de
[[2026-07-10-refactor-flujo-ejecutivo]] en una sola sesión sin restricción
y se agotó el presupuesto de tokens de un golpe.

## Cuándo un subagente ahorra y cuándo sale carísimo

`/verify` mete el navegador en un subagente y ahorra. Se intentó lo mismo con
`/sync` y **habría multiplicado el coste por seis**. La diferencia no es la
skill: es de dónde sale su contexto.

Un subagente arranca en frío. Eso lo hace ideal para trabajo que **genera**
mucho contexto y necesita poco de entrada, y pésimo para trabajo que
**hereda** mucho y genera poco:

| | Genera (snapshots, dumps) | Hereda (la conversación) |
|---|---|---|
| **Se factura** | tarifa completa, y se reenvía en cada turno | `cache_read`, al 10% |
| **Subagente** | lo quema fuera y devuelve un veredicto → **gran ahorro** | hay que reenviarlo en frío a tarifa completa → **10x peor** |

Medido sobre 122 sesiones de `sigac-3` (2026-08-08): `/sync` hereda 267k de
media y produce 12k propios — **21,6x más heredado que generado**, y el
**96,4%** de ese contexto es `cache_read`. Cuesta ~$34/mes; en subagente, ese
mismo contenido como input fresco costaría **~$217**.

**La regla:** antes de mover algo a un subagente, mira si su contexto es
heredado-y-cacheado o generado-en-el-momento. El contexto heredado **ya es
barato**; sacarlo a un subagente lo convierte en caro. Solo paga cuando el
subagente puede hacer su trabajo sin que le reenvíes la sesión entera.

Corolario para medir: no ordenes skills por "contexto por turno". `/sync`
lideraba esa tabla (hasta 838k/turno) y resultó ser el **1,45%** del gasto,
mientras `/implementa` era el **71,95%**. Un turno gordo que solo relee caché
es barato; muchos turnos que escriben caché son caros. Ordena por coste
facturable, no por tamaño de ventana.
