# Sesión 2026-07-31 — Ingesta Daily Scrum: incrementables COVE

Relacionado: [[2026-07-31 - Daily Scrum - incrementables de factura para manifestación de valor y subdivisión por guía]]

## Qué se trabajó

Ingesta con `/reunion` de la transcripción del Daily Scrum de hoy (Enrique Lopez,
Angel Huberto Pulido Burgos, Perla Lopez). Se depuró la transcripción, se
escribió la nota limpia en `Reuniones/`, se reconcilió contra el backlog
abierto de `Tareas.md` (vía subagente Explore, sin encontrar antecedentes) y
se dio de alta la tarea nueva detectada, confirmada por el usuario.

## Commits relevantes

Ninguno — el repo de código `sigac-3` en esta máquina no es un repositorio
git (`git status` falla con "no es un repositorio git"). Los cambios de esta
sesión son solo sobre el baúl de Obsidian (sin commitear, quedan para
revisión/commit humano):
- Nota nueva: `Reuniones/2026-07-31 - Daily Scrum - incrementables de factura para manifestación de valor y subdivisión por guía.md`
- `Tareas.md` modificado (nueva sección "Manifestación de Valor / COVE")

## Decisiones (con su porqué)

- Se crea la sección nueva "Manifestación de Valor / COVE" en `Tareas.md`
  porque ninguna sección existente calzaba con el tema (no hay sección de
  facturación/COVE previa).
- Se confirma como tarea **nueva** (no fusión) tras verificar con un
  subagente que no hay antecedentes de "incrementables"/"manifestación de
  valor" con este nivel de detalle en el baúl — solo menciones generales del
  proceso manual de MV en `2026-06-30 - Prueba operación real - glosa, pago y DODA.md`.

## Aprendizajes / errores a no repetir

- El directorio de trabajo del repo de código (`carmi/sigac-3`) no es un
  repositorio git en esta máquina — para `/sync` futuros, no asumir que
  `git log`/`git status` funcionarán ahí sin verificar primero.

## Pendientes

- Tarea de código: implementar el desglose de incrementables de factura
  según COVE (6-7 conceptos) antes de que arranque el proceso de
  manifestación de valor el 2026-08-01 — ver `Tareas.md` sección
  "Manifestación de Valor / COVE". Aún no tiene `/plan` ni sesión de
  implementación.
- Enrique adelantó que la semana del 2026-08-03 viene un cambio importante
  adicional en manifestación de valor — sin detalle aún, dar seguimiento en
  el próximo daily.
