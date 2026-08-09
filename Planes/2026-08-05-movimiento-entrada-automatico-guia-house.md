# Plan: Movimiento de entrada automático por Guía House

## Contexto

Origen: [[2026-07-31 - Revisión de Movimientos - Guía]] (tarea 2: "Guías (backend): al
crear una guía House, generar automáticamente un movimiento de entrada asociado"),
marcada como **PRIORIDAD** por el usuario el 2026-08-05.

Problema de fondo: en Querétaro (flujo aéreo) no existe hoy un "movimiento de entrada"
como tal — la entrada se da directo a la guía House y no hay forma de vincularle
transporte, previo de almacén, DGO, etc. Un avión trae una guía Master que engloba N
guías House (hasta ~20), y cada guía House equivale a un previo.

Área de código (mapeada con graphify + exploración dirigida):

- **Backend** (`carmi-odin-api-v2`): modelo `Guia` (`prisma/schema.prisma:7061-7113`,
  flujo exclusivamente aéreo hoy — status `RECIBIDA/EN_REVISION/REGIMEN_ASIGNADO/
  REFERENCIA_CREADA`) y servicio `src/guias/services/guias.service.ts` (`create()`
  L54, `createBulk()` L84, `linkGuiasToReference()` ~L399-536). El concepto
  "movimiento" en este código es el modelo `Shipment` (`prisma/schema.prisma:7380+`),
  con `flowType`/`warehouseRouting`/`metadata`. **Hoy `Guia` y `Shipment` no tienen
  relación alguna.**
- Precedente directo a replicar: `src/appointments/appointments.service.ts:
  createShipmentFromAppointment` (~L253-360) ya crea un `Shipment` esqueleto
  automáticamente (desde una cita), con solo los campos obligatorios resueltos por
  lookup + fallback, `metadata.origin` como discriminador, y `createdBy` con un actor
  del request o un UUID sentinel (`00000000-0000-0000-0000-000000000000`) si no hay
  usuario. Este plan sigue ese mismo patrón para Guía → Shipment.
- **Frontend** (`carmi-digital`): `ReferenceShipmentsTerrestre.tsx` lista movimientos
  filtrando `shipments` por `flowType`; `ReferenceShipmentsAereo.tsx` es hoy un stub
  (87 líneas) que solo muestra `AirManifestBoard` y admite asignar transporte a un
  `shipmentId` capturado a mano — **no está cableado a ningún movimiento real**.
  `ShipmentCreationModal.tsx`/`InboundForm.tsx` es el formulario manual terrestre
  existente; no rama por tipo de tráfico.

Nota: existe una rama remota sin mergear `feat/SCRUM-92-MOVIMIENTOS-DE-ENTRADAS-
EMBARQUES-INBOND` — se investigó y **no está relacionada**: trata sobre documentos
aduanales "In-Bond" (IT/IE/TE/TW), un concepto distinto que comparte solo la palabra
"inbond". No representa trabajo previo sobre esta feature.

## Decisiones tomadas

- **Trigger**: el `Shipment` esqueleto se crea al dar de alta la Guía House
  (`create()` y `createBulk()`), **aunque la guía todavía no tenga `referenceId`**.
  Cuando después se vincule a una referencia vía `linkGuiasToReference()`, ese mismo
  `Shipment` actualiza su `referenceId` (no se crea uno nuevo).
  - Porqué: el usuario prefirió fidelidad al pedido original ("al crearse la guía")
    sobre esperar a que exista referencia.
- **Cardinalidad 1:1 estricta** Guía↔Shipment: se agrega `guiaId` único (FK) en
  `Shipment` vía migración de Prisma CLI (`npx prisma migrate dev --name
  add-guia-shipment-link`, conforme a la regla global de este entorno — nunca SQL de
  migración a mano). La unicidad en BD es la garantía de idempotencia.
- **Sin snapshot — lectura en vivo desde `Guia`**: `Shipment` NO copia campos de la
  guía (Master, House, régimen, clave de pedimento, origen, destinatario, peso,
  piezas, etc.). El front resuelve esos campos vía join a `Guia.guiaId`. Si la guía
  cambia después (ej. régimen asignado), el movimiento siempre refleja el dato
  actual sin lógica de sincronización.
- **`Shipment` esqueleto, no un modelo/tipo de movimiento aparte**: se descartó
  explícitamente crear un `flowType` nuevo tipo `INBOUND_GUIA_AEREA` o un modelo
  paralelo. Se sigue el patrón ya existente de `appointments.service.ts`:
  `flowType: INBOUND`, `warehouseRouting: ENTRY`, `maneuverType: N_A`, y el
  discriminador va en `metadata.origin: 'AIR_GUIDE'` (+ `guiaId`, `guiaMaster`,
  `guiaHouse` en el metadata, igual que `APPOINTMENT` guarda `appointmentId`/
  `confirmationCode`).
  - Porqué: de las ~40 columnas de `Shipment` solo `transportModeId`, `statusId` y
    `createdBy` son obligatorias; el resto ya es nullable y está pensado para
    llenarse progresivamente durante la operación de almacén (el previo) — no hay
    necesidad de "todos los campos de un movimiento de entrada" al crear el
    esqueleto.
  - Porqué (decisión explícita del usuario): **almacén debe seguir trabajando sobre
    `Shipment`** — si se creara un modelo/tipo paralelo, `WarehousemanTasksDashboard`
    y el resto de la tubería de previos/DGO/subdivisión tendrían que re-cablearse
    para reconocerlo, duplicando lógica que ya funciona.
- **Resolución de campos obligatorios del Shipment**, replicando exactamente el
  patrón de `createShipmentFromAppointment`:
  - `transportModeId`: lookup de `TransportMode` por `code: 'AIR'` (código ya usado
    en `legacy-receipts.service.ts` para detectar shipments aéreos), con el mismo
    fallback a "primer TransportMode activo" si no se encuentra, como hace el flujo
    de citas con GROUND/TRUCK/ROAD.
  - `statusId`: lookup de `ShipmentStatus` por `code: 'PENDING'` (mismo estatus
    inicial que usa el flujo de citas).
  - `createdBy`: el actor de la request de alta de guía (`dto.registeredBy`, campo
    que ya existe en `CreateGuiaDto`/`BulkCreateGuiasDto`), con fallback al mismo
    UUID sentinel `00000000-0000-0000-0000-000000000000` que usa `appointments.
    service.ts` cuando no hay usuario.
  - `metadata`: `{ origin: 'AIR_GUIDE', guiaId, guiaMaster, guiaHouse }`.
  - `notes`: texto autogenerado análogo a "Movimiento generado automáticamente desde
    guía House {guiaHouse}".
- **Se dispara en ambos endpoints** del servicio de guías: `create()` (alta
  individual) y `createBulk()` (alta masiva desde manifiesto) — cualquier guía House
  que entre al sistema por cualquier canal genera su Shipment esqueleto.
- **Fallback de lookups obligatorios → falla toda la operación**: a diferencia de
  `createShipmentFromAppointment` (que ante `TransportMode`/`ShipmentStatus` no
  encontrados solo loguea warning y devuelve `null`, dejando la cita sin Shipment),
  aquí el Shipment **es obligatorio** para el flujo aéreo. Si no se puede resolver
  `TransportMode` código `AIR` (ni el fallback a primer activo) o `ShipmentStatus`
  código `PENDING`, se aborta toda la transacción (rollback de la guía incluida) en
  vez de crear una guía huérfana sin movimiento.
  - Porqué: decisión explícita del usuario — prefiere que un seed/entorno roto
    bloquee el alta de guías antes que dejar guías sin su ancla de movimiento.
- **Idempotencia por verificación previa, no por captura de error**: antes de crear
  el Shipment esqueleto se hace `findUnique` por `guiaId`; si ya existe, se omite la
  creación (no se intenta crear y capturar el error de unique constraint P2002).
  - Porqué: el usuario prefirió el enfoque explícito/legible sobre confiar en el
    error de BD, aunque cueste una consulta extra por alta.
- **Vista aérea reutiliza el endpoint existente `GET /shipments/reference/:id`**
  (el mismo que ya usa `ReferenceShipmentsTerrestre.tsx`), ampliándolo para incluir
  la relación a `Guia` cuando aplique. El front filtra los resultados por
  `metadata.origin === 'AIR_GUIDE'`. No se crea un endpoint dedicado nuevo (p.ej.
  `/shipments/reference/:id/air-guides`).
  - Porqué: un solo endpoint sirve a ambos tráficos; evita duplicar la ruta de
    consulta de shipments por referencia.
- **La pestaña "Movimientos" del detalle de referencia debe mostrar las guías por
  defecto, sin pasos manuales**: confirmado el wiring actual —
  `ReferenceDetailShell.tsx` → sección `"shipments"` → `ReferenceShipments.tsx` (lee
  `reference.trafficType?.code === 'AEREO'`) → `ReferenceShipmentsAereo.tsx`. Hoy esa
  pantalla solo pide un `shipmentId` tecleado a mano para habilitar
  `TransportAssignmentPanel`/`AirRevalidationPanel`; ese input manual se **elimina**
  (no se deja como alternativa) — la lista real de guías/Shipments se carga sola al
  entrar a la pestaña, y cada fila permite abrir esos paneles para *ese* shipment sin
  que el usuario tenga que conocer o teclear ningún ID.
  - Porqué: el requerimiento explícito del usuario es que al abrir el detalle de una
    referencia aérea y entrar a Movimientos, las guías **ya estén ahí** — no que
    exista la capacidad técnica de listarlas solo si alguien más provee un ID.
  - `AirManifestBoard` (datos a nivel Manifiesto, no Shipment) se conserva sin
    cambios y convive con el nuevo listado — no es redundante, es otro nivel de
    tracking (ver Contexto/graphify: comentario explícito en el propio componente).
- **La vista aérea de movimientos en el detalle de referencia es de solo lectura**
  para los campos de guía (se deduce de "lectura en vivo, sin snapshot": si el dato
  vive en `Guia` y no en `Shipment`, editarlo implica editar la Guía desde su propio
  formulario, no desde la vista de movimiento). No se reutiliza `ShipmentCreationModal`/
  `InboundForm` para el caso aéreo.
- **Visible en UI, adaptado por tráfico** (confirmado en sesión previa — contradice
  la idea original de la reunión de dejarlo "transparente/oculto"): el movimiento
  aéreo sí se muestra en el listado de movimientos de entrada del detalle de
  referencia, con campos de guía en vez de campos terrestres.

## Fuera de alcance

- Backfill de guías ya existentes en producción sin `Shipment` asociado (solo aplica
  hacia adelante).
- Vinculación de facturas a la guía House (tarea 1 y 3 de la reunión — plan aparte).
- Formulario de citas marítimas/aéreas (tarea 6 de la reunión).
- Avatar de responsable en cabecera de referencia (tarea 7, depende del módulo de
  equipos de Elian).
- Acción de marcar referencia como urgente (tarea 8).
- Extender `ShipmentCreationModal`/`InboundForm` para permitir crear/editar
  manualmente un movimiento de tipo guía aérea — el flujo es 100% automático.
- Cualquier cambio sobre la rama remota `feat/SCRUM-92-...-INBOND` (no relacionada,
  ver Contexto).
- Lógica de "previo" en sí (qué hace almacén con el Shipment una vez creado) — este
  plan solo entrega el ancla; el flujo de previo ya existente para Shipments debe
  poder operar sobre él sin cambios adicionales (ver Riesgos).

## Pasos

- [x] **Migración Prisma**: agregar `guiaId String? @unique` + relación `Guia` en el
      modelo `Shipment` (y su lado inverso opcional en `Guia`), vía
      `npx prisma migrate dev --name add-guia-shipment-link`. No escribir el SQL a
      mano.
- [x] **Backend — helper de creación**: nuevo método privado en `GuiasService` (o
      servicio colaborador) que, dada una `Guia` recién creada, resuelve
      `transportModeId` (AIR + fallback), `statusId` (PENDING), `createdBy`
      (`registeredBy` + fallback sentinel), verifica con `findUnique` por `guiaId`
      que no exista ya un Shipment (idempotencia) y crea el `Shipment` esqueleto con
      `flowType: INBOUND`, `warehouseRouting: ENTRY`, `maneuverType: N_A`,
      `metadata.origin: 'AIR_GUIDE'`, `guiaId`, `referenceId` (si la guía ya lo
      tiene), `clientCompany` = `guia.companyId`. Si `transportModeId` o `statusId`
      no se pueden resolver, lanzar excepción (no `return null` como en el
      precedente de citas) para que la transacción completa haga rollback.
- [x] **Backend — enganchar en `create()`**: envolver `checkDuplicate` + `guia.create`
      + creación del shipment esqueleto en una transacción (`$transaction(async tx =>
      ...)`), ya que hoy `create()` no es atómico.
- [x] **Backend — enganchar en `createBulk()`**: convertir el
      `$transaction(createPromises)` (array de creates) a una transacción
      interactiva que, por cada fila, cree la guía y su shipment esqueleto; resolver
      `transportModeId`/`statusId` **una sola vez fuera del loop** (no N lookups).
      Vigilar el timeout de transacción con manifiestos grandes (mismo patrón de
      `{ timeout: 20_000, maxWait: 10_000 }` que usa `appointments.service.ts`).
- [x] **Backend — propagar referencia**: en `linkGuiasToReference()`, cuando una
      guía sin referencia se vincula, actualizar el `referenceId` del `Shipment`
      esqueleto ya existente (no crear uno nuevo).
- [x] **Backend — ampliar `GET /shipments/reference/:id`**: incluir la relación a
      `Guia` en la respuesta (join por `guiaId`) para que el front pueda leer
      Master/House/régimen/origen/destinatario/peso/piezas/etc. sin una segunda
      llamada. Es el mismo endpoint que ya consume `ReferenceShipmentsTerrestre.tsx`
      (`fetchShipments()`); no se crea ruta nueva.
- [x] **Frontend — cablear `ReferenceShipmentsAereo.tsx`**: reemplazar el input
      manual de `shipmentId` (líneas ~50-68 hoy) por el listado real de movimientos
      de entrada por guía House de la referencia, consumiendo el endpoint ampliado y
      filtrando por `metadata.origin === 'AIR_GUIDE'` / `guiaId` no nulo. La lista se
      carga automáticamente al entrar a la pestaña (sin acción del usuario), solo
      lectura para los campos de guía, mostrando el estatus del Shipment (equivalente
      al estado del previo). Cada fila abre `TransportAssignmentPanel`/
      `AirRevalidationPanel` para el `shipmentId` de esa fila (reemplaza la necesidad
      de teclearlo). `AirManifestBoard` se mantiene tal cual, sin tocar.
- [x] **Verificación end-to-end**: alta individual y masiva de guías genera su
      Shipment (o falla completa si no se resuelven los lookups obligatorios); el
      detalle de referencia aérea lo muestra; los consumidores reales de campos de
      `Shipment` en frontend (`detail-overview.tsx` de movements/[id],
      `ShipmentCreationModal.tsx`, `movement-form/*`, `AppointmentDetailSheet.tsx`)
      siguen funcionando sin errores con estos Shipments esqueleto (campos
      mayormente null).

## Cierre

**Completado 2026-08-05.** Commits: `carmi-odin-api-v2` `607650ea`,
`carmi-digital` `fedd2f76` (ambos en `feat/movimiento-entrada-automatico-guia-house`,
pusheados, sin PR abierto todavía). Ver [[Sesiones/2026-08-05 - Implementación movimiento-entrada-automatico-guia-house]]
para el detalle de decisiones y hallazgos — el alcance final creció bastante
respecto a lo escrito arriba (llenado de campos reales del Shipment,
resincronización, fix de vinculación incremental de DGO, y varios bugs
preexistentes encontrados y corregidos en el camino).

## Riesgos y side effects a vigilar

- `createBulk()` pasa de un array de `prisma.guia.create` en `$transaction` a una
  transacción interactiva más pesada (guía + shipment por fila) — vigilar timeout y
  performance con manifiestos de muchas guías House.
- Consumidores de `Shipment` en frontend que asuman campos no nulos (`carrierId`,
  `maneuverType` más allá de `N_A`, `warehouseDoorId`, etc.) deben tolerar un
  Shipment esqueleto con casi todo en null. Verificado que `WarehousemanTasksDashboard.tsx`
  **no** aplica aquí — no lee campos de `Shipment` (usa `ActiveTaskMetrics` de
  `warehousemanService.ts`). Los consumidores reales a vigilar son
  `detail-overview.tsx` (movements/[id]), `ShipmentCreationModal.tsx`,
  `movement-form/*` y `AppointmentDetailSheet.tsx`.
- Confirmar que `Guia.companyId` es el mismo `Company` que debe conectarse en
  `Shipment.clientCompanyId` (no se validó a nivel de datos, solo de schema).
- Reintentos de alta (duplicados) no deben generar un segundo Shipment — se
  resuelve con `findUnique` por `guiaId` antes de crear (ver Decisiones), no
  capturando el error de unique constraint.
- El fallback silencioso de `createShipmentFromAppointment` (warning + `null` si no
  hay `TransportMode`/`ShipmentStatus`) **no aplica aquí**: en este flujo esos
  lookups fallidos deben abortar toda la transacción de alta de guía.
- No confundir con la rama remota `feat/SCRUM-92-...-INBOND` (documentos In-Bond
  aduanales) al buscar trabajo previo relacionado.

## Criterios de verificación

- Crear una guía individual (`POST` de `create()`) → existe un `Shipment` con ese
  `guiaId`, `flowType: INBOUND`, `warehouseRouting: ENTRY`, `metadata.origin ===
  'AIR_GUIDE'`.
- Alta masiva de N guías (`createBulk()`, ej. manifiesto con 5 house) → N Shipments
  creados, cada uno con `guiaId` único, sin duplicados.
- `linkGuiasToReference()` sobre una guía sin referencia previa → el `Shipment`
  existente actualiza su `referenceId` (no se crea uno nuevo).
- Front: abrir el detalle de una referencia aérea → entrar a la pestaña
  "Movimientos" → la lista de guías House con sus datos (leídos en vivo de `Guia`)
  aparece **de inmediato, sin teclear ningún ID ni acción manual previa**, sin
  controles de edición sobre los campos de guía. El input manual de `shipmentId`
  ya no existe.
- `detail-overview.tsx`, `ShipmentCreationModal.tsx`, `movement-form/*` y
  `AppointmentDetailSheet.tsx` siguen cargando y operando sin errores sobre
  Shipments creados por este flujo (campos null no rompen la vista).
- Reintentar la creación de la misma guía (duplicado ya detectado por
  `checkDuplicate`) no genera un segundo Shipment (verificado vía `findUnique` por
  `guiaId` antes de crear).
- Si `TransportMode('AIR')` o `ShipmentStatus('PENDING')` no se pueden resolver
  (simulando seed roto), la creación de la guía falla por completo (rollback) en
  vez de crear una guía sin Shipment.
