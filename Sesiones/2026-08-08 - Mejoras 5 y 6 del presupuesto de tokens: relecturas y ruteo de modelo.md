# Sesión 2026-08-08 — Mejoras 5 y 6 del presupuesto de tokens

Relacionado: [[Presupuesto de tokens y sesiones limpias]],
[[2026-08-08 - Mejora 4 del presupuesto de tokens: la skill como camino por defecto]]

Continuación del plan de 7 mejoras de gasto de tokens en `mi-ide-claude`.
Cerradas hoy **#5 (round-trips)** y **#6 (ruteo de modelo por skill)**. Todo
queda en el working tree del cascarón, **sin commitear**.

## Qué se trabajó

**#5 — Round-trips.** Se re-midió sobre 118 transcripciones reales
(jul–ago 2026) antes de tocar nada, y la re-medición cambió el diagnóstico:
los 16,5 round-trips por mensaje del análisis original son hoy **9,8**, con
**mediana 3**. El gasto no está repartido: **el 10% de turnos más pesados
concentra el 52% de las llamadas**. Dos hallazgos nuevos:

- **El 53% de los `Read` eran relecturas** (869 de 1.635); un servicio se
  releyó 75 veces en una sola sesión.
- **El 98,9% de las respuestas usaba UNA sola herramienta** — el paralelismo
  prácticamente no se usa.

Se implementó `template/.claude/reread-guard.js`, hook `PreToolUse` sobre
`Read` que **deniega releer un archivo que no ha cambiado**. Lo de agrupar
llamadas quedó como regla escrita: ningún hook puede forzarlo.

**#6 — Ruteo de modelo por skill.** También se midió primero, y el dato
**invirtió el plan** (ver Decisiones). Cinco skills quedaron con `model:`
fijo en su frontmatter.

## Commits relevantes

Ninguno: la sesión no commiteó nada, por diseño. El cascarón queda con
`reread-guard.js` nuevo y modificaciones en `README.md`, `server/index.js`,
los dos `settings.json`, la regla del baúl y seis `SKILL.md`.

## Decisiones (con su porqué)

**El guardián de relecturas bloquea en vez de avisar.** Al contrario que el
guardián de contexto, aquí no se pierde nada: el contenido sigue en la
conversación, así que basta con mirar más arriba. Un aviso costaría tokens y
dejaría pasar la relectura igual. Lleva **tres puertas de escape** para que
nunca sea un callejón sin salida: pasa si el archivo cambió (el caso normal
tras un `Edit`), si hubo compactación (lo anterior ya no está en la ventana)
y si se insiste tras un bloqueo.

**Escanea el transcript en vez de guardar estado propio**, porque el
transcript *es* la ventana: cualquier caché paralelo se desincronizaría justo
en los casos que importan (compactación, `/clear`, sesión reanudada).

**#6 se ejecutó al revés del plan original.** El plan decía "Haiku para
`/hoy`, `/reunion`, `/sync`; Sonnet para `/plan`, `/implementa`". La medición
mostró que el gasto está concentradísimo:

| Skill | % del gasto | Modelo que ya usaba |
|---|---|---|
| `/implementa` | 75,9% | sonnet |
| `/verify` | 11,5% | sonnet |
| `/hoy` + `/sync` + `/reunion` | 4,1% | mezcla, ya con haiku |

Es decir: el plan apuntaba a las skills que suman el 4% —y que ya corrían
parcialmente en Haiku— mientras el 87% estaba en dos que **ya usaban** el
modelo que #6 quería asignarles. Ejecutado tal cual, no habría hecho nada.

**Lo que sí movía la aguja era el sentido contrario:** impedir que las skills
caras hereden un `opus` global puesto con `/model` y fácil de olvidar. Se
fijó `model:` en el frontmatter de seis skills:

| Skill | Modelo | Criterio |
|---|---|---|
| `/implementa` `/verify` `/sync` `/reunion` `/tickets` | `sonnet` | procedimiento escrito |
| `/hoy` | `haiku` | listar y traer contexto |
| `/plan` | hereda el global | único diseño abierto |

**Criterio destilado, que es lo reutilizable:** *fija `model:` en toda skill
que siga un procedimiento escrito en su `SKILL.md`; deja heredar solo donde
se produce juicio abierto.* La pregunta no es "¿qué tan importante es esta
skill?" sino "¿el modelo caro da un resultado que se note?".

`/sync` y `/reunion` se añadieron a petición del usuario ("opus es muy
caro"), y el dato lo respaldó: `/sync` corrió históricamente el **100% en
sonnet** y `/reunion` mayoritariamente en sonnet y haiku. No es bajarlas de
modelo, es congelar lo que ya funcionaba. Proyectado al volumen medido, la
diferencia entre opus y sonnet en esas dos ronda **$1.650/mes**.

`/plan` es la excepción deliberada: es donde se decide qué se construye, y un
plan mediocre cuesta mucho más que la diferencia de tarifa.

## Aprendizajes / errores a no repetir

**El número bruto engañaba, y por mucho.** El titular "53% de las lecturas
son relecturas" sugería un ahorro enorme. Aplicando las tres puertas de
escape al histórico, el hook solo habría bloqueado **40 relecturas** en 118
sesiones (~400k tokens/mes): la inmensa mayoría eran legítimas porque el
archivo se había editado entre medias. **20× menos de lo que sugería el dato
crudo.** Sigue valiendo la pena, pero el 53% no debe usarse como métrica de
éxito.

**Re-medir antes de implementar acertó las dos veces.** La nota del plan
avisaba de que el tope de 160k de #1 ya había recortado el margen de #5/#6/#7.
En #5 el margen encogió; en #6 la medición **cambió la solución entera**.
Vale para #7: medir primero, decidir después.

**Un filtro de rendimiento puede introducir un bug de corrección.** El hook
salta el `JSON.parse` de las líneas que no mencionan el archivo, para poder
escanear transcripciones de decenas de MB. Eso descartaba el `tool_result` de
una lectura **fallida** —que no repite la ruta, solo el `tool_use_id`—, así
que una lectura que falló contaba como buena y bloqueaba el reintento
legítimo. Lo cazó el test, no la revisión a ojo.

**Verificar el mecanismo antes de confiar en él.** `model:` en el frontmatter
de una skill se confirmó primero en el binario del CLI (aparece como
*capability frontmatter* junto a `allowed-tools`, `hooks`, `shell`) y después
**empíricamente**: una skill de prueba con `model: haiku` lanzada con
`claude -p --model opus` registró `claude-haiku-4-5` en todos los turnos. El
frontmatter **gana** sobre el modelo global y se aplica solo, sin tocar
`/model`.

**`/tickets` quedó fuera por omisión, no por decisión.** No aparecía en la
medición (0 sesiones registradas), así que se pasó por alto al aplicar el
criterio. Detectado al verificar el resultado final. Cuidado con las piezas
sin datos: no aparecer en una tabla no es lo mismo que no aplicar.

## Pendientes

- **Nada está commiteado.** #4, #5 y #6 conviven en el working tree del
  cascarón.
- **#7 — `/sync` en subagente** (corría a 399k ctx/turno). Es lo único que
  queda del plan de 7. **Re-medir antes**: el tope de 160k ya recorta lo peor.
- **Verificar en uso**: que el bloqueo de relecturas no estorbe en el día a
  día, y si el modelo vuelve al global al salir de una skill dentro de una
  sesión interactiva larga (solo se probó en `-p`, donde la sesión termina
  con la skill).
- **Decisión abierta:** `implementa-paraguas` existe en
  `template/.claude/skills/` pero **no** está en `TEMPLATE_SKILLS` de
  `server/index.js`, así que no se aprovisiona a ningún cliente. Falta saber
  si es intencional.
