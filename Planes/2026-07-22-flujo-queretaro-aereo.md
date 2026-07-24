# Plan paraguas: Flujo Querétaro (aéreo) — tablero de guías y captura de manifiesto — SIGAC 3

> Plan paraguas según [[Planes paraguas y replaneo]]: este archivo lleva el
> contexto y las decisiones de toda la iniciativa y el índice de sub-planes
> hijos (cada uno es un plan normal, apto para una sesión limpia de
> `/implementa`, con sus propios criterios de verificación). El paraguas **no
> se implementa**: se actualiza al cerrar cada hijo.

## Contexto

Iniciativa derivada de la visita a Querétaro (16 y 20 de julio de 2026),
reconciliada contra el as-is de código en dos sesiones de análisis:
[[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]] y
[[2026-07-22 - Refinamiento de tareas Querétaro]]. Reunión de origen:
[[2026-07-20 - Recap visita Querétaro]] (reparto de trabajo confirmado:
Ángel).

**Decisión explícita del usuario (22-jul):** esta iniciativa es un plan
aparte, **NO extiende** el paraguas `2026-07-10-refactor-flujo-ejecutivo`.

Cubre las tareas de la sección "Flujo Querétaro (aéreo)" de `Tareas.md`
(tareas 13-19, subsección Tablero de Guías + Régimen y DGO).

Área de código: back `carmi-odin-api-v2/src/{references,shipments,air-manifest,email-documents}/**`
+ módulo nuevo `guias` (a crear); front `carmi-digital` — pantalla nueva de
tablero + wizard de creación de referencia existente.

## Decisiones tomadas (entrevista de este plan)

1. **Estructura**: paraguas + un sub-plan por pieza grande, cada uno en su
   propia sesión limpia (regla dura del cliente).
2. **El tablero de guías (sub-plan 01) NO reemplaza "Por identificar" del
   Inbox** — coexisten sin overlap. Por qué: semánticas distintas —
   `UnidentifiedWaybill`/"Por identificar" representa incertidumbre real de
   identidad de cliente tras un matching fallido; el tablero de guías es el
   flujo normal día a día donde el cliente ya se conoce desde la captura.
3. **Modelo de datos nuevo: entidad `Guia`**, independiente de
   `UnidentifiedWaybill` — no se reusa esa entidad por el motivo anterior.
4. **v1 de captura es manual**, para uso real de producción de Querétaro
   (no es maqueta). La ingesta automática del correo/Excel (tarea 14) queda
   en un sub-plan futuro y debe poder alimentar el mismo modelo `Guia` sin
   perder lo ya capturado a mano.
5. **Volumen esperado bajo** (decenas de guías/día) — no exige
   optimización especial en ningún sub-plan de esta ronda.
6. **"Comparación contra prealerta" fuera de alcance de toda la
   iniciativa** hasta que exista una fuente de datos de prealerta en el
   sistema — hoy es dato externo (correo/Excel de terceros) que sigac-3 no
   ingiere en ningún lado.
7. **Régimen por guía (tarea 19) se fusiona al sub-plan 01** (tablero),
   porque "crear referencia(s) desde selección de guías" (parte de la
   tarea 13) lo necesita para decidir cuántas Referencias/DGOs generar (1
   por régimen distinto presente en la selección).
8. **Hallazgo técnico**: `regimen` es string libre `VARCHAR(3)` en
   `Reference`/`Dgo` (`schema.prisma:3945` y `:3652`) — no hay catálogo
   Prisma. Existe una lista TS de códigos válidos en el front
   (`carmi-digital/lib/schemas/new-reference-basicData.ts:6`).
9. **Bug pre-existente encontrado**: `ReferencesService.create()`
   (`references.service.ts:399-528`) recibe `dto.regimen` pero nunca lo
   persiste — solo `update()` lo hace (línea 2884). Corrección incluida en
   el sub-plan 01, ya que la herencia de régimen desde la guía depende de
   que se guarde bien desde el alta.

## Fuera de alcance (toda la iniciativa, todos los sub-planes)

- Equivalente marítimo del tablero de guías (tarea diferida aparte en
  `Tareas.md`, pendiente de la reunión de marítimo no documentada).
- Cualquier cambio a "Por identificar" / `InboxDashboard.tsx`.
- Integración HTTP real con Zeus (tarea aparte, ya identificada en
  `Tareas.md` como fuera del alcance de la UI de Expediente Aduanero).

## Sub-planes hijos (índice, en orden de dependencia)

### Sub-plan 01 — Tablero de guías + régimen (tareas 13, 19) — 🟡 diseñado, listo para `/implementa`
Archivo: [[2026-07-22-flujo-queretaro-aereo-sp01-tablero-guias]]
Cubre: modelo `Guia` + bitácora de estatus, captura manual, tablero con
kanban visual, selección múltiple → creación de referencia(s) agrupada por
régimen, fix del bug de `regimen` en `create()`.

### Sub-plan 02 — Ingesta línea por línea del manifiesto (tarea 14) — ⚪ pendiente de diseñar
Depende de: sub-plan 01 (el modelo `Guia` debe existir primero, para que la
ingesta automática lo alimente en vez de requerir captura manual).

### Sub-plan 03 — Matching Destinatario→Company (tarea 15) — ⚪ pendiente de diseñar
Depende de: sub-plan 02 (necesita los campos reales del manifiesto
ingeridos para tener un "Destinatario" contra el cual matchear).

### Sub-plan 04 — Entidad `Team` y vínculo con `Company` (tarea 16) — ⚪ pendiente de diseñar
Independiente de los anteriores — puede diseñarse/implementarse en
paralelo.

### Sub-plan 05 — Extender ciclo de vida de `UnidentifiedWaybill` + puente con `AirManifest` (tareas 17, 18) — ⚪ pendiente de diseñar
Depende de: reconciliar con el sub-plan 01 — quedan dos "entidades de guía"
coexistiendo (`Guia` nueva vs. `UnidentifiedWaybill`); este sub-plan debe
evaluar explícitamente si conviene o no converger a futuro, sin asumirlo en
silencio.

## Riesgos y side effects transversales

- Dos "entidades de guía" coexistiendo (`Guia` nueva vs.
  `UnidentifiedWaybill`) puede confundir a futuros desarrolladores si no se
  documenta bien la diferencia semántica — dejar comentario claro en el
  modelo Prisma de `Guia` desde el sub-plan 01.
- Estatus kanban del tablero sin validar con el cliente (ver sub-plan 01) —
  alto riesgo de retrabajo temprano en toda la iniciativa.
- El fix del bug de `regimen` no persistido (decisión 9) puede tener
  callers existentes que dependan, sin saberlo, del comportamiento actual —
  revisar antes de corregir (detalle en sub-plan 01).

## Criterios de verificación (globales)

- Cada sub-plan cierra con su propio `/verify` antes de darse por
  terminado.
- Al cerrar el sub-plan 01: confirmar explícitamente que "Por identificar"
  del Inbox sigue funcionando sin regresión (no se tocó su código, pero
  ambas superficies conviven en la misma sección del producto).

## Regla de ramas de esta iniciativa (excepción a la regla general)

**Decisión explícita del usuario (22-jul, al arrancar `/implementa` del
sub-plan 01):** toda la iniciativa Querétaro (aéreo) — todos sus
sub-planes, no solo el 01 — se trabaja **sobre la rama ya existente
`feat/movimientos-dinamicos-por-trafico`** en ambos repos (`carmi-digital`,
`carmi-odin-api-v2`), **sin crear una rama `feat/<slug>` nueva** por
sub-plan. Motivo operativo: al iniciar el sub-plan 01, esa rama ya tenía en
ambos repos una cantidad grande de trabajo sin commitear (feature
`movimientos-dinamicos-por-trafico`) que se decidió commitear tal cual en
vez de aislar el sub-plan en una rama separada.

Por qué importa dejarlo explícito: la regla general de `CLAUDE.local.md`
("Ramas de trabajo por plan") dice ramificar `feat/<slug-del-plan>` al
iniciar cada plan/paraguas. Para **esta iniciativa en particular** esa
regla general queda **reemplazada** por la de este apartado — próximos
`/implementa` de los sub-planes 02-05 (y cualquier retrabajo del propio
sub-plan 01) deben seguir trabajando sobre `feat/movimientos-dinamicos-por-trafico`
en ambos repos, sin crear `feat/2026-07-22-flujo-queretaro-aereo-sp0X-...`.

Antes de commitear código de esta iniciativa en el back
(`carmi-odin-api-v2`), tener en cuenta que su hook de pre-commit corre la
suite completa (`test:cov`) y además un check heurístico de "cobertura de
specs" (cada método público de un `.service.ts`/`.controller.ts` staged
debe aparecer referenciado por nombre en su `.spec.ts`) — este segundo
check ya tenía, al 22-jul, un backlog de ~100 métodos preexistentes del
feature `movimientos-dinamicos-por-trafico` sin cobertura (ajeno a
Querétaro); se saltó una vez con `--no-verify` bajo autorización explícita
del usuario. Si el check vuelve a bloquear un commit de esta iniciativa,
preguntar de nuevo antes de repetir `--no-verify` — no asumir la
autorización como permanente.

## Estado del paraguas

**2026-07-22**: creado. Sub-plan 01 diseñado y listo para `/implementa` en
sesión limpia. Sub-planes 02-05 pendientes de su propia entrevista de
diseño (`/plan`) antes de escribirse.
