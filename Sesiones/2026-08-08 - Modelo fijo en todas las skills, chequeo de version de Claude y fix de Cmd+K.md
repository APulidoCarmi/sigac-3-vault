# Sesión 2026-08-08 — Modelo fijo en todas las skills, chequeo de versión de Claude y fix de Cmd+K

Relacionado: [[Presupuesto de tokens y sesiones limpias]], [[Mapa de flujos del cascaron]], [[Correr la suite antes de commitear]], [[Disciplina multi-cliente y sincronizacion del baul]]

Sesión sobre el cascarón (`mi-ide-claude`), no sobre código de sigac-3.

## Qué se trabajó

Tres cosas encadenadas: se arregló el Cmd+K de la consola del IDE, se cerraron
las dos decisiones que quedaban abiertas del presupuesto de tokens, y se agregó
un chequeo de versión de Claude Code al arranque del IDE.

## Commits relevantes

- `483a2ba` — presupuesto de tokens (guardianes, modelo fijo por skill,
  `implementa-paraguas` aprovisionada) y fix de Cmd+K. Arrastraba todo el
  working tree acumulado de sesiones anteriores: 21 archivos, +1679/-114.
- `b486683` — chequeo de actualización de Claude Code al arrancar, y el test
  que `483a2ba` dejó en rojo.

Ambos empujados a `origin/main`.

## Decisiones (con su porqué)

**Cmd+K mandaba `Ctrl+L` al pty en vez de borrar por su cuenta.** El bug era
que xterm es solo la pantalla: el handler borraba el buffer del lado del
navegador, pero nadie avisaba al proceso dueño de la pantalla (zsh o la TUI de
Claude), que no repintaba. Quedaba negro hasta la siguiente tecla. En el buffer
alternativo era peor, porque `\x1b[2J\x1b[3J\x1b[H` borraba lo que Claude había
dibujado y Claude nunca lo redibujaba. La solución no es borrar mejor, es pedir
el repintado a quien manda: `\x0c` por el websocket. zsh lo trata como
clear-screen y redibuja el prompt conservando lo escrito; Claude repinta su TUI.

**Las 8 skills declaran `model:`; ninguna hereda ya.** `/plan` e
`/implementa-paraguas` eran las últimas que heredaban. El argumento que movió a
`/plan` —que estaba documentado como excepción deliberada por ser la fase de
juicio abierto— fue que heredar no significa "elegir opus para planificar":
significa dejar que decida lo que quedó puesto con `/model`, que es otra cosa.
Fijarlo no cierra la puerta, la vuelve explícita: para subirlo se edita el
frontmatter a mano. En `/implementa-paraguas` pesó además que sus subagentes no
declaran modelo, así que heredaban del padre y un opus global se multiplicaba
por cada hijo.

**`implementa-paraguas` se agregó a `TEMPLATE_SKILLS`.** Existía en `template/`
y estaba commiteada, pero no figuraba en la lista que `provisionTemplate()`
copia, así que nunca llegó a ningún repo de cliente. No era intencional. Con
eso los nombres reservados del cascarón pasan de 7 a 8, dato que importa porque
es lo que le dice a un cliente qué nombres no puede usar para sus skills
propias: una skill del cliente llamada así se sobreescribiría en cada arranque.

**El chequeo de versión consulta el registro de npm.** `claude update` no tiene
modo "solo comprobar" —comprueba e instala en el mismo paso—, así que para
poder *ofrecer* la opción hay que saber la última versión por otra vía. Se
verificó que npm publica exactamente la misma versión que el build nativo
instalado (2.1.226 en ambos), así que los dos trenes van sincronizados; si se
desviaran, lo peor que pasa es ofrecer algo que `claude update` resuelve como
no-op.

**Se pregunta antes de levantar el pty**, porque `claude update` reemplaza el
binario y hacerlo con una sesión de Claude viva la deja corriendo sobre una
versión que ya no está en disco. Y la consulta de red se dispara sin `await` al
inicio de `main()`, para que su latencia se solape con la selección de cliente
en vez de sumarse. Nunca bloquea el arranque: sin red o con timeout devuelve
`null`, avisa y sigue.

## Aprendizajes / errores a no repetir

**Se commiteó y empujó `483a2ba` con la suite en rojo.** No se corrió `npm test`
antes de commitear. El test `provisionTemplate: un settings.json ajeno no se
pisa` llevaba roto desde que se escribió `ensureTokenBudgetSettings`, que
fusiona en el settings del cliente a propósito mientras el test exigía que
quedara byte a byte igual. Se detectó recién en el commit siguiente. Regla
nueva: [[Correr la suite antes de commitear]].

**Un test que falla puede estar describiendo un contrato viejo, no un bug.**
Aquí la respuesta correcta no era "arreglar el código para que el test pase"
sino reescribir el test contra el contrato real: la config propia del cliente
sobrevive, y las claves del presupuesto se fusionan. Conviene leer qué afirma
el test antes de asumir de qué lado está el error.

**`model:` sí es frontmatter válido de una skill, y lo respeta el CLI, no el
repo.** Verificado en el bundle de Claude Code 2.1.226, que documenta
internamente "Heavy skills can be scoped down or run with a cheaper model via
skill frontmatter". El matiz importante: el soporte lo pone la versión del CLI
instalada en cada máquina. En una con CLI viejo el campo se ignora **en
silencio** —sin error, sin aviso— y todo corre en opus. O sea que el ahorro por
modelo no viaja con el `git pull`; depende de la máquina.

**Cómo se propagan las mejoras a otras máquinas.** No hace falta re-crear los
baúles: `provisionTemplate()` recopia skills y los `.js` del cascarón, y
`ensureTokenBudgetSettings()` fusiona presupuesto y hooks en el settings del
cliente, **en cada arranque del server**. Lo que no se re-siembra por
`setup-baul.sh` es solo `Reglas/`. Para un archivo estático como `public/js/app.js`
basta el pull más una recarga dura del navegador.

**Archivos nuevos sin trackear pueden romper el arranque, no solo "no mejorar".**
Los cuatro hooks estaban en `??`; `git commit -a` no los habría incluido, y
`provisionTemplate()` los copia con `fs.copyFileSync` sin try/catch desde una
llamada pelada en `main()`. Un pull con el settings pero sin los `.js` habría
tirado `ENOENT` antes de `app.listen`.

## Pendientes

- **Probar el arranque real del IDE.** Se verificó sintaxis, el comparador de
  versiones (7 casos), la suite completa y a mano los dos comandos externos,
  pero no se corrió `main()` de punta a punta. La rama "hay versión nueva"
  recién se ejercita cuando Anthropic publique la siguiente.
- **Correr `claude --version` en las otras máquinas** antes de dar por hecho el
  ahorro por modelo.
- **Candidato #8 del presupuesto** (`/implementa`, ~72-76% del gasto) sigue
  abierto, sin abrir todavía.
- **Observación de #4 y #5** en el día a día: si el ruteo del `skill-router.js`
  molesta, y si el bloqueo de relecturas estorba.
