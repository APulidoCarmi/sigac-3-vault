# Sesión 2026-08-09 — Subagentes con allowlist de tools y contrato entre capas

Relacionado: [[Presupuesto de tokens y sesiones limpias]], [[Contrato entre capas]],
[[Planes paraguas y replaneo]], [[Mapa de flujos del cascaron]]

> Sesión sin compactación: el detalle de abajo es de primera mano, no un resumen.

## Qué se trabajó

Partió de una pregunta de criterio, no de un plan: si conviene dejar de darle
back y front a una sola sesión y usar "experto en back" / "experto en front".

La respuesta corta a la que se llegó: **la persona en el prompt es folclore, el
aislamiento de contexto es lo real**, y el cascarón ya tenía lo segundo casi
entero (`/implementa-paraguas` con subagente por sub-plan, `/implementa` con
corte obligatorio, `model:` fijo por skill). De ahí salió un diagnóstico de tres
mejoras, y se implementaron dos.

### Mejora 1 — subagentes con allowlist de tools (hecha)

- `template/.claude/agents/verificador-front.md` — único contexto del entorno
  con `mcp__playwright__*` en su `tools`. Sin `Edit`/`Write` a propósito.
- `template/.claude/agents/implementador-subplan.md` — el que lanza el paraguas,
  uno por hijo. **Sin** Playwright. Absorbe las ~20 líneas de disciplina que el
  orquestador reenviaba en cada spawn.
- `template/.claude/playwright-guard.js` — hook `PreToolUse` que deniega
  Playwright cuando la llamada no nace en un subagente.
- Aprovisionamiento en `server/index.js` (`TEMPLATE_AGENTS`, copia a
  `.claude/agents/`, exclusión en `.git/info/exclude`, `playwright-guard.js` en
  `TEMPLATE_CLAUDE_FILES` y `TEMPLATE_HOOK_FILES`).
- Skills actualizadas: `/verify` e `/implementa` delegan por `subagent_type`,
  `/implementa-paraguas` deja de reenviar boilerplate.

### Mejora 3 — el contrato entre capas (hecha)

- Regla nueva [[Contrato entre capas]] en `template/Reglas/`.
- `/plan`: pregunta de entrevista sobre fronteras + sección `## Contrato` en la
  plantilla, solo si el requerimiento cruza capas.
- `/implementa` y `implementador-subplan`: el contrato es de solo lectura;
  si no sirve, `ESTADO: replaneo`.
- `/implementa-paraguas`: **puerta de contrato** antes de paralelizar, y el
  contrato se pega **textual** en el prompt de cada hijo.

### Mejora 2 — propuesta y NO implementada

Cambiar el eje de partición de "sub-plan" a "capa". **Se descartó
deliberadamente**, ver Decisiones.

## Commits relevantes

Ninguno: todo quedó en el working tree sin commitear, según la regla del
cascarón. 13 archivos tocados (3 nuevos), +418/−75 antes de la mejora 3.
Suite en 19/19.

## Decisiones (con su porqué)

- **No partir el trabajo por capa (back/front), aunque sea la práctica que se
  leyó.** El eje actual del paraguas particiona por *archivos disjuntos*, y esa
  invariante es la que hace seguro el paralelismo. Un split por capa la rompe en
  cuanto un sub-plan toca ambas. El eje que ya había es mejor que el de la
  práctica de referencia.

- **Convertir las reglas de prosa en garantías del harness.** La regla "el hilo
  principal nunca llama a `mcp__playwright__*`" existía y se violó igual: 201
  `browser_snapshot`, 188M de tokens, 16,6% del costo de `/implementa`. Una
  allowlist `tools:` la aplica el harness; una advertencia no.

- **El guardián distingue hilo por `isSidechain` del transcript**, el mismo
  mecanismo que ya usaba `reread-guard.js`. Se descartó `permissions.deny` en
  `settings.json` porque es project-wide: bloquearía también al subagente y
  dejaría la verificación sin salida.

- **Ante la duda, el guardián PERMITE.** Un falso bloqueo deja la verificación
  de front sin salida; un falso permiso solo cuesta tokens. Escape deliberado:
  `MI_IDE_ALLOW_PLAYWRIGHT=1`.

- **El contrato se pega textual, no se enlaza ni se resume.** El subagente
  arranca en frío y una paráfrasis del payload es exactamente por donde divergen
  las mitades.

## Aprendizajes / errores a no repetir

- **"Sin archivos en común" NO garantiza que las mitades encajen.** Es el hueco
  de calidad que se encontró: el hijo del front y el del back no comparten ni
  una línea, así que pasan el chequeo de paralelismo y aun así uno puede esperar
  `{ user: { id } }` y el otro devolver `{ userId }`. Nadie lo detecta hasta
  integrar, con las dos mitades ya escritas. Es el motivo de la mejora 3.

- **Una regla en prosa dentro de una skill no es enforcement.** Ya había
  evidencia medida de que se ignora. Cuando el harness ofrece un mecanismo
  (allowlist, hook), usarlo.

- **La mejora 3 no tiene garantía mecánica y hay que asumirlo.** No existe hook
  que valide un contrato. Lo más cercano es la puerta del paraguas
  (detenerse y preguntar) y un test que verifica que la cadena
  plan→implementa→agente→paraguas no se rompa.

- **La delegación a subagente no siempre ahorra** (ya estaba medido para
  `/sync`, se confirmó como criterio): cada subagente arranca en frío y
  re-deriva contexto. Paga cuando la tarea es grande, no cuando es cómoda de
  dividir.

- **`graphify query` no sirvió aquí**: la consulta sobre la estructura del
  template devolvió 285 nodos del plugin `obsidian-git` de `demo-vault/`. El
  grafo del cascarón está contaminado por dependencias vendorizadas. Pendiente
  abajo.

- Bug preexistente encontrado y corregido de paso: la lista `SKILLS` del test
  tenía 7 entradas y hay 8 — faltaba `implementa-paraguas`, así que esa skill
  no tenía cobertura de aprovisionamiento.

## Pendientes

- **Commitear el trabajo**: 13 archivos en el working tree, sin commit.
- **Mejora 3 cambia el comportamiento con paraguas existentes**: ninguno tiene
  `## Contrato`, así que la puerta nueva se detendrá a preguntar en los que
  crucen capas. Es intencional, pero se verá la primera vez.
- **Remedir el 16,6% de Playwright en `/implementa`** con el guardián ya activo.
  Conecta con la mejora 8 abierta (amplificación de `/implementa`): esto era una
  de sus palancas y ahora está aplicada de verdad.
- **Mejora 2 no se implementó** (decisión, no olvido).
- **`.graphifyignore` del cascarón**: `demo-vault/.obsidian/plugins/` debería
  excluirse para que `graphify query` deje de devolver ruido de `obsidian-git`.
- Dependencia frágil a documentar: la allowlist y el prefijo del guardián
  asumen que el MCP se registra como `playwright` (así lo hace `setup.sh`). Con
  otro nombre, ambos dejan de aplicar en silencio.
