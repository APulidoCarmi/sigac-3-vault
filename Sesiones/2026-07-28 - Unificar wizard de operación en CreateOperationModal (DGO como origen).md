# Sesión 2026-07-28 — Unificar wizard de operación en CreateOperationModal (DGO como origen)

Relacionado: [[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]],
[[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]],
[[2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]],
[[2026-07-10-refactor-flujo-ejecutivo-sp15-proforma-pedimento]],
[[2026-07-10-refactor-flujo-ejecutivo-sp19-configuracion-pedimento-en-dgo]]

## Qué se trabajó

`/implementa` completo del plan de 2026-07-28 (17 tareas, ambos repos) que invierte la
arquitectura de creación de operación: `CreateOperationModal.tsx` pasa a ser el único
wizard (antes coexistían dos), alimentado por DGOs, colapsado a un solo pase (1 DGO = 1
pedimento, sin loop multi-grupo). Incluye cableado de cálculo de impuestos, ajuste de
`handleSubmit`, eliminación del wizard de página (4 pasos), selección múltiple de DGOs en
`ReferenceDGOTab.tsx`, proforma a nivel DGO (`GET /dgo/:id/proforma`, filtro `dgoId` en
`preview-taxes`), eliminación del chequeo de discrepancias en `sign()`, y `signedBy` real.
Verificación final con Playwright vía subagente (modelo Haiku).

## Commits relevantes

Ninguno. Diff sin commitear en ambos repos, rama compartida
`feat/unificar-wizard-operacion-dgo-proforma` (`carmi-digital` y `carmi-odin-api-v2`),
para revisión humana tarea por tarea.

## Decisiones (con su porqué)

- **Inversión de arquitectura confirmada por el usuario tras auditoría de código**: los
  sub-planes SP-06/SP-12/SP-15/SP-19 (2026-07-10 a 07-12) habían declarado
  `CreateOperationModal.tsx` fuera de alcance y el wizard de página de 4 pasos como el
  flujo "real". Verificado en código real que ambos wizards seguían vivos en paralelo. El
  usuario decidió invertir: se elimina el wizard de página, `CreateOperationModal.tsx`
  queda como único wizard.
- **1 DGO = 1 pedimento, sin recaptura**: régimen/clave/destino/observaciones se leen de
  solo lectura desde el DGO (ya capturados en `DgoPedimentoForm`, SP-19); aduana y patente
  pasan a ser dato de **operación completa** (antes se asumían per-DGO), capturados una
  sola vez en `Step2OperationConfig.tsx`.
- **Bloqueo de backend real descubierto en la tarea 5**: `POST /operations` en
  `carmi-odin-api-v2` exige `shipments`/`movementInvoices` no vacíos — restricción de
  esquema (`OperationInvoice.operationShipmentId` es `NOT NULL`), no solo un guard de
  código. Crear una operación solo-por-DGO requiere migración de Prisma + nueva rama de
  servicio. **Acordado con el usuario**: se implementó todo lo demás (frontend puro) y se
  agregó un guard explícito en runtime (toast claro, no un TODO en código) que bloquea el
  submit real hasta que ese trabajo de backend se aborde como **sub-plan nuevo vía `/plan`
  en sesión limpia**, al terminar este plan.
- **Proforma por DGO reusa el mismo componente** (`OperationProformaDrawer.tsx`), no se
  duplica — corrige una decisión intermedia del plan original que proponía crear uno
  nuevo. Se adaptó para aceptar `dgoId` (además de `operationId`) y delegar a la proforma
  real cuando el DGO ya está `locked`.
- **Se elimina el chequeo de discrepancias en `sign()`** (no se reubica a ningún otro
  punto del flujo) — decisión explícita del usuario, no un descuido.

## Aprendizajes / errores a no repetir

- **Un plan escrito hace semanas puede tener texto desactualizado respecto al código
  real, incluso dentro de la misma sesión de implementación**: en este plan concreto, el
  texto de la tarea 3 nombraba el archivo equivocado como "el step de agrupar por clave"
  (era `Step2OperationConfig`, que en realidad NO agrupa nada — la lógica real vivía en
  `Step1SourceData` → `MultiShipmentSummary`). La tarea 8 duplicaba trabajo que en
  realidad dependía de la tarea 12 (no escrita aún al momento de redactar la 8). La
  Decisión 7 (auditoría de entry points de `CreateOperationModal`) no había encontrado 2
  entry points reales (`ReferenceInventoryTable.tsx`, `OperationClient.tsx`). En los tres
  casos, la auditoría de código directa (no solo confiar en el texto del plan) fue lo que
  detectó la discrepancia — y en los tres casos se paró a preguntarle al usuario antes de
  decidir en vez de improvisar. Vale la pena seguir aplicando esta disciplina en
  `/implementa` de planes grandes o con varias semanas entre `/plan` e `/implementa`.
- **Un hallazgo de bloqueo de backend real (no solo una limitación de alcance) amerita
  parar la implementación de esa pieza específica y ofrecer opciones concretas al
  usuario**, en vez de mockear/simular el comportamiento esperado. Aquí se optó por
  completar el resto (frontend) y dejar un guard explícito en runtime — nunca un
  `TODO` de código sin resolver (regla dura del proyecto).
- **Verificación Playwright vía subagente con modelo Haiku funciona bien** para flujos de
  UI moderadamente largos (9 pasos, wizard de 3 steps + drawers) cuando el prompt es
  autocontenido: contexto de qué cambió, qué es "correcto", y qué limitación conocida NO
  debe reportarse como bug. Redujo el consumo de contexto del hilo principal.

## Pendientes

- **Sub-plan nuevo** (backend, `carmi-odin-api-v2`): soporte para crear operación
  solo-por-DGO sin `shipments` — migración de Prisma (`OperationInvoice.operationShipmentId`
  nullable o modelo alterno) + `createFromDgos()` + fuente alterna de
  `pesoBruto`/`totalBultos` en `operation-dispatch.service.ts`. Vía `/plan` en sesión
  limpia.
- **Confirmar manualmente el cálculo de impuestos** en el Step 2 del wizard con datos de
  prueba completos (TC, régimen) — la verificación Playwright automatizada recibió un
  toast de validación en vez de un cálculo exitoso, no confirmado si es dato de prueba
  incompleto o un bug real.
- Diff de ambos repos sin commitear, pendiente de revisión humana y decisión de merge.
