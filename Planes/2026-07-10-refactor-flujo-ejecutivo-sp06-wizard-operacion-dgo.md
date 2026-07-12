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

## Propuesta de tareas (ajustada a la realidad del código, pendiente de validar preguntas abiertas)

> Nota: estas tareas son un **borrador de diseño**, no una lista lista para
> `/implementa`. Varias dependen de las decisiones de la sección final.

1. **Construir el Step 0 "Seleccionar DGO(s)" desde cero** (no hay componente
   previo utilizable tal cual; sí hay dos patrones de referencia: el acordeón de
   `ReferenceDGOTab.tsx` y las cards jerárquicas huérfanas de
   `reference-selection/`). Alcance mínimo acordado en el paraguas: operar dentro
   de **una** referencia (consistente con el límite real de `Operation.referenceId`
   y de `GET /dgo/reference/:referenceId`) — selección multi-referencia queda
   pendiente de la Pregunta 2.
   - Llamar `GET /dgo/reference/:referenceId` para listar DGOs seleccionables
     (status, # facturas, clave/aduana/patente/régimen).
   - Antes de permitir avanzar: llamar
     `GET /reference-documents/reference/:referenceId/can-start-operation` y
     bloquear con mensaje claro si falla (glosa con error) — wiring nuevo (punto E).
   - Guardar la selección en el store: **extender** (no reemplazar) el store
     existente — usar/adaptar `selectedReferences` + un nuevo campo de DGOs
     seleccionados (p.ej. `selectedDgos: SelectedDgo[]`), en vez de inventar un
     store paralelo.
2. **Adaptar `pedimentoGroups` (ya existente en el store) para que 1 DGO
   seleccionado = 1 grupo/pedimento**, en vez de agrupar por factura manualmente.
   Esto reaprovecha el mecanismo `formData` por grupo que el store ya modela,
   evitando construir de cero el "multi-pedimento".
3. **Prellenar los pasos 1-3 (Datos Básicos/Transporte/Despacho) desde el/los
   DGO(s) seleccionados**: mapear `dgo.aduana/clavePedimento/patente/regimen` a
   los campos equivalentes de `CustomsData`/`OperationBasicData` (mismo mapeo que
   ya usa `stepOperationBasicData.tsx` con datos de referencia, extendido a DGO).
   Cuando haya N DGOs, decidir en la entrevista si se prellena por grupo/pedimento
   o se muestra un resumen agregado (Pregunta 4).
4. **Cablear el cálculo de impuestos desde un componente nuevo in-scope**
   (ej. dentro de `stepCustomsInfo.tsx` o un componente hermano nuevo en
   `createOperation/components/`), llamando `POST /operations/preview-taxes`
   directamente (es un endpoint del módulo backend `operations` que el wizard
   in-scope ya consume para otras cosas — no es tocar `components/operations`
   del front). Añadir el wrapper correspondiente a
   `lib/api/modules/customs-operation.ts` (hoy no existe). Usar `isCalculatingTaxes`
   y `CustomsData.taxCalculation`, ya modelados en el store, para el estado de
   carga y el resultado.
5. **Ajustar el payload de `POST /operations`** en `handleFinish()` de `page.tsx`
   para construir `shipments[]`/`invoices[]`/`items[]` a partir de las facturas de
   cada DGO seleccionado (hoy usa un único `sourceShipment`). Para el caso "N DGOs
   de la misma referencia → N pedimentos bajo 1 operación", evaluar si basta con
   1 sola llamada a `POST /operations` seguida de N operaciones sobre
   `OperationPedimento` (ya soporta N:N), o si se requiere la migración de
   `dgoId` en `OperationPedimento`/`Pedimento` (punto D) para trazabilidad — esto
   necesita una decisión explícita (Pregunta 3) antes de codificar, porque implica
   backend.
6. **Limpieza de código muerto relacionado** (fuera del criterio estricto de
   este sub-plan, pero se debe decidir si se hace aquí o en otro): evaluar borrar
   o dejar como referencia histórica `PedimentoWizardContext.tsx`,
   `StepInventorySelection.tsx`, `StepIncrementables.tsx`, `StepPartidasTable.tsx`,
   `StepPedimentoHeader.tsx`, y la carpeta `reference-selection/` + sus hooks, ya
   que ninguno se reutiliza tal cual (Pregunta 5).

## Riesgos y side effects
- **No tocar `components/operations`** (front, homónimo fuera de alcance) —
  aclarado: sí se puede consumir el endpoint backend `POST /operations/preview-taxes`
  del módulo NestJS `operations`, que es distinto del front `components/operations`
  (ver Pregunta 1 para confirmar esta lectura con el humano).
- Si la Pregunta 2 se resuelve como "sí, multi-referencia", este sub-plan pasa a
  requerir cambios de esquema en `carmi-odin-api-v2` (rama nueva en ese repo,
  hoy no creada) — cambia el alcance de "solo frontend" asumido originalmente.
- Depende de SP-05 (DGO) — ya disponible en `refactor/customs-operation-sp05`.

## Criterios de verificación (sin cambios de fondo, pendiente ajuste fino tras la entrevista)
- Gate estático verde. Playwright: crear una operación seleccionando 1+ DGOs de
  una referencia → 1+ pedimentos bajo la operación, config prellenada del DGO,
  impuestos calculados, bloqueo si la referencia tiene facturas con glosa en
  error; sin errores de consola.

## Preguntas abiertas para validar con el usuario antes de `/implementa`

Estas preguntas son de producto/negocio o de alcance técnico que este sub-agente
**no decidió por su cuenta** (política Plan-First / Human in the Loop):

1. **Frontera de "fuera de alcance = `operations`".** ¿Se confirma que el límite
   del paraguas aplica solo a rutas/componentes de **frontend** llamados
   `operations` (Control Tower, `components/operations`, `customerPortal/operations`,
   `actions/operations`), y que consumir el endpoint backend
   `POST /operations/preview-taxes` (mismo módulo NestJS que ya usa el wizard
   in-scope) desde un componente nuevo dentro de `createOperation/` **sí** está
   permitido? Si la respuesta es "no, ni el backend", entonces hace falta diseñar
   un TaxEngine paralelo o posponer el cálculo de impuestos a un sub-plan futuro.
2. **Alcance de selección multi-DGO.** ¿La operación agrupa DGOs de **una sola
   referencia** (más simple, consistente con `Operation.referenceId` singular en
   BD y con `GET /dgo/reference/:referenceId`), o debe poder agrupar DGOs de
   **varias referencias** (como sugiere el `referenceIds[]` ya aceptado —pero no
   realmente soportado a nivel de relación— por el DTO backend)? Esto determina
   si el sub-plan se queda en frontend o necesita una rama y cambios en
   `carmi-odin-api-v2`.
3. **Trazabilidad Pedimento↔DGO en BD.** Dado que `OperationPedimento` ya soporta
   N pedimentos por operación pero **no** tiene columna `dgoId`, ¿se requiere una
   migración pequeña para vincular explícitamente cada pedimento con su DGO de
   origen, o basta con derivarlo indirectamente vía las facturas (`Invoice.dgoId`)
   que ya trae cada pedimento?
4. **Prellenado con múltiples DGOs.** Si se seleccionan 2+ DGOs con datos de
   pedimento distintos (aduana/clave/régimen), ¿se prellena un formulario por
   pedimento/grupo (consistente con `pedimentoGroups.formData` ya modelado en el
   store), o se pide un dato único a nivel operación y se replica a todos los
   pedimentos?
5. **Limpieza de código muerto.** ¿Se borra en este mismo sub-plan el código
   huérfano identificado (`PedimentoWizardContext.tsx` y familia, más
   `reference-selection/` y sus hooks), o se deja intacto y se limpia en un
   sub-plan de "deuda técnica" aparte para no mezclar alcances?

## Estado
📋 Redactado — pendiente de validar preguntas abiertas con el usuario. No
implementar todavía. D1 verificado directamente contra el código fuente real
(no solo contra el manifiesto del intento anterior) el 2026-07-11. Historial del
intento previo conservado arriba (no se borró el rastro). El manifiesto original
con el diagnóstico que originó este replanteo sigue en
`Planes/.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo.md`.
