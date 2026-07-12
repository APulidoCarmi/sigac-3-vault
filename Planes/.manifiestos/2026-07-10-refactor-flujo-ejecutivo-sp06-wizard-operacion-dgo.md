# Manifiesto — SP-06: Wizard de Crear/Editar Operación → Step 0 "Seleccionar DGO(s)"

**Estado: ✅ IMPLEMENTADO (2026-07-12), tercer paso de este sub-plan.** El primer intento
(abajo, sección "Intento 1 — histórico") se detuvo en fase de diseño porque el D1 original
citaba código que no existía; un replanteo posterior verificó el código real y amplió el
D1 (ver `Planes/2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo.md`,
sección "D1 corregido y ampliado"); el usuario resolvió las decisiones de producto
pendientes ("Decisiones finales" del sub-plan, 2026-07-12); esta sesión implementó las 11
tareas de la sección "## Tareas" en ambos repos. Se conserva el rastro del intento 1 más
abajo, sin editarlo, para no perder el diagnóstico original.

## Resumen de lo implementado

### Backend (`carmi-odin-api-v2`, rama `refactor/customs-operation-sp06`)

- **Migración de esquema** (CLI, `npx prisma migrate dev --name
  add-dgo-id-to-operation-pedimento`): `20260712034718_add_dgo_id_to_operation_pedimento`
  agrega `OperationPedimento.dgoId` (FK opcional → `Dgo`, `onDelete: SetNull`) + relación
  inversa `Dgo.operationPedimentos`. `npx prisma migrate status` confirmó estado limpio
  antes de generar. **`Operation.referenceId` se mantiene NOT NULL/singular** (decisión
  documentada, no un ajuste mecánico): cambiarlo a opcional/derivado tocaría todos los
  consumidores existentes de `operation.reference` (proforma, sync de expediente,
  permisos, índices) fuera del alcance de este sub-plan; la relación multi-referencia
  *real* vive ahora en `OperationPedimento.dgoId → Dgo.referenceId` — cada `Pedimento`
  de una operación se conecta a la referencia real de su propio DGO, no a la
  `primaryReferenceId` de la operación.
- `prisma/schema.prisma`: `OperationPedimento.dgoId` + `Dgo.operationPedimentos`.
- `src/dgo/dtos/dgo.dto.ts`: `SelectableDgoQueryDto`, `ValidateDgoSelectionDto` (nuevos).
- `src/dgo/services/dgo.service.ts`:
  - `listSelectableReferences(query)` — referencias paginadas con sus DGOs firmados y sin
    pedimento vinculado (fuente de `GET /dgo/selectable`).
  - `validateHomogeneousRegimen(dgoIds)` — valida existencia, `status === 'SIGNED'`, que
    ninguno esté ya vinculado a un pedimento, y régimen aduanero idéntico y no-nulo entre
    todos; devuelve los DGOs (con su `reference`) para que el llamador derive la
    referencia real de cada pedimento.
  - `assertNotLocked(id)` (privado) — bloquea mutaciones sobre un DGO ya vinculado a un
    pedimento; aplicado en `update()` y `splitByClave()` (este último también valida los
    DGOs de origen/destino de las facturas reasignadas).
- `src/dgo/controllers/dgo.controller.ts`: `GET /dgo/selectable` (antes de `GET /dgo/:id`
  para no chocar con el parámetro de ruta), `POST /dgo/validate-selection`.
- `src/operations/operations.module.ts`: importa `DgoModule` y `ReferenceDocumentsModule`
  (sin `forwardRef`; no hay ciclo — ninguno de los dos importa `OperationsModule`).
- `src/operations/dtos/create-operation.dto.ts`: `PedimentoGroupDto.dgoId?` (nuevo,
  opcional por retrocompatibilidad con el flujo legacy de 1 pedimento).
- `src/operations/services/operations.service.ts` (`create()`):
  - Si algún grupo trae `dgoId`: valida régimen homogéneo + no-bloqueo
    (`dgoService.validateHomogeneousRegimen`) y, por cada referencia real involucrada,
    el bloqueo de glosa (`referenceDocumentsService.assertInvoicesGlossedOk`) — defensa
    en profundidad además de la validación que ya hace el frontend en el Step 0.
  - Cada `Pedimento` del loop se conecta a la referencia real de su DGO (`dgo.referenceId`)
    en vez de siempre `primaryReferenceId`; `OperationPedimento.dgoId` se persiste.
  - Grupo sintético legacy (sin `dto.pedimentos`) sigue funcionando igual (fallback).
- Tests: `src/dgo/services/dgo.service.spec.ts` (+8 casos: lock en `update`,
  `validateHomogeneousRegimen` x4, `listSelectableReferences`),
  `src/operations/services/operations.service.spec.ts` (mocks de `DgoService`/
  `ReferenceDocumentsService` agregados a la construcción del `TestingModule`).

### Frontend (`carmi-digital`, rama `refactor/customs-operation-sp06`)

- **Nuevo** `lib/api/modules/dgo.ts`: `dgoServices.getSelectableDgos`,
  `validateDgoSelection`, `getDgoById` (wrappers `apiOdinClient`, mismo patrón que
  `customs-operation.ts`).
- **Nuevo** `lib/api/modules/reference-documents.ts`: `referenceDocumentsServices.
  canStartOperation` (no existía ningún wrapper para este endpoint de SP-04).
- `lib/api/modules/customs-operation.ts`: `CreateOperationDto` acepta `referenceIds[]` y
  `pedimentos[]` (con `dgoId`); nuevo `customsOperationServices.previewTaxes` (wrapper de
  `POST /operations/preview-taxes`, no existía).
- `stores/create-operation.store.ts`: tipo `SelectedDgo` nuevo; `state.selectedDgos` +
  `setSelectedDgos`; `pedimentoGroups[].dgoId`/`.referenceId` nuevos (opcionales);
  `initializeGroupsFromDgos()` — arma 1 grupo por DGO seleccionado, con `formData.
  customsData` prellenado desde ESE DGO (régimen/clave/aduana). `closeModal` limpia
  `selectedDgos`. Tests nuevos en `stores/__tests__/create-operation.store.spec.ts`.
- **Nuevo** `app/(customerPortal)/customs-operation/createOperation/components/
  StepDgoSelection.tsx`: Step 0 real (reemplaza el placeholder). Acordeón de referencias
  → DGOs seleccionables (checkbox), patrón visual adaptado de `ReferenceDGOTab.tsx`.
  Valida en vivo (feedback visual) régimen homogéneo + glosa por referencia mientras el
  usuario selecciona.
- `page.tsx`: `validateStep(0)` ahora es async — revalida (autoritativo) régimen
  homogéneo + glosa, resuelve `invoiceIds` reales de cada DGO (`getDgoById`), fija
  `selectedReference`/`selectedReferences` (retrocompat con los steps 1-3, que siguen
  leyendo el singular) y llama `initializeGroupsFromDgos()`. `handleFinish()` construye
  `referenceIds[]` (únicas, derivadas de `selectedDgos`) y `pedimentos[]` (uno por grupo,
  con `dgoId`, régimen/clave/aduana propios del grupo, resto de campos de despacho
  compartidos). El resumen final muestra los DGOs seleccionados (referencia + `dgoNumber`
  + régimen).
- `stepCustomsInfo.tsx`: nueva sección "Cálculo de Impuestos (TaxEngine)" en la pestaña
  "Formas de Pago" — botón que llama `previewTaxes` y muestra el desglose (IGI/IVA/DTA/
  PRV/CNT + total), usando `isCalculatingTaxes`/`CustomsData.taxCalculation` ya
  modelados en el store; badges con referencia + `dgoNumber` en la pestaña "Aduanas".
- **Limpieza de código muerto** (verificado por grep que nada fuera de estos archivos los
  importaba, salvo referencias a un archivo homónimo no relacionado en otra feature):
  `context/PedimentoWizardContext.tsx`, `components/StepInventorySelection.tsx`,
  `components/StepIncrementables.tsx`, `components/StepPartidasTable.tsx`,
  `components/StepPedimentoHeader.tsx`, `components/reference-selection/` (carpeta
  completa) y `hooks/` (carpeta completa: `useReferencesData.ts`,
  `usePreselectedReference.ts`, `useItemAvailability.ts`, `useExpandedState.ts`,
  `index.ts`).

## Limitaciones y desviaciones documentadas

1. **Prellenado por-grupo parcial (punto 4 de "Decisiones finales").** El wizard
   in-scope tiene 4 pasos (no el wizard "0) Selector, 1) Grouping, 2) Operation Config,
   3-5) Per-pedimento" que los comentarios del store describen como diseño original no
   terminado de cablear). Implementar un loop completo de pasos por-pedimento es un
   rediseño de UI mayor. Se optó por una interpretación honesta y acotada: los campos que
   SÍ difieren por DGO (régimen, clave de pedimento, aduana) viajan por grupo desde
   `initializeGroupsFromDgos` y se preservan en `handleFinish()`; el resto de campos de
   despacho (fechas, agente, transporte, identificadores, notas) se siguen capturando una
   sola vez a nivel operación, consistente con el propio comentario del store sobre
   `OperationLevelData` ("shared across all pedimentos, captured once"). Si el negocio
   necesita edición por-pedimento de esos campos compartidos, es un sub-plan aparte.
2. **`dgoNumber` alcanza sin ajuste** (punto 7) — no se creó un campo/folio nuevo.
3. **Bloqueo de edición del DGO (punto 6)**: implementado a nivel backend
   (`DgoService.assertNotLocked`, sin respaldo en meets, documentado como decisión de
   producto del usuario 2026-07-12 en el propio sub-plan). No se agregó UI específica de
   "DGO bloqueado" más allá de que las pantallas de gestión de DGO recibirán el error 400
   si se intenta editar — deshabilitar visualmente los controles de edición en
   `ReferenceDGOTab.tsx` para un DGO ya vinculado queda como mejora de UX menor, no crítica
   (el guard de backend es la fuente de verdad y ya bloquea).
4. **`GET /dgo/selectable` no filtra por `clientCompanyId` del usuario actual en
   `StepDgoSelection.tsx`** — el wrapper lo soporta (`dgoServices.getSelectableDgos({
   clientCompanyId })`) pero el componente no tiene hoy una fuente confiable del
   `clientCompanyId` activo en este contexto del wizard (se crea antes de escoger cliente
   explícito) — el filtro de "solo mis DGOs" queda pendiente de a qué contexto de sesión
   engancharlo; documentado, no improvisado.
5. **`next build` (Turbopack) no pudo completarse** por un problema **preexistente y no
   relacionado**: los paquetes `three` y `mammoth` están en `package.json` pero no se
   resuelven en `node_modules` (mismo error ya lo reportaba `tsc --noEmit` antes de
   tocar cualquier archivo de este sub-plan, en `features/locations/warehouse/
   components/Trailer3D.tsx` y `features/zeus-shell/components/Renderers/
   DocxRenderer.tsx`, módulos no relacionados con `customs-operation`). Verificación
   usada en su lugar: `npx tsc --noEmit` (sin errores nuevos en los archivos tocados) +
   `npx eslint` (0 errores, solo warnings preexistentes de estilo `any`) + `npx jest`
   (todos los tests de store pasan). Recomendado: `pnpm install` completo en un entorno
   limpio antes de intentar `next build`/Playwright.

## Verificación

- **Backend**: `npx prisma migrate status` (limpio antes de migrar) → `npx prisma
  migrate dev --name add-dgo-id-to-operation-pedimento` (aplicada) → `npx nest build`
  (verde) → `npx eslint` sobre los archivos tocados (0 errores, solo 2 warnings
  preexistentes sin relación) → `npx jest` sobre `src/dgo`, `src/operations`,
  `src/reference-documents` (7 suites, 53 tests, todos verdes).
- **Frontend**: `npx tsc --noEmit` (sin errores nuevos) → `npx eslint` sobre los archivos
  tocados (0 errores, solo warnings preexistentes de estilo) → `npx jest
  stores/__tests__/create-operation.store.spec.ts` (11 tests verdes) → `next build`
  bloqueado por el problema preexistente de dependencias (ver arriba).
- **Pendiente de sesión humana**: validación end-to-end con Playwright MCP (crear una
  operación real seleccionando DGOs de 2+ referencias con régimen homogéneo → verificar
  N pedimentos, impuestos calculados, bloqueos de glosa/régimen/DGO-ya-vinculado, sin
  errores de consola) — no se pudo levantar el entorno dev en esta sesión (sin `dev_url`
  configurado para este cliente y sin poder resolver el bloqueo de `next build` de arriba
  dentro del alcance de este sub-plan).

## Archivos tocados

**Backend** (`carmi-odin-api-v2`): `prisma/schema.prisma`,
`prisma/migrations/20260712034718_add_dgo_id_to_operation_pedimento/`,
`src/dgo/dtos/dgo.dto.ts`, `src/dgo/services/dgo.service.ts`,
`src/dgo/services/dgo.service.spec.ts`, `src/dgo/controllers/dgo.controller.ts`,
`src/operations/operations.module.ts`, `src/operations/dtos/create-operation.dto.ts`,
`src/operations/services/operations.service.ts`,
`src/operations/services/operations.service.spec.ts`.

**Frontend** (`carmi-digital`): `lib/api/modules/dgo.ts` (nuevo),
`lib/api/modules/reference-documents.ts` (nuevo), `lib/api/modules/customs-operation.ts`,
`stores/create-operation.store.ts`, `stores/__tests__/create-operation.store.spec.ts`,
`app/(customerPortal)/customs-operation/createOperation/components/StepDgoSelection.tsx`
(nuevo), `app/(customerPortal)/customs-operation/createOperation/page.tsx`,
`app/(customerPortal)/customs-operation/createOperation/components/stepCustomsInfo.tsx`;
eliminados: `context/PedimentoWizardContext.tsx`,
`components/StepInventorySelection.tsx`, `components/StepIncrementables.tsx`,
`components/StepPartidasTable.tsx`, `components/StepPedimentoHeader.tsx`,
`components/reference-selection/*`, `hooks/*`.

---

## Intento 1 — histórico (2026-07-10, bloqueado en fase de diseño)

**Estado en ese momento: 🚧 BLOQUEADO — el plan describe un wizard que no es el que existe en el
código. Se detiene la implementación (regla dura de `/implementa`: "si descubres algo
que invalida el plan, detente y repórtalo, no improvises un diseño distinto") y se
propone actualizar el sub-plan antes de escribir código.**

## Ramas
- `carmi-digital`: se creó `refactor/customs-operation-sp06` (encadenada desde
  `refactor/customs-operation-sp05`, working tree previo intacto, nada comiteado).
- `carmi-odin-api-v2`: no se tocó (no se creó rama sp06 porque, según el análisis, este
  sub-plan tal como está escrito no requiere cambios de backend — ver más abajo). Sigue
  en `refactor/customs-operation-sp05`.

## Qué se investigó (antes de tocar código, siguiendo el paso 2 de `/implementa`)
Se leyó completo el wizard IN-SCOPE `app/(customerPortal)/customs-operation/createOperation/`:
`page.tsx`, `context/PedimentoWizardContext.tsx`, `components/StepInventorySelection.tsx`,
`components/stepOperationBasicData.tsx`, `stores/create-operation.store.ts`, y se
verificó (grep + lectura) el backend `src/dgo/**` (SP-05) y `src/operations/controllers/operations.controller.ts`.

## Hallazgos que invalidan el D1 del plan

1. **El Step 0 real de `page.tsx` no es `StepInventorySelection`.** El wizard vivo
   renderiza en `currentStepIndex === 0` un `<div>` placeholder literal: *"Reference
   selection UI needs to be implemented for single-reference flow"* (`page.tsx:429-433`).
   No hay ningún `import` de `StepInventorySelection` en `page.tsx`.

2. **`PedimentoWizardContext.tsx` / `StepInventorySelection.tsx` / `StepIncrementables.tsx`
   / `StepPartidasTable.tsx` / `StepPedimentoHeader.tsx` son código huérfano ("FASE
   5.3").** `grep -rln "PedimentoWizardProvider"` en todo el repo solo encuentra el
   propio archivo que lo define — `PedimentoWizardProvider` **nunca se monta** en ningún
   layout ni página. El wizard real usa exclusivamente `useCreateOperationStore`
   (zustand, `stores/create-operation.store.ts`). Este grupo de archivos usa datos mock
   (`mockPedimentoId = \`ped-draft-${Date.now()}\``) y llamadas a fetch comentadas que
   nunca se conectaron al store real. El D1 del plan los lista como "Reusa" — no se puede
   reusar código que no está en el flujo ejecutado.

3. **La línea "reactivar el cálculo de impuestos… `PedimentoWizardContext.tsx:191`
   (`/api/pedimentos/calculate-reserve`)" es un dato erróneo del plan.** Esa línea
   comentada es una llamada mock a `calculateAndReserveSubdivision` (reserva de
   inventario, Paso 2 del wizard huérfano), no tiene relación con impuestos, y vive en el
   mismo archivo muerto del hallazgo #2.

4. **El cálculo de impuestos real (TaxEngine) no existe en ningún archivo in-scope.**
   Se ubica por completo en el módulo **fuera de alcance** `components/operations/**`:
   `Step3CustomsHeader.tsx` (llama `POST /operations/preview-taxes` con debounce 500ms),
   `Step2FiscalConfig.tsx`, `Step4PaymentForms.tsx`, `TaxCalculationCard.tsx`. El store
   `create-operation.store.ts` sí tiene un campo `customsData.taxCalculation` y un
   passthrough (`state.customsData.taxCalculation = operationData.taxesCalculated`,
   línea 749) pero **nada en el wizard in-scope dispara ese cálculo** — no hay una
   llamada comentada que "reactivar"; sería wiring nuevo, tomando como referencia un
   patrón que vive en el módulo declarado fuera de alcance del refactor completo.

5. **El store real es de una sola referencia, no multi-DGO.** `create-operation.store.ts`
   modela `selectedReference: SelectedReference` (singular) y `sourceShipment` (singular)
   como el flujo activo ("Single reference flow — simplified shipment structure",
   comentario propio del archivo), aunque `OperationBasicData.referenceIds?: string[]`
   sugiere un scaffold multi-referencia abandonado a medias. Construir "seleccionar 2+
   DGOs → 2+ pedimentos" exige decidir de fondo cómo cambia esta forma (¿la operación
   sigue atada a 1 referencia con N DGOs de esa referencia, o puede abarcar DGOs de
   varias referencias?) — es una decisión de diseño de producto, no un ajuste mecánico.

6. **Backend DGO (SP-05) confirma la limitación ya documentada por SP-05**: `GET
   /dgo/reference/:referenceId` solo lista DGOs de **una** referencia (no hay endpoint
   para listar/seleccionar DGOs entre referencias). Y las consultas de Mercancía/Gastos/
   Identificadores (usadas por `stepOperationBasicData.tsx` vía `/operations/prepare/:id`,
   que a su vez llama a `operationsService.prepare(referenceIds, shipmentId)`) siguen
   filtrando por `referenceId`, no por `dgoId` — la limitación heredada de SP-05 que el
   prompt orquestador anticipó. Confirmado, no arreglado (fuera de mi alcance declarado).

## Por qué no se improvisó una solución
Los 4 pasos del plan ("cambiar Step 0 a DGOs", "prellenar desde DGO", "reactivar
impuestos", "soportar multi-DGO→multi-pedimento") requieren, dado lo anterior, diseñar
de cero: (a) qué reemplaza al placeholder del Step 0 (no existe un componente previo que
adaptar), (b) cómo el store pasa de single-reference a N-DGO (posiblemente N-referencia),
(c) cómo se dispara `preview-taxes` desde el wizard in-scope sin tocar el módulo
`operations` declarado fuera de alcance del refactor, y (d) cómo el payload de
`POST /operations` (que hoy es 1 `referenceId` + 1 array `shipments`) se convierte en
N pedimentos. Esto es rediseño de arquitectura, no "ajustar fuente de datos" como dice el
D1. Escribir código sobre esa base habría sido inventar un diseño no revisado por el
humano, exactamente lo que la regla dura de `/implementa` prohíbe.

## Recomendación
Regresar SP-06 a `/plan` para una entrevista corta que resuelva, con datos reales del
código (no los asumidos en el D1 actual):
- Confirmar que el Step 0 se construye desde cero (no hay componente previo que adaptar).
- Definir el modelo de datos del store para multi-DGO (¿multi-referencia también?).
- Decidir el mecanismo de cálculo de impuestos in-scope: ¿llamar `preview-taxes`
  directamente desde un componente nuevo del wizard in-scope (permitido: es consumir un
  endpoint existente, no tocar el módulo `operations`), o es necesario un endpoint nuevo?
- Confirmar si `POST /operations` acepta ya N pedimentos por operación o si eso también
  requiere un cambio de contrato (backend), lo cual añadiría un repo y una rama más al
  alcance del sub-plan.

## Archivos tocados
Ninguno de código de producto. Solo este manifiesto y la rama
`refactor/customs-operation-sp06` creada en `carmi-digital` (vacía, sin commits, working
tree previo de sp05 intacto).

## Pasos del sub-plan
Los 4 `- [ ]` **no se marcan como hechos** — no se implementó nada, para no dejar código
especulativo o parches rápidos sobre una base que el propio análisis muestra que no
aplica tal como está descrita.

## Verificación
No aplica todavía (no hay diff de producto que verificar). Gate estático y Playwright
quedan pendientes de la siguiente iteración, una vez el plan se actualice.
