# Sesión 2026-08-01 — Ingesta reunión Revisión de Movimientos - Guía

Relacionado: [[2026-07-31 - Revisión de Movimientos - Guía]]

## Qué se trabajó

- Se procesó con `/reunion` la transcripción de la reunión "Revisión de
  Movimientos - Guía" (2026-07-31, Angel/German/Carlos), depurando la charla
  irrelevante y creando la nota [[2026-07-31 - Revisión de Movimientos - Guía]]
  con decisiones, lógica de negocio y tareas detectadas.
- Se reconciliaron las tareas detectadas contra el backlog abierto de
  `Tareas.md`. Todas se clasificaron como nuevas (sin match existente),
  salvo el tema de citas que sí tenía una tarea abierta relacionada en la
  sección Marítimo.
- Se dieron de alta 6 tareas en `Tareas.md` (revisadas una por una con el
  usuario, dos marcadas como prioridad):
  - Bandeja de entrada: marcar referencia como urgente.
  - Nueva subsección "Movimientos y facturas por guía (Querétaro)": formulario
    de facturas en Guías (prioridad), generación automática de movimiento de
    entrada por guía House, preguntar a qué guía pertenece una factura subida
    sin guía asociada (prioridad).
  - Expediente Aduanero: integrar factura al expediente del
    movimiento/referencia (prioridad), y tarea de documento↔factura
    reescrita según aclaración del usuario en la sesión (el documento debe
    quedar vinculado a la factura y visible en Expediente Aduanero, con
    posibilidad de ligar un documento ya existente si no se sube en el
    momento).

## Commits relevantes

Ninguno — esta sesión no tocó ningún repo de código (`carmi-digital` /
`carmi-odin-api-v2`), solo el baúl.

## Decisiones (con su porqué)

Las decisiones de producto/negocio quedaron documentadas en
[[2026-07-31 - Revisión de Movimientos - Guía]] (movimiento de entrada
autogenerado por guía House, vinculación factura↔guía, unificación de
expediente, aclaración de subdivisión sobre movimiento no sobre DGO,
avatar de responsable, urgencia de referencia). Dos decisiones de proceso
propias de esta sesión:

- El usuario decidió **no dar de alta la tarea de "avatar de responsable"**
  en `Tareas.md` por ahora — se descartó explícitamente al revisarla, sin
  dar motivo adicional más allá de no priorizarla todavía.
- El usuario decidió **no dar de alta ni fusionar las tareas de citas
  marítimas/aéreas** de esta reunión, dejando intacta la tarea existente en
  la sección Marítimo (posible match, pero se optó por no tocarla).

## Pendientes

- Retomar (si aplica) la tarea de "avatar de responsable" en cabecera de
  referencia — quedó descartada esta sesión, no en el backlog.
- Revisar si el contexto de citas de esta reunión (campos, alcance a aéreo,
  historial/motivo de reprogramación) debe fusionarse más adelante con la
  tarea abierta de citas en la sección Marítimo de `Tareas.md`.
- Posible tensión sin resolver entre la decisión de "movimiento de entrada
  autogenerado por guía House" (esta reunión) y el rediseño guía-céntrico de
  Querétaro del 20/21-jul (Previo remodelado 1:1 con guía, no con
  movimiento) — el usuario pidió dar de alta la tarea tal cual sin
  reconciliarla todavía; queda pendiente resolver esa reconciliación antes
  de implementar.
- No se corrió `graphify merge-graphs` con los repos de código
  (`carmi-digital` / `carmi-odin-api-v2`): no se tocó código esta sesión y
  no se tienen a mano sus rutas locales en este entorno.
