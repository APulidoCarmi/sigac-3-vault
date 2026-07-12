# Sub-plan SP-06: Wizard de Crear/Editar Operación → Step 0 "Seleccionar DGO(s)"

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #21 del [[Inventario_Pantallas_v3]] (🟡 cambiar Step 0). La Operación
**agrupa** uno o más DGO/pedimentos — no los crea ni transforma. Origen:
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b: la operación
nace de entrada/subdivisión, refinado luego a "desde el DGO"), glosario
"Operación"/"DGO" del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].

**Decisión de la entrevista (aclara una ambigüedad del Documento):** el refactor
opera sobre el wizard **IN-SCOPE de 4 pasos** `customs-operation/createOperation/page.tsx`,
NO sobre el de 6 pasos `components/operations/CreateOperationModal.tsx` (ese está en
`components/operations` = fuera de alcance; el Documento lo describía por error).

## Historial de esta sección (no borrar el rastro)

- **2026-07-10, primer intento de `/implementa`:** se detuvo en fase de diseño
  porque el D1 original (abajo, ver bloque "❌ D1 original — INVALIDADO") citaba
  archivos y mecanismos que no correspondían al código real. Diagnóstico completo en
  el manifiesto `Planes/.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo.md`.
- **2026-07-11, replanteo (este documento):** se verificó cada hallazgo del manifiesto
  leyendo el código fuente directamente (no solo el manifiesto) en
  `carmi-digital@refactor/customs-operation-sp06` y `carmi-odin-api-v2@refactor/customs-operation-sp05`,
  y se amplió el D1 con hallazgos nuevos que el manifiesto no había cubierto (ver
  "D1 corregido y ampliado"). Se redactan tareas ajustadas a la realidad, pero el
  sub-plan **NO** queda listo para implementar: hay decisiones de producto pendientes
  (sección final).

### ❌ D1 original — INVALIDADO (se conserva para trazabilidad, NO USAR)

> - **Reusa:** `customs-operation/createOperation/page.tsx:18-40` (4 pasos: Referencias
>   → Datos Básicos → Transporte → Despacho); store `stores/create-operation.store.ts`;
>   cuerpos `createOperation/components/StepInventorySelection.tsx`, `stepOperationBasicData.tsx`
>   (`GET /operations/prepare`), `stepTransportConfig.tsx`, `stepCustomsInfo.tsx`,
>   `StepIncrementables.tsx`, `StepPartidasTable.tsx`, `StepPedimentoHeader.tsx`; context
>   `createOperation/context/PedimentoWizardContext.tsx`. APIs `GET /operations/prepare`,
>   `POST /operations` (`lib/api/modules/customs-operation.ts:193`), `PATCH /operations/:id`.
> - **Refactoriza:** Step 0 (`StepInventorySelection`) → "Seleccionar DGO(s)". Reactivar
>   cálculo de impuestos comentado en `PedimentoWizardContext.tsx:191`
>   (`/api/pedimentos/calculate-reserve`). Prellenar desde DGO.
> - **Crea:** nada estructural; ajustar fuente de datos (de movimientos → DGOs).

**Por qué es inválido (verificado línea por línea contra el código, no solo contra el manifiesto):**
1. `page.tsx` (líneas 429-433) renderiza en Step 0 un `<div>` placeholder literal
   ("Reference selection UI needs to be implemented for single-reference flow"),
   con un comentario explícito `// StepReferencesSelection removed - incompatible
   with single-reference store architecture` (línea 8). No importa `StepInventorySelection`.
2. `PedimentoWizardContext.tsx`, `StepInventorySelection.tsx`, `StepIncrementables.tsx`,
   `StepPartidasTable.tsx`, `StepPedimentoHeader.tsx` son código huérfano ("FASE 5.3"):
   `PedimentoWizardProvider` no se monta en ningún layout/página (`grep -rln
   "PedimentoWizardProvider"` solo encuentra el propio archivo que lo define).
3. La línea `PedimentoWizardContext.tsx:191` es una llamada mock comentada a
   `calculateAndReserveSubdivision` (reserva de inventario), no impuestos.
4. El TaxEngine real (`POST /operations/preview-taxes`) solo se consume desde
   `components/operations/steps/Step3CustomsHeader.tsx` (módulo fuera de alcance);
   no hay wrapper en `lib/api/modules/customs-operation.ts` ni en ningún archivo
   in-scope.

## D1 corregido y ampliado (verificado 2026-07-11 leyendo código fuente directo)

### A. Wizard in-scope real
- `app/(customerPortal)/customs-operation/createOperation/page.tsx`: 4 pasos
  (Referencias/Step 0, Datos Básicos, Transporte, Despacho). Step 0 es un
  **placeholder sin implementar** (línea 429-433). Los pasos 1-3 sí funcionan y
  leen/escriben del store zustand `stores/create-operation.store.ts` vía
  `stepOperationBasicData.tsx`, `stepTransportConfig.tsx`, `stepCustomsInfo.tsx`
  (estos SÍ son reusables, confirmado por sus imports: ninguno referencia el
  contexto huérfano ni los hooks muertos).
- `handleFinish()` (página) arma el payload de `POST /operations` asumiendo
  **una sola** `selectedReference` y **un solo** `sourceShipment` (`shipmentsArray`
  de longitud 0 o 1, línea 196-199, comentario "Single reference flow - simplified
  shipment structure").
- **Código huérfano adicional no detectado por el manifiesto anterior**: la
  carpeta `createOperation/components/reference-selection/` (`ReferenceCard.tsx`,
  `ShipmentCard.tsx`, `InvoiceCard.tsx`, `ItemCard.tsx`) y `createOperation/hooks/`
  (`useReferencesData.ts`, `usePreselectedReference.ts`, `useItemAvailability.ts`,
  `useExpandedState.ts`) — 818 líneas de una UI jerárquica de selección
  multi-referencia→embarque→factura→partida que existió antes (`StepReferencesSelection`,
  hoy eliminado según el comentario de `page.tsx:8`) y quedó huérfana: `grep` confirma
  que nada fuera de esa misma carpeta las importa. Son un patrón de UI reutilizable
  (cards jerárquicas con expand/collapse) aunque su fuente de datos (movimientos/
  facturas/partidas) no es DGO — habría que adaptarlas, no importarlas tal cual.
- El store (`stores/create-operation.store.ts`, 862 líneas) es **más rico de lo que
  el manifiesto anterior reportó**: además de los campos singulares usados por
  `page.tsx` (`selectedReference`, `sourceShipment`), ya tiene sin usar por la
  página actual:
  - `selectedReferences: SelectedReference[]` y `modalReferenceIds: string[]`
    (soporte multi-referencia parcialmente scaffolded, acciones `setSelectedReferences`,
    `openModalFromReferences`).
  - `groupPedimentosBy: 'ALL_INVOICES' | 'BY_INVOICE' | 'CUSTOM'` y
    `pedimentoGroups: Array<{ id, label, invoiceIds, formData }>` — un mecanismo YA
    MODELADO de agrupar facturas en N pedimentos dentro de una operación, con
    `formData` independiente por grupo (basicData/transportData/customsData).
  - `isCalculatingTaxes: boolean` y `CustomsData.taxCalculation` — flags/campo
    preparados para un cálculo de impuestos, pero **sin ningún fetch real que los
    dispare** (solo hay un setter `setIsCalculatingTaxes`, nada lo llama).
  - Comentarios del propio archivo lo confirman: "SPEC-003 FASE 3: Step-based
    wizard: 0) Selector, 1) Grouping, 2) Operation Config, 3-5) Per-pedimento" —
    es decir, el store fue diseñado para un wizard de selección+agrupación+N
    pedimentos que **nunca se terminó de cablear en `page.tsx`** (que quedó en una
    versión simplificada de 1 referencia). Esto es una base de partida mucho mejor
    de lo que el manifiesto anterior asumía (no hay que inventar el modelo de
    datos multi-pedimento desde cero: ya existe, solo no está conectado a la UI).

### B. TaxEngine (cálculo de impuestos) real
- Backend: `POST /operations/preview-taxes` (`operations.controller.ts:77-90` →
  `operations.service.ts:404`, `previewTaxes()`). **Ya acepta `referenceIds?: string[]`
  además de `referenceId`** (`preview-taxes.dto.ts`, comentario "para modo
  multi-referencia") y agrega/prorratea correctamente sobre varias referencias
  (`operations.service.ts:404-416`, hace `findMany` con `id: { in: referenceIds }`).
  **Limitación real:** filtra por **referencia** (y opcionalmente un `shipmentId`
  para prorrateo por peso), NO por `dgoId` — no puede aislar el cálculo a un DGO
  específico dentro de una referencia con varios DGOs.
- Frontend: el único consumidor es `components/operations/steps/Step3CustomsHeader.tsx`
  (fuera de alcance), con `fetch` directo (sin wrapper en
  `lib/api/modules/customs-operation.ts`). **No existe ninguna línea comentada
  "por reactivar"** — sería wiring nuevo desde cero en un componente in-scope.
- **El endpoint backend vive en el mismo módulo NestJS `src/operations` que ya usa
  el wizard in-scope para `POST /operations`, `GET /operations/prepare`, etc.**
  Esto importa para resolver la ambigüedad del paraguas ("todo lo llamado
  `operations`... módulos operations de las APIs = fuera de alcance"): en el
  backend solo existe **un** módulo `operations` (no hay un segundo módulo
  homónimo como sí ocurre en el front con `components/operations`), y el propio
  D1 original de este sub-plan ya dependía de él (`POST /operations`). Se
  interpreta (a validar con el humano, ver preguntas abiertas) que el límite de
  "fuera de alcance" aplica a las **rutas/componentes de front** llamados
  `operations` (Control Tower, `components/operations`, `customerPortal/operations`,
  `actions/operations`), no al módulo backend NestJS que el wizard in-scope ya
  consume — de otro modo el propio wizard in-scope quedaría sin backend.

### C. Módulo DGO (SP-05, back) — confirmado y ampliado
- `GET /dgo/reference/:referenceId` — lista DGOs de **una** referencia
  (`dgo.controller.ts:29-33` → `dgo.service.ts:63`, `listByReference`).
- `GET /dgo/:id` — **sí existe** un `findOne` por DGO individual
  (`dgo.controller.ts:53-56` → `dgo.service.ts:84-100`), con `invoices` (+ items),
  `globalExpenses`, `identifiers` incluidos. Esto es aprovechable: el front puede
  resolver N DGOs de referencias distintas haciendo N llamadas `GET /dgo/:id` en
  paralelo, **sin necesitar un endpoint nuevo de "listar por ids"** para el caso
  simple (aunque para UX de listado/paginación en Step 0 sí conviene un endpoint
  que liste referencias con sus DGOs, hoy inexistente).
- Modelo `Dgo` (prisma): `referenceId` (FK obligatoria, un DGO pertenece a **una**
  referencia), `dgoNumber`, `status`, campos de pedimento embebidos (`aduana`,
  `clavePedimento`, `patente`, `regimen`, `destino`) heredados de Reference si
  null, `invoices[]`, `globalExpenses[]`, `identifiers[]`. **No tiene relación
  hacia `Operation` ni `OperationPedimento`** (ver punto D).
- Ya existe una UI real y funcional que consume este módulo:
  `app/(customerPortal)/references/components/tabs/ReferenceDGOTab.tsx` (tab
  "DGO" del detalle de referencia, SP-05 front) — lista DGOs de una referencia en
  acordeón (número, status, # facturas, aduana/clave/patente/régimen), con panel
  de comparación y acción de firma. **Patrón de UI reutilizable** para construir
  las tarjetas de selección del Step 0 (aunque ahí es de solo-lectura/gestión, no
  de selección con checkbox).
- Confirmado (ya lo decía el manifiesto anterior): Mercancía/Gastos/Identificadores
  siguen filtrando por `referenceId`, no `dgoId`, en los endpoints que usa
  `stepOperationBasicData.tsx` (`GET /operations/prepare/:referenceId`).

### D. Relación DGO ↔ Operación/Pedimento a nivel de base de datos — **hallazgo nuevo, crítico, no cubierto por el manifiesto anterior**
- `model Operation`: `referenceId String @db.Uuid` es **obligatoria y singular**
  a nivel de esquema (no nullable, no es un arreglo). El DTO `CreateOperationDto`
  (`src/operations/dtos/create-operation.dto.ts`) sí acepta `referenceId?` **y**
  `referenceIds?: string[]` (ya soporta multi-referencia en el payload de
  entrada), pero el servicio (`operations.service.ts:882-990`, método `create()`)
  solo usa el **primero** (`primaryReferenceId = referenceIds?.[0] ?? referenceId`)
  para la relación real (`reference: { connect: { id: primaryReferenceId } }`); el
  resto de `referenceIds` se guarda **solo como metadata JSON**
  (`operationMetadata.additionalReferenceIds`), sin relación de BD ni efecto
  funcional real. Es decir: **hoy una Operación solo puede pertenecer de verdad a
  UNA referencia**, aunque el contrato HTTP aparente soportar varias.
- `model OperationPedimento` (tabla puente `operationId ↔ pedimentoId`, N:N) **ya
  existe y sí soporta N pedimentos por operación** a nivel de esquema — contradice
  la suposición del manifiesto anterior de que "multi-DGO → multi-pedimento"
  requeriría inventar esa relación desde cero. Lo que falta es **cablear el DGO**
  a esa relación: ni `Operation`, ni `OperationPedimento`, ni `Pedimento` tienen
  columna `dgoId` (`dgoId` solo existe hoy en `Invoice`, `GlobalExpense`,
  `GlobalIdentifier`, todas relaciones hacia `Dgo`, no al revés).
- **Conclusión de este punto**: construir "operación = N DGOs → N pedimentos" es
  viable sin rediseñar la BD desde cero (la tabla puente N:N ya está), pero si se
  quiere trazabilidad Pedimento↔DGO explícita (recomendable, para no depender de
  reconstruir la relación vía invoices) se necesita **una migración pequeña**
  (columna `dgoId` en `OperationPedimento` o en `Pedimento`) — trabajo de backend,
  no solo de frontend. Y si se quiere que una Operación agrupe DGOs de **más de
  una referencia**, además hay que decidir qué pasa con `Operation.referenceId`
  (hoy singular NOT NULL) — no es un ajuste mecánico, es cambio de esquema.

### E. Bloqueo de glosa (SP-04) — confirmado, no consumido hoy
- `GET /reference-documents/reference/:referenceId/can-start-operation` existe y
  llama a `assertInvoicesGlossedOk(referenceId)`
  (`reference-documents.controller.ts:77-86`,
  `reference-documents.service.ts:455`). Es a nivel **referencia**, no DGO.
- Confirmado por grep en `carmi-digital`: **ningún archivo** (ni el wizard
  in-scope, ni el huérfano) consume hoy este endpoint. Es wiring 100% nuevo si se
  quiere bloquear "iniciar operación con factura con error de glosa" desde el
  Step 0 del wizard.

### F. Payload actual de creación de operación (frontend)
- `lib/api/modules/customs-operation.ts`: `CreateOperationDto` (interfaz TS del
  front, no confundir con el DTO backend) **solo tiene `referenceId: string`**
  (singular, sin `referenceIds`) — el front-end nunca aprovechó el soporte
  multi-referencia que ya existe en el DTO backend. Si Step 0 selecciona DGOs de
  una sola referencia, el contrato actual alcanza tal cual (con ajuste menor: en
  vez de mandar 1 `shipmentId`, se construye `shipments[]` desde las facturas del
  DGO). Si se seleccionan DGOs de **más de una referencia**, hoy el payload
  frontend no tiene ni el campo `referenceIds` (aunque el backend sí lo acepta) —
  ajuste de tipos menor en el front, pero además choca con el punto D
  (`Operation.referenceId` singular a nivel BD).

## Fuera de alcance (sin cambios respecto al original)
- El wizard de 6 pasos `components/operations/**`.
- El vínculo Movimiento↔DGO como mecanismo de creación (dato de trazabilidad en
  el tab Movimientos, SP-07).

## Decisiones finales (entrevista + verificación en meets, 2026-07-12)

Todas las preguntas abiertas de la ronda de replanteo quedaron resueltas. Este
sub-plan ya está **listo para `/implementa`**.

1. **Frontera `operations`:** confirmado por el usuario — el `/operations` de
   **frontend** (Control Tower, `components/operations`, `customerPortal/operations`,
   `actions/operations`) sigue fuera de alcance. El módulo backend NestJS
   `src/operations` de `carmi-odin-api-v2` (el que respalda `/customs-operation`,
   in-scope) **sí se puede y se debe tocar** cuando el refactor lo necesite —
   incluye `preview-taxes` y cualquier endpoint nuevo que este sub-plan requiera.
2. **Multi-referencia confirmado por los meets** (no solo multi-DGO de una
   referencia): `Documento_Entendimiento_SIGAC3_Ejecutivo_v2` e
   `Inventario_Pantallas_v3` fijan explícitamente "una operación agrupa uno o
   más DGO/pedimentos, que pueden venir de varias referencias (consolidado)"
   — con casos reales de hasta 16-19 referencias consolidadas
   ([[2026-06-30 - Prueba operación real - glosa, pago y DODA]],
   [[2026-06-26 - Prueba operación real - flujo ejecutivo y consolidación de documentos]]).
   **Restricción a validar en Step 0:** régimen aduanero homogéneo entre los
   DGOs seleccionados ([[2026-07-08 - Aéreo SIGAC 3.0]]: "criterios para
   separar referencias: principalmente el régimen aduanero; también recepción
   parcial y volumen"). Esto implica: `Operation.referenceId` deja de alcanzar
   como única relación real — se necesita el cambio de esquema del punto 3.
3. **Migración de esquema requerida** (back, `carmi-odin-api-v2`): agregar
   `dgoId` a `OperationPedimento` (o a `Pedimento`) para trazabilidad explícita
   Pedimento↔DGO — **no** basta con derivarlo vía `Invoice.dgoId`. Además, dado
   que ahora una Operación agrupa DGOs de varias referencias, evaluar si
   `Operation.referenceId` debe volverse opcional/derivado (la relación real
   pasa a vivir en `OperationPedimento`→`Dgo`→`Reference`), documentando el
   cambio de esquema. Generar SIEMPRE vía `npx prisma migrate dev --name
   <kebab-case>` (regla global, nunca a mano).
4. **Prellenado con múltiples DGOs:** un formulario por pedimento/grupo — cada
   DGO seleccionado arma su propio grupo en `pedimentoGroups` con su propio
   `formData` prellenado desde ESE DGO (aduana/clave/patente/régimen propios).
   El usuario revisa/ajusta cada pedimento por separado dentro de la misma
   operación.
5. **Limpieza de código muerto: sí, en este sub-plan.** Al tocar exactamente
   esa zona del wizard, se elimina el código huérfano confirmado:
   `PedimentoWizardContext.tsx`, `StepInventorySelection.tsx`,
   `StepIncrementables.tsx`, `StepPartidasTable.tsx`, `StepPedimentoHeader.tsx`,
   la carpeta `reference-selection/` completa y sus hooks
   (`useReferencesData.ts`, `usePreselectedReference.ts`,
   `useItemAvailability.ts`, `useExpandedState.ts`).
6. **Bloqueo de edición del DGO tras generar pedimento:** una vez que un DGO
   está vinculado a un pedimento dentro de una operación, se congela — no se
   permite reabrir/editar ese DGO (correcciones posteriores se hacen a nivel
   pedimento/despacho, no reabriendo el DGO). Sin evidencia en meets para este
   punto; es una decisión de producto del usuario (2026-07-12), documentar como
   tal si más adelante se cuestiona.
7. **Sin folio adicional, con código legible:** no se necesita un folio
   separado que "viaje" del DGO al pedimento — la relación de BD (`dgoId`) es
   suficiente para trazabilidad. Sí se quiere un identificador **legible por el
   usuario** (no UUID) visible en las pantallas de despacho/pedimento. Antes de
   crear un campo nuevo, **verificar si `dgoNumber`** (ya existe en el modelo
   `Dgo` desde SP-05) cubre este uso — si su formato/generación no es apto para
   mostrarse como "código" de cara al usuario, documentarlo y proponer ajuste
   mínimo en vez de crear un campo paralelo redundante.

## Tareas

- [ ] **Migración de esquema (back, `carmi-odin-api-v2`):** agregar `dgoId` a
  `OperationPedimento` (FK hacia `Dgo`); evaluar y documentar el ajuste de
  `Operation.referenceId` para el caso multi-referencia (punto 2/3). Generar
  vía `npx prisma migrate dev --name <kebab-case>`.
- [ ] **Backend — endpoint(s) de selección multi-DGO/multi-referencia:** algo
  equivalente a "listar referencias con sus DGOs seleccionables" (hoy no
  existe; se resuelve hoy con N llamadas a `GET /dgo/:id` o `GET
  /dgo/reference/:referenceId`, pero para UX de Step 0 con posibles cientos de
  referencias conviene un endpoint dedicado, paginado, con filtro de régimen
  aduanero homogéneo).
- [ ] **Backend — validación de régimen aduanero homogéneo** entre los DGOs
  seleccionados antes de crear la operación (o al momento de seleccionar).
- [ ] **Backend — enforcement del bloqueo de edición del DGO** una vez
  vinculado a un pedimento (rechazar mutaciones sobre un DGO en ese estado).
- [ ] **Backend — ajustar `POST /operations`** para aceptar y persistir de
  verdad selección multi-referencia (no solo como metadata JSON): usar el
  `referenceIds[]` ya aceptado por el DTO y conectarlo realmente vía
  `OperationPedimento`→`Dgo`, en vez del `primaryReferenceId` actual.
- [ ] **Frontend — construir el Step 0 "Seleccionar DGO(s)"** desde cero
  (patrones de referencia: acordeón de `ReferenceDGOTab.tsx`, cards del
  `reference-selection/` ya eliminado — adaptar el patrón visual, no el
  código). Debe soportar seleccionar DGOs de **varias referencias**. Antes de
  avanzar: llamar `GET /reference-documents/reference/:referenceId/can-start-operation`
  por cada referencia involucrada y bloquear con mensaje claro si alguna falla
  (glosa con error).
- [ ] **Frontend — extender el store** (`selectedReferences` + nuevo campo de
  DGOs seleccionados, p.ej. `selectedDgos: SelectedDgo[]`), sin reemplazar el
  store existente.
- [ ] **Frontend — adaptar `pedimentoGroups`** para que 1 DGO seleccionado = 1
  grupo/pedimento, con `formData` prellenado desde CADA DGO (aduana/clave/
  patente/régimen propios, punto 4).
- [ ] **Frontend — cablear el cálculo de impuestos** desde un componente nuevo
  in-scope (dentro de `stepCustomsInfo.tsx` o hermano nuevo), llamando `POST
  /operations/preview-taxes` con wrapper nuevo en
  `lib/api/modules/customs-operation.ts`. Usar `isCalculatingTaxes` y
  `CustomsData.taxCalculation` ya modelados en el store.
- [ ] **Frontend — mostrar el identificador legible del DGO** (`dgoNumber` u
  ajuste si no alcanza, punto 7) en las pantallas de pedimento/despacho que
  correspondan.
- [ ] **Frontend — ajustar `handleFinish()`** en `page.tsx` para construir el
  payload multi-referencia/multi-DGO real (`referenceIds[]`, `shipments[]`/
  `invoices[]`/`items[]` derivados de las facturas de cada DGO seleccionado).
- [ ] **Limpieza de código muerto** (punto 5): eliminar
  `PedimentoWizardContext.tsx`, `StepInventorySelection.tsx`,
  `StepIncrementables.tsx`, `StepPartidasTable.tsx`, `StepPedimentoHeader.tsx`,
  `reference-selection/` y sus hooks.

## Riesgos y side effects
- Requiere cambios de esquema en `carmi-odin-api-v2` (migración `dgoId` +
  ajuste de `Operation.referenceId`) — deja de ser "solo frontend".
- El bloqueo de edición del DGO tras generar pedimento (decisión 6) no tiene
  respaldo en meets — si en despacho (SP-16) aparece un caso real de necesitar
  reabrir un DGO ya vinculado, revisar esta regla.
- Depende de SP-05 (DGO) — ya disponible en `refactor/customs-operation-sp05`.

## Criterios de verificación
- Gate estático verde. Playwright: crear una operación seleccionando DGOs de
  **más de una referencia** con régimen aduanero homogéneo → N pedimentos bajo
  la operación (uno por DGO, con su propio formulario prellenado), impuestos
  calculados, bloqueo si alguna referencia tiene facturas con glosa en error,
  bloqueo si se intenta editar un DGO ya vinculado a un pedimento; sin errores
  de consola.

## Estado
✍️ Redactado — listo para `/implementa`. D1 verificado directamente contra el
código fuente real (2026-07-11) y todas las decisiones de producto validadas
con el usuario + evidencia de meets (2026-07-12, ver "Decisiones finales").
Historial del intento previo conservado arriba (no se borró el rastro). El
manifiesto original con el diagnóstico que originó el replanteo sigue en
`Planes/.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo.md`.
