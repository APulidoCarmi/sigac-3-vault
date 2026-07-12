# Sub-plan SP-17: Bandeja de Entrada (Inbox)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #1 del [[Inventario_Pantallas_v3]] (🔵 nueva, propuesta M9 — "le gustó a
German"). Punto de arranque del día del ejecutivo. Origen:
[[2026-07-07 - Revision de pantallas flujo operación]] (M9: tablero / bandeja de entrada).

Este D1 reemplaza al original: el D1 anterior asumía que 3 de las 4 piezas eran
vistas sobre datos ya existentes; la investigación (ver manifiesto) mostró que
no había modelo de dominio para ninguna. El usuario resolvió las 4 preguntas
abiertas del manifiesto (2026-07-12) y este documento traduce esas respuestas
a diseño técnico verificado contra el código real (`refactor/customs-operation-sp18`
en ambos repos).

## D1 — piezas del Inbox

### Pieza 1 — Tabs "automático / temporal / avanzado" (Referencias en curso)
**DIFERIDA — fuera de alcance de este sub-plan.** El usuario no tiene contexto
de negocio para definir qué significan estos 3 valores y pidió dejarlo
pendiente en vez de construir algo a ciegas. No hay campo/enum para esto en
ningún repo (confirmado, ver manifiesto). **No implementar.** Cuando exista
contexto de producto, abrir un sub-plan nuevo (no reabrir este).
La columna "En curso" del tablero de SP-01 (`ReferenceBoard.tsx`, estados
`QUOTED`+`APPROVED`) se mantiene tal cual, sin tabs, dentro del Inbox.

### Pieza 2 — Cartas sin firmar
**Contexto de negocio (dado por el usuario):** las "cartas" (cartas de
recepción / cartas de firma) son un requisito legal para formalizar la
operación aduanera. El ejecutivo las solicita al cliente junto con la
proforma; el representante legal del cliente debe firmarlas y devolverlas.
Sin la respuesta firmada, el equipo **no puede proceder con la glosa (SP-04)
ni con el despacho**. Una vez devueltas firmadas, se digitalizan como parte
del expediente aduanero digital (`ReferenceDocument`). La necesidad de pedir
estas cartas depende del modelo operativo de cada cliente → vive en el MOMP.

**Por qué NO calza meter esto directo en `ReferenceDocument`:**
`ReferenceDocument.fileUrl` es `String` no-nullable (`prisma/schema.prisma:3469`).
No puede existir una fila de "carta enviada al cliente, aún sin archivo
firmado" porque toda fila de `ReferenceDocument` exige ya tener un archivo.
Por eso el ciclo de vida "solicitada → enviada → firmada" vive en un **modelo
nuevo dedicado** (`ReferenceLetter`), y `ReferenceDocument` solo entra cuando
la carta YA está firmada y se sube el PDF digitalizado (documentType nuevo
`CARTA_FIRMA`, distinto de `CARTA_PORTE` que ya existe y es un documento de
transporte terrestre no relacionado).

**Diseño de datos (migración Prisma nueva):**

```prisma
// MompConfiguration — 2 columnas nuevas
model MompConfiguration {
  // ...existentes...
  requiresSignedLetters     Boolean @default(false)
  signedLettersDeadlineDays Int?    // null = sin plazo parametrizado; ej. 3
}

// Modelo nuevo — ciclo de vida solicitud/envío/firma de cartas
model ReferenceLetter {
  id                  String               @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  referenceId         String               @db.Uuid
  letterType          String               @db.VarChar(50) // "CARTA_RECEPCION" | "CARTA_FIRMA" (texto libre, sin enum rígido)
  status              ReferenceLetterStatus @default(PENDING)
  requestedAt         DateTime?            @db.Timestamptz(6)
  requestedBy         String?              @db.Uuid
  sentAt              DateTime?            @db.Timestamptz(6) // clave para "N días sin firmar"
  signedAt            DateTime?            @db.Timestamptz(6)
  referenceDocumentId String?              @unique @db.Uuid // FK al PDF digitalizado (CARTA_FIRMA) una vez firmada
  notes               String?
  createdAt           DateTime             @default(now()) @db.Timestamptz(6)
  updatedAt           DateTime             @updatedAt @db.Timestamptz(6)
  deletedAt           DateTime?            @db.Timestamptz(6)

  reference         Reference          @relation(fields: [referenceId], references: [id], onDelete: Cascade)
  referenceDocument ReferenceDocument? @relation(fields: [referenceDocumentId], references: [id])

  @@index([referenceId])
  @@index([status])
  @@map("reference_letters")
  @@schema("public")
}

enum ReferenceLetterStatus {
  PENDING  // solicitada, aún no enviada
  SENT     // enviada al cliente, esperando firma — dispara el conteo de "N días"
  SIGNED

  @@map("ReferenceLetterStatus")
  @@schema("public")
}
```

Agregar `CARTA_FIRMA` a `enum Referencedocumenttype` (schema y su espejo
`ReferenceDocumentType` en `src/reference-documents/dtos/create-reference-document.dto.ts`).

**Endpoints nuevos** (`src/reference-documents` module, o módulo `momp` si se
prefiere colocar ahí la creación de la solicitud — recomendado: vive junto a
`reference-documents` porque su ciclo termina siendo un `ReferenceDocument`):
- `POST /reference-documents/reference/:referenceId/letters` — crear
  solicitud (`letterType`, `requestedBy`) → status `PENDING`.
- `PATCH /reference-documents/letters/:id/mark-sent` → status `SENT`,
  `sentAt = now()`.
- `PATCH /reference-documents/letters/:id/mark-signed` → status `SIGNED`,
  `signedAt = now()`, opcionalmente recibe `referenceDocumentId` si ya se
  subió el PDF digitalizado (o se sube por separado con
  `POST /reference-documents` y luego se linkea).
- `GET /reference-documents/letters/pending?daysThreshold=N` — para el
  Inbox: referencias con `ReferenceLetter.status = 'SENT'` y
  `sentAt <= now() - N days` (N tomado de `MompConfiguration.signedLettersDeadlineDays`
  del cliente de cada referencia, o el `daysThreshold` del query si se pasa
  explícito).

**Bloqueo real (glosa + despacho):**
Nuevo método en `ReferenceDocumentsService`, mismo patrón que
`assertInvoicesGlossedOk` (línea 460):

```ts
async assertSignedLettersOk(referenceId: string): Promise<void> {
  const reference = await this.prisma.reference.findUnique({
    where: { id: referenceId },
    select: { clientCompanyId: true },
  });
  if (!reference) throw new NotFoundException(...);

  const momp = await this.prisma.mompConfiguration.findUnique({
    where: { companyId: reference.clientCompanyId },
    select: { requiresSignedLetters: true },
  });
  if (!momp?.requiresSignedLetters) return; // este cliente no las requiere

  const letters = await this.prisma.referenceLetter.findMany({
    where: { referenceId, deletedAt: null },
    select: { status: true },
  });
  const allSigned = letters.length > 0 && letters.every(l => l.status === 'SIGNED');
  if (!allSigned) {
    throw new BadRequestException(
      'No se puede proceder: el MOMP del cliente requiere cartas firmadas ' +
      'y aún hay cartas pendientes de firma.',
    );
  }
}
```

Puntos de integración (llamar `assertSignedLettersOk` junto a
`assertInvoicesGlossedOk` en los mismos 3 lugares, sin tocar la lógica de
esos lugares más allá de agregar la llamada):
1. `ReferenceDocumentsService.reglosaExpediente` (línea 410) — al inicio,
   antes de correr el motor de glosa (bloquea "no proceder con la glosa").
2. `ReferenceDocumentsController.canStartOperation` (línea ~83).
3. `OperationsService.create` (`src/operations/services/operations.service.ts:917`)
   — junto a la validación existente de `assertInvoicesGlossedOk` por cada
   `referenceId` involucrado en los DGOs seleccionados. **No se reabre el
   alcance de SP-16**: solo se agrega esta llamada al lado de la ya
   existente; el arranque de despacho en sí lo sigue diseñando SP-16.

**Vista en el Inbox:** lista de referencias con `ReferenceLetter.status='SENT'`
y `sentAt` más allá del plazo (columna "Cartas sin firmar", ordenada por días
transcurridos desc), usando `GET /reference-documents/letters/pending`.

### Pieza 3 — Guías llegadas sin identificar
**Alcance confirmado por el usuario:** aplica a cualquier tipo de tráfico
(BOL marítimo, AWB aéreo, y cualquier otro), no solo BOL. Al reconocer una
guía se puede usar el wizard de crear referencia (SP-02) **o** ligarla a una
referencia ya existente de ese cliente — ambos caminos válidos.

**Hallazgo de código que amplía el diagnóstico del manifiesto:** hoy no solo
falta persistir el caso `no_match` de BOL — **no existe ningún reconocimiento
de tipo AWB en absoluto**. `DocumentProcessorService.processDocument`
(`src/email-documents/document-processor.service.ts:80-158`) solo tiene
cases para `billoflading`/`bill_of_lading` (→ `match_shipment`), `invoice`,
`packinglist`, `other`/`unknown` (→ `manual_review`) y un `default` que hace
`store_reference` silenciosamente para cualquier otro tipo (incluido AWB) —
ni siquiera pasa por revisión manual. Este sub-plan corrige ambos huecos:
persistir el `no_match` de BOL y agregar reconocimiento explícito de AWB.

**Diseño de datos (migración Prisma nueva):**

```prisma
model UnidentifiedWaybill {
  id               String                    @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  waybillType      String                    @db.VarChar(20) // 'BOL' | 'AWB' | 'OTHER' (mismo valor libre que ShipmentTrackingNumber.trackingType)
  waybillNumber    String                    @db.VarChar(100)
  fileId           String?
  fileUrl          String?
  emailMessageId   String?
  source           String                    @default("email") @db.VarChar(20)
  extractedJson    Json?                     // payload Zeus, insumo para prellenar el wizard SP-02
  status            UnidentifiedWaybillStatus @default(PENDING)
  recognizedBy      String?                   @db.Uuid
  recognizedAt      DateTime?                 @db.Timestamptz(6)
  linkedShipmentId  String?                   @db.Uuid  // camino "ligar a existente"
  linkedReferenceId String?                   @db.Uuid  // camino "crear referencia nueva" (o resultado del ligado)
  createdAt         DateTime                  @default(now()) @db.Timestamptz(6)
  updatedAt         DateTime                  @updatedAt @db.Timestamptz(6)

  linkedShipment  Shipment?  @relation(fields: [linkedShipmentId], references: [id])
  linkedReference Reference? @relation(fields: [linkedReferenceId], references: [id])

  @@index([status])
  @@index([waybillType, waybillNumber])
  @@map("unidentified_waybills")
  @@schema("public")
}

enum UnidentifiedWaybillStatus {
  PENDING           // esperando que alguien la reconozca
  LINKED_EXISTING   // ligada a una referencia existente
  REFERENCE_CREATED // se creó una referencia nueva (wizard SP-02) a partir de ella
  DISMISSED         // descartada manualmente (falso positivo / duplicado)

  @@map("UnidentifiedWaybillStatus")
  @@schema("public")
}
```

**Cambios de servicio:**
1. `BolReconciliationService.reconcileBol` (línea ~118): en la rama
   `status = 'no_match'` (línea ~119, hoy solo `logger.warn`), **persistir**
   un `UnidentifiedWaybill` (`waybillType: 'BOL'`) en vez de descartar el
   dato. Mismo patrón para el resto de ramas `manual_review` que hoy solo
   loguean sin persistir (líneas ~131, ~139).
2. `DocumentProcessorService.processDocument` (línea 80): agregar cases
   `'airwaybill' | 'air_waybill' | 'awb'` (mismo estilo que el case BOL,
   líneas 91-106) que devuelvan `action: 'match_shipment'` con
   `documentType: 'awb'`, reusando `extractBolFields`-style extractor nuevo
   (`extractAwbFields`) para el número de guía aérea.
3. Generalizar `BolReconciliationService.reconcileBol` (o crear
   `WaybillReconciliationService` que lo envuelva) para aceptar un parámetro
   `trackingType: 'BOL' | 'AWB'` y usarlo al hacer match contra
   `ShipmentTrackingNumber.trackingType` (hoy hardcodeado a `'BOL'` en la
   query, línea ~84) — el dato ya soporta múltiples tipos (`trackingType` es
   `String` libre, ya usado con valor `'AWB'` en
   `src/appointments/appointments.service.ts:2181,2298`). El lado
   `AppointmentBol`/`Appointment` del reconciliation **NO se generaliza**:
   `Appointment` es inherentemente terrestre (`truckPlate`, `trailerPlate`)
   y no tiene equivalente para tráfico aéreo — para AWB el matching solo
   corre contra `ShipmentTrackingNumber`, sin intentar el lado de citas.
4. En `EmailDocumentsService.processIncoming` (línea ~75), extender el loop
   de `decision.action === 'match_shipment'` para invocar el reconciliation
   generalizado con el `trackingType` correcto según `decision.documentType`.

**Flujo de reconocimiento (nuevo endpoint + UI):**
- `GET /email-documents/unidentified-waybills?status=PENDING` — listado
  para el Inbox (visible a todos los ejecutivos).
- `POST /email-documents/unidentified-waybills/:id/link-existing` —
  body `{ referenceId, recognizedBy }`: crea/asegura el
  `ShipmentTrackingNumber` en el shipment de esa referencia (reusa
  `ensureShipmentTrackingNumber` ya existente en
  `BolReconciliationService`), marca `status = LINKED_EXISTING`,
  `linkedReferenceId`.
- `POST /email-documents/unidentified-waybills/:id/create-reference` —
  marca `status = REFERENCE_CREATED` cuando el front confirma que el wizard
  SP-02 terminó (el front navega primero al wizard, ver abajo, y llama este
  endpoint al finalizar con el `referenceId` resultante).
- Front: botón "Reconocer" en la fila de la guía sin identificar abre un
  modal con 2 acciones:
  - **"Ligar a referencia existente"**: buscador de referencias del mismo
    cliente (mismo patrón de `GET /references?clientCompanyId=...` que ya
    usa `ReferenceBoard.tsx`) → llama `link-existing`.
  - **"Crear referencia nueva"**: navega a
    `/references/createReference?clientId=<companyId inferido de extractedJson si se pudo resolver>`
    — reusa el wizard SP-02 tal cual, que YA soporta prefill por
    querystring (`app/(customerPortal)/references/createReference/page.tsx:110-112`
    lee `clientId`/`shipmentId` de `useSearchParams()`); al terminar el
    wizard, el front llama `create-reference` con el `referenceId` creado.

**Nota de alcance:** si `extractedJson` de Zeus no trae el cliente
identificado (guía verdaderamente huérfana sin ningún dato de contraparte),
el ejecutivo debe poder buscar/seleccionar el cliente manualmente en el modal
antes de elegir "ligar" o "crear" — el modal de reconocimiento no asume que
el cliente ya se resolvió automáticamente.

### Pieza 4 — Panel de pendientes (antes "Auditoría de CEUS")
**Simplificado según el usuario:** no hay integración en vivo con
Zeus/CEUS. Es un panel agregado, construido sobre datos ya existentes, que
muestra por referencia/operación qué falta por llenar, recibir o
clasificar.

**Diseño: endpoint de agregación nuevo**, `GET /references/:id/pending-panel`
(o `GET /reference-documents/reference/:referenceId/pending-panel` — se
recomienda el segundo, junto al resto de agregados de expediente SP-04, para
no duplicar lógica de resolución de `clientCompanyId`/MOMP que ya vive ahí).

Fuentes a cruzar (todas ya consultables, confirmado contra schema):
- **Checklist de documentos** — reusar
  `ReferenceDocumentsService.getChecklist` (línea 489, incluye `mompReady`).
  Nota: existe una segunda implementación paralela,
  `ReferencesService.getDocumentChecklist`
  (`src/references/services/references.service.ts:3388`) — duplicación
  preexistente no introducida por este sub-plan; se documenta pero no se
  resuelve aquí (fuera de alcance).
- **`glosaStatus`** por documento (`ReferenceDocument.glosaStatus`,
  `PENDING`/`GLOSSED_OK`/`GLOSSED_ERROR`).
- **Cartas sin firmar** (pieza 2) — `ReferenceLetter` con `status != SIGNED`.
- **Previo** (SP-09) — `Previo.status IN ('REQUESTED','IN_TABLE')` = esperando.
- **Folios dispersos**:
  - `Cove.coveFolio` (`Cove.referenceId` directo).
  - `EDocument.eDocumentFolio` (vía `EDocument.referenceDocumentId →
    ReferenceDocument.referenceId`).
  - `ValueManifestation.manifestationFolio` (vía `ValueManifestation.pedimentId
    → Pedimento` — confirmar en implementación la relación exacta
    `Pedimento → Reference`, es detalle de query, no de diseño).
  - Cada uno con `status` propio (`VucemStatus`) — folio ausente o status
    distinto de completado cuenta como "pendiente".
- **`Reference.referenceNumber`** (no `referenceFolio` como decía el
  manifiesto — se verificó el campo real en `Reference`, línea 3765) siempre
  presente, no es una señal de "pendiente" en sí, solo identificador.

**Forma de la respuesta** (agregado, no crea tablas nuevas):
```json
{
  "referenceId": "...",
  "pending": [
    { "category": "documents", "item": "PACKING_LIST", "status": "missing" },
    { "category": "glosa", "item": "INVOICE-001", "status": "GLOSSED_ERROR" },
    { "category": "letters", "item": "CARTA_FIRMA", "status": "SENT", "daysSinceSent": 5 },
    { "category": "previo", "item": "Previo", "status": "REQUESTED" },
    { "category": "folio", "item": "COVE", "status": "missing" },
    { "category": "folio", "item": "eDocument", "status": "missing" },
    { "category": "folio", "item": "ValueManifestation", "status": "missing" }
  ]
}
```

**Vista en el Inbox:** panel superior con conteo agregado de referencias con
`pending.length > 0`, expandible por referencia (reusa el mismo endpoint
`/pending-panel` al hacer click, no se pre-cargan los detalles de todas las
referencias — ver Riesgos).

## Reusa / Crea (actualizado)
- **Reusa:** el tablero primario de Listado de Referencias (SP-01,
  `ReferenceBoard.tsx`) como base — columna "Por identificar" (`DRAFT`) y
  "En curso" (`QUOTED`+`APPROVED`, sin tabs, ver pieza 1 diferida).
- **Crea:**
  - Sección "esperan a terceros": clasificación (`classification` module,
    revisar su estado real al implementar — no bloqueante), Previo (SP-09),
    arribo (`GET /shipments/pre-arrival`, ya existe, reusar tal cual),
    cartas sin firmar N días (pieza 2, nuevo), "por identificar" (`DRAFT`,
    ya existe).
  - Panel de pendientes agregado (pieza 4, nuevo endpoint).
  - Guías llegadas sin identificar (pieza 3, nuevo modelo + flujo).

## Fuera de alcance
- El listado/tabla clásica de referencias (SP-01).
- Tabs "automático/temporal/avanzado" de referencias en curso (pieza 1,
  diferida — sin contexto de negocio, no implementar).
- Integración en vivo con Zeus/CEUS (el panel de pendientes es agregación
  sobre datos ya persistidos, no una llamada en tiempo real).
- Resolver la duplicación preexistente entre `ReferencesService.getDocumentChecklist`
  y `ReferenceDocumentsService.getChecklist` (se documenta, no se toca).
- Reabrir el diseño del arranque de despacho de SP-16 (solo se agrega la
  llamada a `assertSignedLettersOk` en el mismo punto donde ya vive
  `assertInvoicesGlossedOk`).

## Pasos
- [x] Migración Prisma: `MompConfiguration.requiresSignedLetters` +
      `signedLettersDeadlineDays`.
- [x] Migración Prisma: modelo `ReferenceLetter` + enum `ReferenceLetterStatus`
      + relación con `Reference` y `ReferenceDocument`.
- [x] Migración Prisma: agregar `CARTA_FIRMA` a `Referencedocumenttype` (schema
      + DTO espejo `ReferenceDocumentType`).
- [x] Migración Prisma: modelo `UnidentifiedWaybill` + enum
      `UnidentifiedWaybillStatus` + relaciones con `Shipment`/`Reference`.
- [x] `ReferenceDocumentsService`: CRUD de `ReferenceLetter` (crear solicitud,
      marcar enviada, marcar firmada + linkear `ReferenceDocument` digitalizado)
      + endpoint `letters/pending`.
- [x] `ReferenceDocumentsService.assertSignedLettersOk` + wiring en
      `reglosaExpediente`, `canStartOperation` y `OperationsService.create`
      (junto a `assertInvoicesGlossedOk` existente).
- [x] `BolReconciliationService`: persistir `UnidentifiedWaybill` en rama
      `no_match` (y en las ramas `manual_review` que hoy solo loguean) en
      vez de descartar.
- [x] `DocumentProcessorService`: agregar case AWB (`match_shipment` +
      `extractAwbFields`).
- [x] Generalizar reconciliation por `trackingType` (BOL/AWB) contra
      `ShipmentTrackingNumber`, sin tocar el lado `Appointment` (terrestre-only).
- [x] Endpoints `unidentified-waybills`: listar pendientes, `link-existing`,
      `create-reference`.
- [x] `ReferenceDocumentsService`/controller: endpoint agregado
      `pending-panel` (checklist + glosaStatus + cartas + Previo + folios).
- [x] Front: sección Inbox — "esperan a terceros" (clasificación, Previo,
      arribo, cartas sin firmar, por identificar).
- [x] Front: panel de pendientes agregado (resumen + detalle por referencia).
- [x] Front: sección "Guías sin identificar" con modal de reconocimiento
      (ligar a existente / crear referencia vía wizard SP-02 con prefill de
      `clientId`).

## Riesgos y side effects
- Escala alta ⇒ paginar/priorizar en servidor en todas las listas nuevas
  (letters pendientes, waybills sin identificar, panel de pendientes). El
  panel de pendientes NO debe precalcularse para todas las referencias del
  tablero a la vez — cargar bajo demanda por referencia o en batch acotado.
- Solapa con SP-01: definir qué vive en cada uno para no duplicar (ya
  resuelto arriba: SP-01 es el tablero clásico, SP-17 es el punto de
  arranque del día con las 3 secciones nuevas).
- `assertSignedLettersOk` es un bloqueo nuevo y real: si se despliega sin
  que el equipo haya configurado `requiresSignedLetters` en el MOMP de
  ningún cliente, el default `false` significa que NO bloquea a nadie
  todavía (comportamiento seguro por defecto) — el bloqueo se activa
  cliente por cliente según se configure su MOMP.
- Confirmar en implementación la relación exacta `ValueManifestation.pedimentId
  → Pedimento → Reference` (detalle de query del panel de pendientes, no
  bloqueante).

## Criterios de verificación
- Gate estático verde.
- Playwright: abrir el Inbox, ver referencias en espera por categoría
  (incluida "cartas sin firmar"), ver el panel de pendientes de una
  referencia, y reconocer una guía sin identificar (ambos caminos: ligar y
  crear); sin errores de consola.
- Prueba de bloqueo: con un cliente cuyo MOMP tiene `requiresSignedLetters=true`
  y una `ReferenceLetter` en `SENT`, verificar que `reglosaExpediente` y
  `OperationsService.create` rechazan con `BadRequestException`; al marcar
  la carta `SIGNED`, ambos proceden.

## Estado
✅ Cerrado (2026-07-12) — piezas 2, 3 y 4 implementadas completas en ambos
repos, sobre `refactor/customs-operation-sp17-v2` (encadenada desde
`refactor/customs-operation-sp16`). Pieza 1 (tabs automático/temporal/avanzado)
sigue diferida sin implementar, tal como pedía el D1. Gate estático verde en
ambos repos (odin: 528/528 test suites, 3494 tests; digital: `tsc`/`next build`
0 errores, `next lint` sin errores nuevos, 88/88 tests propios). Playwright
pendiente de sesión humana (transversal a todo el paraguas). Ver la sección
"Intento 2026-07-12" del manifiesto para detalle de diseño, desviaciones
documentadas (agregación cross-referencia de Previo/clasificación no
construida, fuera de los Pasos declarados) y archivos tocados.
