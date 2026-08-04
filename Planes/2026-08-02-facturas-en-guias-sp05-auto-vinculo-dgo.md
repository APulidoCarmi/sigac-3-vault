# Plan: Sub-plan 05 — Auto-vincular facturas de Guías al DGO cuando la guía ya tiene Referencia + mostrar la guía de origen en el acordeón

> Sub-plan hijo de [[2026-08-01-facturas-en-guias]]. Repo: `carmi-odin-api-v2`
> (back) y `carmi-digital` (front). Depende de: sub-plan 01 (guiaId en
> Invoice, migración al crear Referencia) y sub-plan 04 (`InvoiceMerchandiseAccordion`
> compartido) — ambos ya completos.

## Contexto

Origen: pedido directo del usuario (sesión 2026-08-02), tras probar el flujo
end-to-end: la guía con folio 1402027826 tiene una factura creada desde el
módulo de Guías (`GuiaInvoicesModal` → `InvoiceFormModal` con `guiaId`). Esa
guía **ya tiene** Referencia/DGO asociado. Al entrar a esa Referencia, el
listado de facturas del DGO no muestra esa factura, y el usuario pide
además que cuando aparezca se indique a qué guía pertenece.

Investigación de código (misma sesión, fork explorador sobre ambos repos)
encontró la causa raíz — **no es el mismo caso ya documentado como fuera de
alcance en el paraguas** (ese hablaba de guías *sin* Referencia todavía).
Aquí la Referencia **ya existe** y aun así no se vincula:

1. **`InvoicesService.create()`** (`invoices.service.ts:80-97`) solo
   auto-resuelve `referenceId` cuando la factura viene con `shipmentId`
   (copia `shipment.referenceId`). No existe la misma rama para `guiaId` —
   cuando se crea una factura con solo `guiaId` y esa guía ya tiene
   `Guia.referenceId` seteado, la factura nace huérfana
   (`referenceId: null`, `dgoId: null`).
2. **La migración automática de la decisión 3 del paraguas**
   (`GuiasService.linkGuiasToReference`, líneas 397-511) solo corre **una
   vez**, en el momento de vincular guías→Referencia por primera vez
   (`prisma.invoice.updateMany({ where: { guiaId: {in}, referenceId: null }, data: { referenceId, dgoId } })`,
   líneas 504-509). Una factura creada **después** de ese evento nunca
   vuelve a disparar esa migración.
3. **Resolución de `dgoId` no es trivial**: una Referencia puede tener
   varios DGOs (`assignRegimenToDefaultDgo`/`createAdditionalWithRegimen`,
   agrupados por `pedimentoCode` — ver `linkGuiasToReference` líneas
   443-480). El modelo `Guia` no guarda un `dgoId` propio.
   - **Descartado** (validación en sesión post-plan, 2026-08-02): resolver el
     Dgo por *match* de `Dgo.clavePedimento == Guia.pedimentoCode` en el
     momento de crear la factura. Se descarta porque **no es una vinculación
     real**: `Dgo.clavePedimento` es `VarChar(2)` y `Guia.pedimentoCode` es
     `VarChar(5)` — anchos distintos, y `linkGuiasToReference` los cruza hoy
     sin normalización visible (riesgo latente ya documentado). El usuario
     pidió una vinculación real en vez de este *match* frágil.
   - **Decisión final**: agregar un campo propio `Guia.dgoId` (FK real a
     `Dgo`) que se puebla en el mismo momento en que
     `linkGuiasToReference` resuelve el Dgo para cada grupo de guías
     (`guias.service.ts:575`, junto al `dgoService.update(defaultDgo.id,
     { regimen, clavePedimento })`). Así, dado un `Guia` con `referenceId`
     ya seteado, el Dgo correspondiente es simplemente `guia.dgoId` — sin
     match por pedimento.
4. **Falta mostrar el origen**: `Invoice.guia` es una relación Prisma real
   (`schema.prisma:1924`, además del escalar `guiaId` en línea 1912), pero
   **ningún** include actual la trae. Validado en código real: `DgoService`
   (`dgo.service.ts:99-136`) arma su **propio include ad hoc** para
   invoices en `listByReference`/`findOne` — **no** usa
   `invoice-item-include.util.ts` (ese solo lo usa `InvoicesService`, no hay
   include compartido entre ambos servicios). El paso de include solo toca
   el include propio de `DgoService`.

Listados afectados en el front (confirmados por la investigación, incl.
validación de código real):
- `ReferenceDGOTab.tsx` consume `GET /dgo/reference/:referenceId`
  (`DgoService.listByReference`) — es el listado que hoy no muestra la
  factura huérfana. Tiene su propio tipo `DgoInvoiceSummary` (id,
  invoiceNumber, totalAmount, `_count.items`), **definido a mano** (no hay
  codegen OpenAPI en el front; no se regenera nada automático al tocar el
  backend, hay que actualizar el tipo TS manualmente).
- `InvoiceMerchandiseAccordion.tsx` (sub-plan 04) recibe `groupedInvoices`,
  pero **cada caller lo arma por separado con su propio fetch/tipo**: no
  hay un include ni tipo común entre `ReferenceMerchandiseTab.tsx:223` y
  `GuiaInvoicesModal.tsx:44-58`. Solo `ReferenceMerchandiseTab.tsx` (modo
  `dgo`) necesita el campo de guía; `GuiaInvoicesModal.tsx` (modo `guia`) no
  se toca porque el badge no aplica ahí — el badge es puramente
  informativo (indicar a qué guía pertenece la factura), sin navegación.

## Decisiones tomadas

1. **Alcance de esta corrección: solo hacia adelante.** No se hace backfill
   de facturas ya huérfanas en producción (p. ej. la de la guía 1402027826
   se queda como está) — decisión explícita del usuario. Este sub-plan solo
   corrige el flujo para que facturas creadas **a partir de ahora** desde
   Guías, sobre una guía que ya tiene Referencia, queden bien vinculadas.
2. **Vinculación real vía `Guia.dgoId`** (revisado 2026-08-02, reemplaza la
   idea original de *match* por `clavePedimento`): se agrega el campo
   `Guia.dgoId String? @db.Uuid` (FK a `Dgo`, migración por Prisma CLI:
   `npx prisma migrate dev --name add-dgo-id-to-guia`).
   `GuiasService.linkGuiasToReference` puebla ese campo para cada guía del
   grupo en el mismo momento en que resuelve/asigna el Dgo
   (`guias.service.ts:575` y alrededores) — análogo al `updateMany` que ya
   hace sobre `Invoice`. En `InvoicesService.create()`: cuando el DTO trae
   `guiaId` (sin `referenceId` explícito), se busca la `Guia`
   (`select: { referenceId: true, dgoId: true }`); si `guia.referenceId`
   existe, se asignan `referenceId` y `dgoId` **directo desde la guía**
   (sin match por pedimento) — mismo patrón que ya existe para
   `shipmentId`. Si `guia.dgoId` es `null` (p. ej. guía vinculada *antes*
   de este cambio, ver Fuera de alcance), la factura se queda con
   `referenceId` seteado pero `dgoId: null` — no se crea un DGO nuevo
   automáticamente en este flujo (eso solo pasa vía
   `linkGuiasToReference`).
3. **Mostrar la guía de origen en dos lugares** (ambos confirmados por el
   usuario):
   - El listado de facturas de `ReferenceDGOTab.tsx` (el que hoy no
     mostraba la huérfana).
   - El acordeón `InvoiceMerchandiseAccordion.tsx` (sub-plan 04): un badge
     junto al nombre de la factura en el trigger de cada `AccordionItem`,
     visible en modo `dgo` (en modo `guia` no aplica: el usuario ya está
     viendo las facturas de esa guía, sería redundante).
   - El badge solo se muestra si la factura tiene `guiaId` (no todas las
     facturas de un DGO vienen de Guías); formato sugerido "Guía:
     {guiaMaster o guiaHouse}", sin navegación/click (fuera de alcance salvo
     que la implementación lo justifique trivialmente).
4. **No se auto-crea DGO ni se reasigna régimen** en este flujo — la
   creación/gestión de DGOs sigue siendo exclusiva de
   `linkGuiasToReference`/`DgoService`. Este sub-plan solo *vincula* una
   factura ya existente a un DGO ya existente.

## Fuera de alcance

- Backfill de facturas ya huérfanas existentes en producción (decisión 1).
- **Backfill de `Guia.dgoId`** para guías vinculadas *antes* de este cambio
  (ya tienen `referenceId` pero el campo nuevo nacerá `null` para ellas) —
  mismo criterio "solo hacia adelante" de la decisión 1: no se recalculan
  guías ya vinculadas, solo se puebla `dgoId` desde ahora en
  `linkGuiasToReference`. Si una factura se crea sobre una de esas guías
  "viejas", queda con `referenceId` seteado y `dgoId: null` (mismo caso que
  el punto anterior de la decisión 2).
- Caso de guías **sin** Referencia todavía (pre-referencia) — ya documentado
  como riesgo aceptado del paraguas; sigue igual, la factura se queda
  huérfana hasta que se cree la Referencia (flujo ya cubierto por
  `linkGuiasToReference`).
- Auto-creación de un DGO nuevo cuando `guia.dgoId` es `null` (decisión 2,
  caso borde).
- Navegación/click desde el badge de guía hacia el módulo de Guías (badge
  puramente informativo, confirmado por el usuario).
- Cualquier cambio al comportamiento de `linkGuiasToReference` en sí, más
  allá de agregar el `updateMany`/asignación de `Guia.dgoId`.

## Pasos

- [x] **Backend — schema**: agregar `Guia.dgoId String? @db.Uuid` (relación
      a `Dgo`) a `schema.prisma` y generar la migración con
      `npx prisma migrate dev --name add-dgo-id-to-guia` (prohibido escribir
      el SQL de migración a mano).
- [x] **Backend**: en `GuiasService.linkGuiasToReference`, junto a la
      resolución del Dgo por grupo (`assignRegimenToDefaultDgo`/
      `createAdditionalWithRegimen`, alrededor de `guias.service.ts:575`),
      agregar el `updateMany` (o incluirlo en el `data` de la asignación ya
      existente) que setea `Guia.dgoId` para cada guía del grupo con el Dgo
      resuelto. Ajustar/agregar specs de `guias.service.spec.ts` cubriendo
      que las guías quedan con `dgoId` correcto tras vincular.
- [x] **Backend**: en `InvoicesService.create()`, agregar una rama análoga a
      la de `shipmentId` (`invoices.service.ts:80-104`): cuando `dto.guiaId`
      está presente y no viene `referenceId` explícito, buscar el `Guia`
      por id (`select: { referenceId: true, dgoId: true }`); si
      `guia.referenceId` existe, asignar `referenceId` y `dgoId` (el que
      tenga la guía, puede ser `null`) al `data` de creación de la factura
      — sin match por pedimento. Actualizar/agregar specs de
      `invoices.service.spec.ts` cubriendo: guía sin referencia (sin cambio
      de comportamiento), guía con referencia y `dgoId` seteado (asigna
      ambos), guía con referencia y `dgoId: null` (asigna solo
      `referenceId`).
- [x] **Backend**: agregar la relación `guia` (`select: { id, guiaMaster,
      guiaHouse }` o equivalente) al include propio de invoices que arma
      `DgoService.listByReference`/`findOne` (`dgo.service.ts:99-136`) —
      **no** tocar `invoice-item-include.util.ts`, que es un include
      separado usado solo por `InvoicesService` y no aplica aquí (validado
      en código real). Confirmar que no se infla el payload de otros
      consumidores del mismo include ad hoc de `DgoService`.
- [x] **Frontend — `ReferenceDGOTab.tsx`**: extender `DgoInvoiceSummary`
      (tipo definido a mano, sin codegen) con el campo de guía y renderizar
      el badge "Guía: {guiaMaster || guiaHouse}" junto a cada factura del
      listado, solo cuando la factura tenga guía asociada.
- [x] **Frontend — `ReferenceMerchandiseTab.tsx` + `InvoiceMerchandiseAccordion.tsx`**:
      `ReferenceMerchandiseTab.tsx:223` es el único caller que necesita el
      campo de guía (modo `dgo`); extender ahí el tipo/fetch que arma
      `groupedInvoices` y el tipo `InvoiceMerchandiseGroup`/
      `InvoiceMerchandiseItem` compartido con la info de guía, y renderizar
      el badge informativo (sin click/navegación) en el trigger de cada
      `AccordionItem`, solo en `context="dgo"` y solo si la factura tiene
      guía. **No** tocar `GuiaInvoicesModal.tsx` (modo `guia`): no arma el
      campo y el badge no aplica ahí (ya se sabe de qué guía son todas).

## Riesgos y side effects a vigilar

- El include ad hoc de invoices en `DgoService.listByReference`/`findOne`
  puede tener otros consumidores dentro de `DgoService` — confirmar que
  agregar `guia` no rompe serialización ni infla innecesariamente
  respuestas que no la necesitan.
- Guías vinculadas *antes* de este cambio quedan con `Guia.dgoId: null`
  (no hay backfill, ver Fuera de alcance) — una factura creada sobre una de
  esas guías "viejas" seguirá naciendo con `dgoId: null` aunque tenga
  `referenceId`. Esto es esperado, no un bug de esta implementación.
- El DTO ya permite mandar `guiaId` + `referenceId` explícito al mismo
  tiempo (`assertHasAnchor` no es exclusivo) — confirmar que el flujo de
  `shipmentId` (ya existente) y el nuevo de `guiaId` no interactúan mal si
  por error llegan ambos, o si llega `guiaId` junto con `referenceId`
  explícito (en ese caso la nueva rama no debe pisar el `referenceId`
  explícito del DTO).

## Criterios de verificación

- `/verify`: lint/typecheck de front y back; migración de Prisma generada
  por CLI (no a mano); specs de backend ajustados (`guias.service.spec.ts`
  para `Guia.dgoId`, y los 3 casos de `invoices.service.spec.ts`).
  **Completado 2026-08-02**: lint (0 errores nuevos), `tsc --noEmit` (0
  errores nuevos) y suite completa de backend (543/543 suites, 4022/4077
  tests, 55 `it.todo` sin ejecutar) en verde.
- Playwright: crear una guía nueva, vincularla a una Referencia/DGO
  existente (o crear una nueva referencia con esa guía), luego crear una
  factura desde `GuiaInvoicesModal` sobre esa misma guía — confirmar que la
  factura aparece de inmediato en `ReferenceDGOTab` y en el acordeón de
  `ReferenceMerchandiseTab`, con el badge de guía visible en ambos lugares.
  **Completado 2026-08-02**: guía nueva (7954998763) → régimen/pedimento
  A1 asignado → "Crear referencia desde la selección" → REF-00137 → factura
  SP05-VERIFY-003 creada desde `GuiaInvoicesModal`. Confirmado por API
  (`referenceId`/`dgoId` poblados) y visualmente en ambos lugares: badge
  "Guía: 78350184643" en `ReferenceDGOTab` y en el trigger del acordeón de
  `ReferenceMerchandiseTab` (`InvoiceMerchandiseAccordion`, context="dgo").
  **Ajuste post-verificación** (feedback del usuario 2026-08-02): el badge
  de `ReferenceDGOTab` se agregó primero como bloque nuevo "Facturas del
  DGO", duplicando el listado ya existente de `DgoComparisonPanel`
  (`GET /dgo/:id/comparison`, con botón "Editar factura"). Se movió el
  badge a ese listado real (única fuente visible de facturas del DGO en
  pantalla) y se eliminó el bloque duplicado — `DgoComparisonPanel` ahora
  recibe `invoices={dgo.invoices}` como prop y resuelve el `guia` por
  `invoiceId` para renderizar el badge junto al número de factura.
- Regresión: confirmar que una factura creada directamente desde
  Referencia/DGO (sin `guiaId`) no muestra badge de guía, y que el flujo de
  `shipmentId` sigue funcionando igual que antes.

## Hallazgo de verificación (bug preexistente, fuera del alcance original)

Durante la verificación Playwright del criterio anterior, `POST
/guias/link-to-reference` devolvía **500** al vincular cualquier guía con
al menos un grupo de pedimento (afecta también "Crear referencia desde la
selección"). **Causa raíz** (no introducida por este sub-plan — verificado
reproduciendo el error con la adición de `Guia.dgoId` removida
temporalmente, y el 500 persistió igual): el `$transaction([...])` en
forma de arreglo de `linkGuiasToReference` incluye
`this.prisma.invoice.updateMany(...)` (migración de facturas huérfanas,
sub-plan 01). `Invoice` tiene trigger de auditoría
(`AUDITED_MODELS`/`pg_trigger`), así que `AuditedPrismaService` envuelve
esa llamada en `runAudited(...)` cuando se construye fuera de un tx activo
→ devuelve una `Promise` normal, no el `PrismaPromise` perezoso que Prisma
exige para la forma de arreglo → Prisma rechaza **todo** el arreglo con
`"All elements of the array need to be Prisma Client promises"`.

**Decisión (autorizada por el usuario 2026-08-02, fuera del alcance
declarado de este sub-plan pero corregido en la misma sesión):** se
convirtió el `$transaction([...])` de `linkGuiasToReference` a la forma
callback `runAudited(this.prisma, async tx => { ...awaits secuenciales... })`,
el mismo patrón ya usado en `remove()`/`restore()` de este archivo (que sí
soporta modelos auditados). Verificado: 34/34 tests de
`guias.service.spec.ts` en verde, suite completa de backend en verde, y
`POST /guias/link-to-reference` responde 201 con `Guia.dgoId` poblado
correctamente tras el fix.

## Cierre (2026-08-03)

Sub-plan 05 completo: 6/6 tareas implementadas y verificadas (ver bitácora
[[2026-08-03 - Implementación sp05 auto-vínculo DGO y fixes UI factura]]).
Diff sin commitear en ambos repos, pendiente de revisión humana. Sesión
continuó con fixes de UI adicionales sobre el documento oficial de factura
(fuera del alcance de este plan, documentados en la misma bitácora):
rediseño de `InvoiceDocumentUpload.tsx`, vista previa inline del histórico,
fix de `pointer-events` pegado y overlay doble en diálogos anidados, fix de
`<button>` anidado en `InvoiceMerchandiseAccordion.tsx`.
