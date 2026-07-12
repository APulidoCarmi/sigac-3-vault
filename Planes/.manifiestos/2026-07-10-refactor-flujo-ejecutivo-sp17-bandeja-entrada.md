# Manifiesto SP-17: Bandeja de Entrada (Inbox)

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp17-bandeja-entrada]].

## Estado: 🚧 BLOQUEADO en fase de diseño (2026-07-12)

No se escribió código. Ramas creadas por convención (`refactor/customs-operation-sp17`,
encadenadas desde `refactor/customs-operation-sp16` en ambos repos) pero sin
diff propio — solo cargan el diff acumulado de Fase 1-3.

## Diagnóstico: D1 del sub-plan está desactualizado/inválido en 3 de sus 4 piezas

El D1 dice "reusa el tablero primario de SP-01" + "crea" tres secciones dando
a entender que son extensiones de datos ya existentes. La investigación en
código (ambos repos, graphify + lectura de fuente) muestra que **la mayoría
de las señales de negocio que el Inbox necesita mostrar no existen como datos
persistidos ni consultables hoy** — no es un gap de UI, es ausencia de modelo
de dominio. Detalle por pieza:

### 1. Tabs "Referencias en curso": automático / temporal / avanzado
**No existe ningún campo/enum en ningún repo** que represente esta
clasificación de 3 valores. Búsqueda exhaustiva en `prisma/schema.prisma`,
DTOs y servicios de `operations`/`references` (es/en): nada. Lo único
parecido es `CustomsRegime` (catálogo genérico `code/name/category/type`,
sin valores poblados que correspondan) y los campos legacy `regimen`/
`customsRegimeOld` (clave SAT de régimen, no un "tipo de despacho").

**Riesgo de falso amigo detectado:** `DispatchMonitor.tsx`
(`carmi-digital/app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx`)
dice *"Seguimiento en tiempo real del flujo de despacho con Temporal"* —
esto es **Temporal.io** (motor de workflows), sin relación con un "régimen
temporal" aduanero. Si el sub-plan asumió esa lectura, la premisa es errónea.

No hay decisión de qué significan estos 3 valores (¿tipos de tramitación
Anexo 22? ¿modo de captura? ¿algo del Documento de Entendimiento no
levantado en el inventario de código?) — requiere entrevista de producto,
no una implementación a ciegas.

### 2. "Cartas sin firmar N días del cliente"
No existe ningún tipo de documento "carta de instrucciones/autorización" con
firma del cliente. `Referencedocumenttype` (enum Prisma) tiene `CARTA_PORTE`
(bill of lading terrestre — documento distinto). El único rastro de "firma"
es un ejemplo de forma libre en `create-reference-document.dto.ts:127`
(`requirements: { requiresSignature: true, ... }`, JSON sin estructura,
no queryable). No hay campos `sentAt`/`signedAt` en ningún modelo.
Construir esto requiere: decidir el tipo de documento, agregar columnas
Prisma (migración CLI) y decidir la regla de "N días" (¿desde cuándo? ¿quién
la parametriza?) — decisión de producto, no de implementación.

### 3. "Guías llegadas sin identificar" (ligar o crear referencia)
**No existe absolutamente nada.** `Shipment.referenceId` es nullable pero
ningún flujo crea/lista shipments sin referencia intencionalmente. El único
punto de entrada real es `src/email-documents/**` →
`BolReconciliationService.reconcileBol`
(`src/email-documents/bol-reconciliation.service.ts:118`): cuando el BOL
extraído por Zeus no matchea ningún shipment existente, el código hace
`logger.warn(...)` **y descarta el dato** — no persiste nada, no hay tabla
ni endpoint para reconocerlo después. Construir la funcionalidad que pide el
sub-plan (mostrar la guía a todos, reconocerla, ligarla o crear referencia)
exige diseñar un modelo nuevo (`UnidentifiedWaybill` o similar) que capture
ese caso `no match` en vez de perderlo, más el flujo de vinculación/creación
— esto es una feature greenfield completa, no una vista sobre datos
existentes.

### 4. "Auditoría de CEUS" (folios faltantes)
"CEUS" no existe como módulo en ningún repo (solo aparece como sinónimo
textual de "Zeus" en comentarios de SP-04). Lo que sí existe y es
consultable: `GET /references/:id/documents/checklist` (SP-04, checklist de
documentos por referencia individual) y campos de folio dispersos por
modelo (`coveFolio`, `eDocumentFolio`, `manifestationFolio`,
`referenceFolio` — cada uno en su propio modelo, sin relación entre sí).
**No hay ningún endpoint agregado tipo dashboard** que junte "lo que falta"
de muchas referencias a la vez — habría que construir esa capa de
agregación desde cero cruzando checklist + `glosaStatus` + los folios
sueltos. Es factible sin decisión de producto adicional (a diferencia de
los 3 puntos previos), pero es trabajo de diseño técnico no trivial que no
estaba dimensionado en el D1 ("lo que falta: folios, etc." asume que el dato
agregado ya existe).

## Lo que SÍ es viable tal cual (piezas confirmadas)

- **Tablero primario de SP-01** (`ReferenceBoard.tsx`) es reusable/extensible
  como base — ya deja anotado en su propio código (comentario líneas 18-21)
  que "priorización real queda para el Inbox completo (SP-17)".
- **"Por identificar"** → `ReferenceStatus.DRAFT`, patrón ya usado en SP-01.
- **"Previo"** → modelo `Previo` (SP-09) totalmente consultable
  (`Previo.status IN (REQUESTED, IN_TABLE)` = esperando previo). Solo falta
  un endpoint agregado (listar referencias con Previo pendiente), trivial
  dado el modelo existente.
- **"Arribo"** → ya existe `GET /shipments/pre-arrival`
  (`src/shipments/controllers/shipment-pre-arrival.controller.ts`),
  directamente reusable para esta columna.
- **"Clasificación"**: existe el módulo `classification` (no explorado en
  profundidad porque el bloqueo ya está confirmado por los otros 3 puntos;
  no cambia la conclusión).

## Por qué se bloquea y no se improvisa

Instrucción explícita de mi tarea: "Si al leer el sub-plan y el código real
encuentras que su D1 está desactualizado o inválido, NO improvises: detente,
documenta el diagnóstico... y repórtalo como tal." 3 de las 4 piezas del
Inbox (tabs de régimen, cartas sin firmar, guías sin identificar) requieren
decisiones de producto no tomadas (qué significan, qué reglas de negocio
llevan, qué modelo de datos exacto) antes de poder escribir una migración
Prisma o un endpoint — construirlas a ciegas violaría también el estándar
del repo de "nada de atajos/parches" y probablemente generaría rework como
ya pasó con SP-16 (bloqueado por la misma razón: diseño insuficiente).

## Recomendación para desbloquear (para la próxima entrevista `/plan`)

Antes de reabrir este sub-plan, decidir con el usuario:
1. Qué son exactamente "automático/temporal/avanzado" (fuente: revisar
   Documento_Entendimiento / Inventario_Pantallas_v3 con más detalle, o
   preguntar directo — la reunión de origen M9 no lo especifica en el texto
   del sub-plan).
2. Si "cartas sin firmar" es una feature nueva completa (tipo de documento +
   flujo de firma) o si en realidad ya existe en SIGAC 2 con otro nombre que
   no se mapeó.
3. El alcance real de "guías sin identificar": ¿es solo BOL vía
   `email-documents`, o también AWB aéreo / otros? ¿Quién crea la referencia
   nueva desde ahí (mismo wizard de SP-02)?
4. Si la "Auditoría de CEUS" agregada es aceptable como capa nueva construida
   sobre checklist+folios existentes (viable sin decisión adicional), o si
   el usuario esperaba una integración Zeus/CEUS en vivo (no existe, ver
   manifiesto SP-04).

## Archivos tocados
Ninguno (código). Solo este manifiesto y las ramas creadas por convención
(sin commits, sin diff).

## Gates
No aplica — no se escribió código.

---

## Intento 2026-07-12 — implementación completa (piezas 2, 3, 4)

El D1 fue rediseñado por el usuario (ver `2026-07-10-refactor-flujo-ejecutivo-sp17-bandeja-entrada.md`
actual, fechado 2026-07-12) resolviendo las 4 preguntas abiertas de este manifiesto. Este
intento implementa el D1 rediseñado completo, sobre rama nueva
`refactor/customs-operation-sp17-v2` en ambos repos (encadenada desde
`refactor/customs-operation-sp16`, que ya traía el diff acumulado de todo el
paraguas hasta Fase 3). Pieza 1 (tabs automático/temporal/avanzado) sigue
**diferida, no implementada**, tal como pide el D1.

### `carmi-odin-api-v2`

**Migración Prisma** (vía CLI, `npx prisma migrate dev --name
add-inbox-signed-letters-and-unidentified-waybills`, una sola migración para
las 4 piezas de schema):
- `MompConfiguration`: 2 columnas nuevas, `requiresSignedLetters` (default
  `false`, comportamiento seguro por defecto) y `signedLettersDeadlineDays`.
- Modelo nuevo `ReferenceLetter` + enum `ReferenceLetterStatus`
  (PENDING/SENT/SIGNED), con relación a `Reference` (cascade) y FK opcional
  única a `ReferenceDocument` (el PDF digitalizado una vez firmada).
- `Referencedocumenttype` (schema) y su espejo TS `ReferenceDocumentType`
  (`src/reference-documents/dtos/create-reference-document.dto.ts`): valor
  nuevo `CARTA_FIRMA`.
- Modelo nuevo `UnidentifiedWaybill` + enum `UnidentifiedWaybillStatus`
  (PENDING/LINKED_EXISTING/REFERENCE_CREATED/DISMISSED), con FKs opcionales a
  `Shipment` y `Reference` (caminos "ligar existente"/"crear referencia").
- `prisma migrate status` confirmó estado limpio antes de migrar (última
  migración previa: `add_shipper_step_name` de SP-16).

**Pieza 2 — cartas sin firmar** (`src/reference-documents/services/reference-documents.service.ts`,
`.../controllers/reference-documents.controller.ts`, DTO nuevo
`dtos/reference-letter.dto.ts`):
- `createLetter`, `markLetterSent`, `markLetterSigned` (linkea opcionalmente
  el `ReferenceDocument` digitalizado), `getPendingLetters(daysThreshold?)`
  (cruza `MompConfiguration.signedLettersDeadlineDays` por cliente vs. el
  query param explícito; sin plazo configurado en ningún lado, la carta se
  lista igual como informativa sin marcarse vencida).
- `assertSignedLettersOk(referenceId)`: no bloquea si el MOMP del cliente no
  requiere cartas (`requiresSignedLetters=false`, default); si las requiere,
  exige que TODAS las `ReferenceLetter` no borradas de la referencia estén
  `SIGNED`.
- Wiring en los 3 puntos exactos que pedía el D1: `reglosaExpediente` (al
  inicio, antes del motor de glosa), `ReferenceDocumentsController.canStartOperation`
  (línea ~83, junto a la llamada ya existente a `assertInvoicesGlossedOk`), y
  `OperationsService.create` (`src/operations/services/operations.service.ts:917-922`,
  junto a la llamada existente a `assertInvoicesGlossedOk`, dentro del mismo
  bucle sobre `involvedReferenceIds`). Confirmado con `git diff`/lectura de
  código que no pisa nada de `startDispatchWorkflow` (SP-16, método distinto,
  línea 3578+).
- Endpoints nuevos: `POST /reference-documents/reference/:referenceId/letters`,
  `GET /reference-documents/letters/pending`, `PATCH
  /reference-documents/letters/:id/mark-sent`, `PATCH
  /reference-documents/letters/:id/mark-signed`.

**Pieza 3 — guías sin identificar** (`src/email-documents/**`):
- `DocumentProcessorService.processDocument`: case nuevo para
  `airwaybill`/`air_waybill`/`awb` → `action: 'match_shipment'` +
  `extractAwbFields` (nuevo, análogo a `extractBolFields`).
- `BolReconciliationService` generalizado por `trackingType: 'BOL' | 'AWB'`
  (`WaybillReconciliationInput.trackingType`, default `'BOL'` por
  compatibilidad): el match contra `ShipmentTrackingNumber` usa el
  `trackingType` recibido; el lado `AppointmentBol`/`Appointment` (terrestre)
  se gatea explícitamente a `trackingType === 'BOL'` en los 4 puntos donde
  aplicaba (query inicial, replicación BOL→cita, adjuntar documento a cita,
  timeline de cita) — para AWB nunca se toca el lado de citas, tal como pedía
  el D1.
- Persistencia de `UnidentifiedWaybill` (antes solo `logger.warn`) en 4 ramas:
  BOL/AWB vacío, `no_match` (sin cita ni shipment), cita con BOL pero sin
  shipment linkeado, y el `catch` general de la transacción — vía método
  privado `persistUnidentifiedWaybill` (acepta `tx` opcional para participar
  de la transacción cuando aplica, o `this.prisma` cuando la tx ya falló).
- Nuevo método público `ensureTrackingNumberForShipment` en
  `BolReconciliationService` (envuelve el `ensureShipmentTrackingNumber`
  privado ya existente) para que el flujo "ligar a existente" de
  `UnidentifiedWaybill` lo reuse sin duplicar lógica.
- Módulo nuevo `unidentified-waybills` dentro de `email-documents`
  (`services/unidentified-waybills.service.ts`,
  `controllers/unidentified-waybills.controller.ts`,
  `dto/unidentified-waybill.dto.ts`), wireado en `email-documents.module.ts`
  (que ahora también exporta `BolReconciliationService`). Endpoints: `GET
  /email-documents/unidentified-waybills?status=PENDING`, `POST
  .../:id/link-existing` (resuelve el shipment INBOUND de la referencia
  destino, o el primero disponible; 400 si no hay ninguno), `POST
  .../:id/create-reference` (marca `REFERENCE_CREATED`).
- `EmailDocumentsService.processIncoming`: nuevo helper privado
  `resolveTrackingType(documentType)` que decide `'AWB'` vs `'BOL'` según el
  `documentType` que decidió el processor, pasado a `reconcileBol`.

**Pieza 4 — panel de pendientes** (`ReferenceDocumentsService.getPendingPanel`
+ `GET /reference-documents/reference/:referenceId/pending-panel`): agrega,
sin tablas nuevas, `getChecklist` (documentos base), `ReferenceDocument` con
`glosaStatus != GLOSSED_OK` (categoría "glosa"), `ReferenceLetter` no
`SIGNED` (categoría "letters", con `daysSinceSent`), `Previo` en
`REQUESTED`/`IN_TABLE`, y folios `Cove`/`EDocument`/`ValueManifestation` sin
folio o sin `status=COMPLETED` (vía `pediment: { referenceId }` para
`ValueManifestation`, confirmando la relación `ValueManifestation.pedimentId
→ Pedimento.referenceId`). Diseñado para cargarse bajo demanda por
referencia (sin precálculo masivo), tal como pedía el D1.

**Tests nuevos**: `reference-documents.service.spec.ts` (assertSignedLettersOk
— incluye la "prueba de bloqueo" de los Criterios de verificación del D1,
letters CRUD, getPendingPanel), `document-processor.service.spec.ts` (AWB
recognition), `unidentified-waybills.service.spec.ts` (nuevo, linkExisting/
markReferenceCreated), `bol-reconciliation.service.spec.ts` (sin cambios de
código, verificado que sigue verde con el nuevo flujo de persistencia
degradando con gracia si el mock no cubre `unidentifiedWaybill.create`).
**Desviación documentada**: no se agregó un test end-to-end de
`OperationsService.create()` ejerciendo la rama DGO con `assertSignedLettersOk`
real — ese método no tenía NINGÚN test de `create()` antes de este sub-plan
(no existe `describe('create'...)` en `operations.service.spec.ts`), habría
requerido construir desde cero un mock scaffolding grande fuera del alcance
declarado de este sub-plan; sí se corrigió el mock de
`ReferenceDocumentsService` en ese spec para incluir `assertSignedLettersOk`
(evita que cualquier test futuro que ejercite la rama DGO rompa por función
`undefined`), y la lógica de `assertSignedLettersOk` en sí está cubierta
exhaustivamente en `reference-documents.service.spec.ts`.

**Gate**: `npx tsc --noEmit` sin errores nuevos (errores preexistentes solo en
specs no tocados: `twilio.service.spec.ts`, `seal-resolver.service.spec.ts`,
`company-patent-signature-config.service.spec.ts`, `audited.spec.ts`, 3
e2e-spec — todos con problemas de tipado de Jest ajenos a este sub-plan).
`npx eslint` sin errores nuevos (solo warnings/errores preexistentes en
archivos con import/order y una aserción innecesaria, confirmados
comparando contra el estado pre-cambio con `git stash`). `npx jest` completo:
**528/528 test suites, 3488/3494 tests (6 todo preexistentes)**.

### `carmi-digital`

**Pantalla nueva** `app/(customerPortal)/inbox/page.tsx` (ruta `/inbox`), con
3 componentes en `app/(customerPortal)/inbox/components/`:
- `WaitingOnOthersSection.tsx` — 3 columnas: "Por identificar" (`DRAFT`,
  reusa el mismo fetch pattern que `ReferenceBoard.tsx`), "Cartas sin firmar"
  (`GET /reference-documents/letters/pending`, nuevo), "Arribo" (reusa
  `preArrivalService.getAll()` de `lib/api/modules/shipment-pre-arrival.ts`
  tal cual, sin cambios). **Desviación documentada**: "Previo" y
  "clasificación" (mencionados en el D1 como parte de esta sección) NO se
  implementaron como columnas de datos reales — `PrevioController` solo
  expone `GET /previo/reference/:referenceId` (1 por referencia, no hay
  endpoint de listado cross-referencia por status), y construir ese agregado
  nuevo no está en la sección "## Pasos" del D1 (que sólo lista los 3 pasos
  de backend de piezas 2/3/4, ninguno de agregación de Previo/clasificación).
  Se documenta la nota de alcance visible en la propia UI (pie de sección) en
  vez de improvisar un endpoint nuevo fuera de lo declarado — a diseñar en un
  sub-plan de seguimiento si el usuario lo prioriza.
- `PendingPanelSection.tsx` — búsqueda por Reference ID +
  `GET /reference-documents/reference/:id/pending-panel` (carga bajo demanda,
  sin precálculo para todo el tablero, tal como pedía el D1).
- `UnidentifiedWaybillsSection.tsx` + `RecognizeWaybillModal.tsx` — lista de
  `GET /email-documents/unidentified-waybills?status=PENDING`, botón
  "Reconocer" abre modal con 2 tabs: "Ligar a existente" (`SearchableSelect`
  buscando por `GET /references?search=...`, reusa el componente ya existente
  en `@/components/ui/searchable-select`) y "Crear referencia nueva" (navega
  al wizard SP-02 con `clientId` prellenado si `extractedJson.clientId`
  existe, más un `waybillId` nuevo en el querystring).
- **Wizard SP-02** (`app/(customerPortal)/references/createReference/page.tsx`):
  se agregó lectura de `waybillId` desde `useSearchParams()` y, al crear la
  referencia exitosamente, una llamada no bloqueante a `POST
  /email-documents/unidentified-waybills/:id/create-reference` con el
  `referenceId` resultante — cierra el ciclo "crear referencia nueva" del D1
  sin que el ejecutivo tenga que volver manualmente al Inbox a confirmar.
- **Sidebar** (`components/ui/sidebar/Sidebar.tsx`): entrada nueva "Bandeja de
  Entrada" (`/inbox`, ícono `Inbox` de lucide-react) como primer ítem del
  menú, antes de "Dashboard" — es el punto de arranque del día
  (Inventario_Pantallas_v3 #1).
- No se tocó `components/operations`, `customerPortal/operations` ni
  `actions/operations` (fuera de alcance del paraguas, confirmado).

**Gate**: `npx tsc --noEmit -p tsconfig.json` — 0 errores (repo completo).
`npx eslint` sobre los archivos tocados — 0 errores, solo warnings
preexistentes (unused vars/`any` ya presentes antes de este sub-plan en
`createReference/page.tsx` y `Sidebar.tsx`, confirmados por patrón repetido).
`npx next build` — **build exitoso, sin errores**, ruta `/inbox` registrada
(el bloqueo preexistente de Turbopack/`three`/`mammoth` que documentaba el
manifiesto de SP-06 ya no reprodujo en este intento — posible resolución
externa por `pnpm install` de un sub-plan anterior). `npx jest` — 88/88 tests
propios pasan (2 suites ajenas fallan por dependencia faltante
`react-i18next`, preexistente, no relacionado).

### Bloqueos / pendientes
- Playwright end-to-end sigue pendiente de sesión humana (transversal a todo
  el paraguas, sin `dev_url` configurado para este cliente).
- Agregación cross-referencia de Previo/clasificación para el Inbox: gap
  documentado arriba, requiere decisión de si vale la pena un endpoint nuevo
  (fuera de alcance de este sub-plan).

### Archivos tocados (código)
**`carmi-odin-api-v2`**:
- `prisma/schema.prisma`, `prisma/migrations/20260712095908_add_inbox_signed_letters_and_unidentified_waybills/`
- `src/reference-documents/services/reference-documents.service.ts` (+ `.spec.ts`)
- `src/reference-documents/controllers/reference-documents.controller.ts`
- `src/reference-documents/dtos/reference-letter.dto.ts` (nuevo)
- `src/reference-documents/dtos/create-reference-document.dto.ts`
- `src/operations/services/operations.service.ts` (+ `.spec.ts`)
- `src/email-documents/document-processor.service.ts` (+ `.spec.ts`)
- `src/email-documents/bol-reconciliation.service.ts`
- `src/email-documents/email-documents.service.ts`
- `src/email-documents/email-documents.module.ts`
- `src/email-documents/dto/unidentified-waybill.dto.ts` (nuevo)
- `src/email-documents/services/unidentified-waybills.service.ts` (nuevo, + `.spec.ts`)
- `src/email-documents/controllers/unidentified-waybills.controller.ts` (nuevo)

**`carmi-digital`**:
- `app/(customerPortal)/inbox/page.tsx` (nuevo)
- `app/(customerPortal)/inbox/components/WaitingOnOthersSection.tsx` (nuevo)
- `app/(customerPortal)/inbox/components/PendingPanelSection.tsx` (nuevo)
- `app/(customerPortal)/inbox/components/UnidentifiedWaybillsSection.tsx` (nuevo)
- `app/(customerPortal)/inbox/components/RecognizeWaybillModal.tsx` (nuevo)
- `app/(customerPortal)/references/createReference/page.tsx`
- `components/ui/sidebar/Sidebar.tsx`
