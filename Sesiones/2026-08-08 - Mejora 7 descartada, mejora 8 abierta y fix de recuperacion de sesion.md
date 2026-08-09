# Sesión 2026-08-08 — Mejora 7 descartada, mejora 8 abierta y fix de recuperación de sesión

Relacionado: [[Presupuesto de tokens y sesiones limpias]],
[[2026-08-08 - Mejoras 5 y 6 del presupuesto de tokens: relecturas y ruteo de modelo]],
[[2026-08-08 - Mejora 4 del presupuesto de tokens: la skill como camino por defecto]]

## Qué se trabajó

Tres bloques, todos en `mi-ide-claude`:

1. **Mejora #7 (`/sync` en subagente): medida y DESCARTADA.** No se construyó.
2. **Mejora #8 (amplificación de `/implementa`): abierta.** Una de sus tres
   palancas aplicada; las otras dos deliberadamente bloqueadas.
3. **Bug de recuperación de sesión en `server/index.js`: corregido.**

## Commits relevantes

Ninguno. **Todo sigue en el working tree, sin commitear** — igual que #4, #5 y
#6. Son 13 archivos modificados y 4 sin trackear (`context-budget.js`,
`context-guard.js`, `reread-guard.js`, `skill-router.js`). Es el mayor
pendiente abierto: hay trabajo de cuatro mejoras sin proteger en git.

## Decisiones (con su porqué)

### #7 no se construye — el subagente era contraproducente

El plan era mover `/sync` a un subagente porque corría a 399k de contexto por
turno. Al remedir sobre 122 sesiones reales, ese número resultó ser **dos
sesiones atípicas** (07-31 y 08-06), no la norma.

Lo que decidió descartarla: `/sync` **hereda** 267k de contexto de media y
genera solo 12k propios (**21,6×** más heredado que producido), y el **96,4%**
de ese contexto es `cache_read`, que se factura al 10% de tarifa. Un subagente
arranca en frío, así que ese mismo contenido habría que reenviarlo como input
fresco a tarifa completa: de **~$34/mes a ~$217**. La herencia sale barata
precisamente porque está cacheada. `/sync` es solo el 1,45% del contexto y el
1,61% del coste real.

### En su lugar: arreglar un riesgo de fidelidad que creó la propia #1

La medición destapó un daño colateral del tope de 160k: `/sync` corre casi
siempre al final de la sesión (**89% de los episodios la cierran**, en posición
82–98%), y entra por encima del tope en 5 de 9 casos. O sea que **la
compactación ya ocurrió** y la conversación ya no es fuente fiel del detalle
que `/sync` debe volcar al baúl.

Se cambió `skills/sync/SKILL.md` para que tire de fuentes duraderas (cuerpos de
commit, `git diff --stat`, planes, nota de sesión previa) y **declare los
huecos en vez de rellenarlos**. El razonamiento: el riesgo no era escribir de
menos, sino confabular un "porqué" plausible que nadie decidió y que el baúl
luego da por cierto. También se fijó que el paso 4 (`graphify update`) va
después de escribir, y por qué: es la salida más pesada de la skill y podría
disparar una compactación con la bitácora aún sin escribir.

### #8 abierta, pero con dos de tres palancas bloqueadas a propósito

`/implementa` es el **62,9%** del coste facturable ($1.363 de $2.165) y el
80,5% de eso es `cache_read`. Pero **a diferencia de `/sync`, ese contexto no
es heredado: lo mete la propia skill**. Entraron 2,85M de tokens por resultados
de herramienta y se releyeron hasta sumar **1.130M**: amplificación **×398**.

Ese es el criterio que separa #7 de #8, y es lo reutilizable: *contexto
heredado-y-cacheado ya es barato y no se toca; contexto generado por la propia
skill se paga cientos de veces y sí se ataca.*

- **Palanca B (navegador) — APLICADA.** `/implementa` no mencionaba el
  navegador y aun así acumulaba 201 `browser_snapshot`, 302 clicks y 96
  navigates: 16,6% de su coste amplificado (~$56). Se le añadió la misma regla
  dura que ya tenía `/verify` ("el navegador vive en un subagente"). No se
  inventó nada: se copió un patrón ya resuelto.
- **Palancas A (longitud de sesión) y C (`Read`/`Bash`) — BLOQUEADAS.** Son el
  ~90% del valor, pero **la #4 ya puso el corte obligatorio entre tareas** y la
  #5 el guard de relecturas, y los datos medidos son **anteriores** a ambas.
  No demuestran que fallen: demuestran cómo era el mundo antes. Implementar ahí
  ahora sería repetir justo el error que la disciplina de "vuelve a medir"
  existe para evitar.

### El `||` que destruía sesiones

`server/index.js` lanzaba `claude --continue || claude`. Si Claude muere con
código != 0 (cierre forzado de la Mac, crash), el `||` arranca una conversación
**nueva y vacía** en el mismo repo; esa vacía pasa a ser la más reciente y el
siguiente `--continue` la retoma a ella, **enterrando el trabajo real**. El
mecanismo de recuperación se comía justo lo que debía recuperar.

Ahora se decide **antes** de lanzar, mirando el historial en disco
(`~/.claude/projects/<slug>/*.jsonl`): si hay conversación previa,
`claude --continue`; si no, `claude`. Nunca encadenados. Si `--continue` falla,
se cae a la shell sin crear nada y el historial queda intacto.

## Aprendizajes / errores a no repetir

- **No ordenes skills por "contexto por turno".** `/sync` lideraba esa tabla
  (hasta 838k/turno) y era el 1,45% del gasto; `/implementa` era el 71,95%. Un
  turno gordo que solo relee caché es barato; muchos turnos que escriben caché
  son caros. **Ordena por coste facturable.**
- **Un token no cuesta lo que pesa: cuesta peso × turnos que sobrevive.** La
  amplificación media medida es ×398. Escrito como sección nueva en la regla
  [[Presupuesto de tokens y sesiones limpias]].
- **"Vuelve a medir antes de implementar" acertó las tres veces.** En #5 afinó
  la solución, en #6 la invirtió entera, en #7 la canceló. En #8 obligó a
  bloquear las dos palancas grandes.
- **El tope de 160k de la #1 no es una panacea:** resolvió el problema de #7,
  pero en #8 solo recorta ~13% de la amplificación.
- **La #5 se infravaloró:** sus "~400k tokens ahorrados" se contaron en bruto,
  sin amplificación. Un `Read` bloqueado ahorra también sus ~420 relecturas.
  Conviene re-derivarlo.
- **Errores de estado en la propia regla del repo:** decía "solo `/plan`
  hereda" (también hereda `implementa-paraguas`) y omitía `/tickets` de la
  lista de sonnet. Corregidos. Una regla que miente sobre el estado actual es
  peor que no tenerla.

## Pendientes

- **Commitear.** Nada de #4, #5, #6, #7, #8-B ni el fix del `||` está en git.
- **Segundo hueco de recuperación, sin tocar:** si Claude termina dentro de una
  sesión tmux viva, el `exec ${shell}` deja una shell pelada y al reconectar no
  hay auto-resume. Requiere que el servidor detecte, al adjuntar, si Claude
  corre en el panel. Se paró por presupuesto de contexto (81%), no por dudas
  de diseño.
- **Desbloquear #8-A y #8-C:** remedir sobre sesiones *posteriores* a que #1,
  #4 y #5 estén en uso. Línea base a batir: amplificación ×398, mediana 313
  turnos/sesión (máx. 1.709), `Read` 45,5% / `Bash` 28,2% del coste amplificado.
- **Dos decisiones del usuario, sin resolver:** si `implementa-paraguas` debe
  entrar en `TEMPLATE_SKILLS` (hoy no se aprovisiona a nadie), y si el modelo
  global debe seguir en `opus` — el bloque `(sin skill)` fue $340, el 15,7% del
  gasto, y es lo único que hoy corre sin `model:` fijo.
