# Sesión 2026-08-08 — Mejora #4 del presupuesto de tokens: la skill como camino por defecto

Relacionado: [[Presupuesto de tokens y sesiones limpias]],
[[Umbral micro-fix y plan expres]], [[Mapa de flujos del cascaron]]

Trabajo sobre el **cascarón** (`mi-ide-claude`), no sobre el código de
sigac-3. Continúa la iniciativa de reducción de gasto de tokens nacida del
análisis de 111 sesiones reales de este cliente; hoy se cerró la mejora #4,
la última de alto impacto (#1, #2 y #3 se cerraron esta misma fecha).

## Qué se trabajó

**#4 — que la skill sea el camino por defecto.** Las mejoras anteriores
atacaban *cuánto pesa* cada turno (ventana dura de 160k, statusline,
guardián de contexto). Esta ataca *dónde ocurre el trabajo*, que resultó ser
la palanca mayor: el 88% del gasto medido ocurría en conversación suelta
—sin skill— a 294k de contexto por turno, frente a los 73k de una sesión de
`/implementa`. Dos piezas:

1. **Ruteador de skills** — `template/.claude/skill-router.js`, hook
   `UserPromptSubmit`. Si el prompt parece un requerimiento nuevo (encargo o
   reporte de bug) y la sesión no viene de ninguna skill, inyecta el
   recordatorio de pasar por `/plan`, con el umbral de micro-fix como
   escape.
2. **Corte obligatorio en `/implementa`** — cada tarea del plan es una
   sesión. Al terminar, la skill escribe el estado bajo la tarea en el plan
   y manda a `/clear` + `/implementa` para la siguiente, en vez de
   encadenarlas. La revisión del diff sí se atiende en la misma sesión: es
   parte de la tarea, no una tarea nueva.

Plomería: `skill-router.js` entra a `TEMPLATE_CLAUDE_FILES` (se aprovisiona
y se excluye del Git del cliente), `mergeGuardHooks` pasó a
`mergeShellHooks` para re-inyectar ambos hooks del cascarón en el
`settings.json` de un cliente que ya tenga el suyo, y la documentación se
actualizó en la regla del baúl, en el `CLAUDE.local.md` generado y en el
README.

## Commits relevantes

Ninguno: todo el trabajo de la iniciativa (#1 a #4) sigue en el working tree
de `mi-ide-claude`, sin commitear, a la espera de revisión humana.

## Decisiones (con su porqué)

- **El ruteador avisa, no bloquea.** Un hook `UserPromptSubmit` puede
  cancelar el prompt; se descartó. Un falso positivo bloqueante cuesta una
  interrupción real, mientras que el aviso trae escapes explícitos
  (requerimiento cosmético, o que el usuario ya haya pedido trabajar sin
  plan) y el modelo puede seguir de largo.
- **Un aviso por sesión, con marcador en el temporal del sistema.** El
  estado no se deduce del transcript para no depender del formato interno
  del CLI; `/clear` abre sesión nueva (id nuevo), que es justo cuando el
  aviso vuelve a tener sentido. Es la misma "regla de oro" del guardián:
  un hook que habla en cada turno paga con tokens el problema que dice
  resolver.
- **Heurística conservadora y validada con datos, no a ojo.** Se probó
  contra los 1.114 prompts reales de 120 sesiones de este cliente: dispara
  en el 2% de los prompts y en el 18% de las sesiones. Silencio ante
  preguntas, inspección (`revisa`, `analiza`), continuaciones (`sigue con la
  tarea 2`), cualquier mención de plan/ticket/subagente en curso, e
  incidencias de git/deploy (ahí `/plan` no aplica: no hay nada que escribir
  en `Planes/`).
- **Los reportes de bug cuentan como requerimiento.** "No me deja editar las
  guías" no trae verbo imperativo pero es exactamente el prompt que abre una
  sesión larga sin plan; se detecta por síntoma.
- **El corte forzado cede si el usuario insiste.** Tras leer el corte, si
  insiste se obedece avisando del costo en una línea. Manda el usuario; la
  skill no discute dos veces.

## Aprendizajes / errores a no repetir

- **Requerir un módulo que escucha `stdin` cuelga el proceso.** El hook se
  quedó esperando entrada al importarlo para probarlo. Los scripts de hook
  deben envolver la escucha de `stdin` en `require.main === module` para
  poder testear su lógica sin montar el hook completo.
- **Validar heurísticas contra los transcripts reales, no contra ejemplos
  inventados.** Los prompts propios del usuario traen faltas ("analisa") que
  una lista escrita a ojo no cubre: ese caso concreto era un falso positivo
  hasta que se corrió contra los datos.

## Pendientes

- Revisar y commitear el trabajo de #1 a #4 (sigue todo en el working tree).
- Observar en uso real si el ruteo molesta (afinar las listas del router) y
  si el corte forzado de `/implementa` se respeta o se negocia.
- Quedan #5 (round-trips), #6 (ruteo de modelo por skill) y #7 (`/sync` en
  subagente), como paquete menor: el tope de 160k de #1 ya recortó
  mecánicamente lo peor de #5 y #7, así que hay que **volver a medir** antes
  de invertir trabajo ahí.
- Dos decisiones abiertas del usuario: si es intencional que
  `implementa-paraguas` no esté en `TEMPLATE_SKILLS` (hoy no se aprovisiona
  a ningún cliente), y que el modelo global esté en `opus` (~5× Sonnet).
