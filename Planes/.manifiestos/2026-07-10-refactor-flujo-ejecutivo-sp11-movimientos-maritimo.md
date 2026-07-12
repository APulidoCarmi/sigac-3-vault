# Manifiesto — SP-11: Movimientos especializados — Marítimo (Importación y Exportación)

**Estado: ✅ IMPLEMENTADO (2026-07-12).** Ramas `refactor/customs-operation-sp11`
en ambos repos (`carmi-digital`, `carmi-odin-api-v2`), encadenadas desde
`refactor/customs-operation-sp14` (última rama con trabajo real de SP-07..SP-10
+ SP-14). Diff sin commitear en ambos repos, listo para revisión humana.

## Dependencias resueltas antes de implementar

- **SP-07**: dejó el tab Movimientos enrutando por tráfico
  (`ReferenceShipments.tsx` → `ReferenceShipmentsTerrestre.tsx` /
  `ReferenceShipmentsAereo.tsx` (SP-10) / placeholder MARITIMO en
  `ReferenceShipmentsTrafficPending.tsx`). Este sub-plan reemplaza ese
  placeholder MARITIMO por `ReferenceShipmentsMaritimo.tsx` y, como el
  placeholder queda sin ningún otro uso (AEREO ya no lo usaba desde SP-10), se
  eliminó el archivo (código muerto).
- **SP-10**: componente reusable de Asignación de Transporte
  (`components/shipments/transport-assignment/TransportAssignmentPanel.tsx`,
  props `{ shipmentId, readOnly?, onAssigned? }`, path-neutral) — reusado tal
  cual, sin ningún cambio, para Retorno de Vacío (import) y Toma de Vacío
  (export). Cero cambios de backend necesarios: `GET/PUT
  /shipments/:shipmentId/transport-assignment` ya es genérico por Shipment.
- **SP-14**: Solicitud de Fondos (`FundsRequest`, keyed por `operationId`, sin
  FK a `Shipment`/`Reference`). `SolicitudFondosDrawer.tsx` (props `{
  operationId, open, onClose }`) reusado tal cual desde Retorno de Vacío — ver
  desviación documentada abajo.

## Resumen de lo implementado

### Backend (`carmi-odin-api-v2`, rama `refactor/customs-operation-sp11`)

Todo greenfield, 5 módulos nuevos siguiendo el patrón ya establecido por
`air-revalidation`/`air-manifest`/`transport-assignment` (SP-10): módulo +
controller + dto + service + specs, `PrismaService` inyectado directo,
`IdentityJwtGuard` + `@CurrentUser()` (mismo patrón de auth que SP-10), respuesta
uniforme `{ success, data, timestamp }`.

**1. `src/maritime-revalidation/`** — Revalidación Marítima (#18): documental +
   pago ante la naviera. Modelo `MaritimeRevalidation` (schema `warehouse`, 1:1
   con `Shipment`): mismo shape exacto que `AirRevalidation` (status/
   paymentAmount/paymentReference/paymentProofAssetId referenciando
   `ShipmentAsset` ya existente, sin storage nuevo). `PUT`/`GET
   /shipments/:shipmentId/maritime-revalidation`.

**2. `src/maritime-guarantee-recovery/`** — Recuperación de Garantía: proceso
   documental, solicita el depósito/devolución de la garantía. Modelo
   `MaritimeGuaranteeRecovery` (1:1 con `Shipment`): status (`PENDING|
   DOCUMENTS_SUBMITTED|DEPOSIT_REQUESTED|RETURNED|COMPLETED`), depositAmount,
   depositReference, proofAssetId (→ `ShipmentAsset`). `PUT`/`GET
   /shipments/:shipmentId/maritime-guarantee-recovery`.

**3. `src/maritime-empty-return/`** — Retorno de Vacío: solicita transporte
   (delegado a `TransportAssignmentModule`, SP-10, mismo `shipmentId`, sin
   tocar ese módulo), da seguimiento a fumigación/lavado y solicita fondos
   (engancha con SP-14) hasta el acuse de vacío. Modelo `MaritimeEmptyReturn`
   (1:1 con `Shipment`) con máquina de etapas estricta (igual patrón que
   `AirManifest.stage`, sin saltos ni retrocesos): `TRANSPORT_REQUESTED →
   FUMIGATION_WASH_IN_PROGRESS → FUMIGATION_WASH_DONE → FUNDS_REQUESTED →
   EMPTY_ACKNOWLEDGED`, con timestamp dedicado por etapa alcanzada
   (`fumigationWashDoneAt`/`fundsRequestedAt`/`acknowledgedAt`). Transicionar a
   `FUNDS_REQUESTED` exige que `fundsRequestId` ya se haya capturado antes (vía
   `PATCH /maritime-empty-returns/:id`), igual que `ETA_CONFIRMED` en
   `AirManifest` exige `eta` ya seteada.
   - **`fundsRequestId`**: campo de referencia libre (`String? @db.Uuid`, **sin
     FK formal**) — mismo patrón que `FundsRequestItem.sourceRef` (SP-14).
     Decisión tomada por incompatibilidad de dominio: `FundsRequest` vive en el
     schema `public`, se identifica por `operationId` (no `shipmentId`/
     `referenceId`) y no tiene ninguna columna que apunte a `Shipment`. Añadir
     una FK real habría exigido tocar el modelo `FundsRequest` de SP-14 (ya
     cerrado) — fuera del alcance declarado de este sub-plan. Ver desviación
     de front abajo para cómo se captura este id en la práctica.
   - `POST`/`GET /shipments/:shipmentId/maritime-empty-return` (create/find),
     `PATCH /maritime-empty-returns/:id` (actualiza `fundsRequestId`/`notes`),
     `PATCH /maritime-empty-returns/:id/stage` (transición).

**4. `src/maritime-loading/`** — Carga de Mercancía (exportación): seguimiento
   del proceso de carga del contenedor. Modelo `MaritimeLoading` (N por
   `Reference` — el contenedor es la unidad de tracking a nivel Referencia en
   marítimo, igual que el Manifiesto en aéreo): `containerNumber` (regex ISO
   6346: 4 letras + 7 dígitos), `sealNumber`, `stage` (`SCHEDULED →
   LOADING_IN_PROGRESS → LOADED → SEALED`, mismo patrón de máquina de etapas
   que `AirManifest`, con timestamp por etapa). `POST`/`GET
   /references/:referenceId/maritime-loadings`, `PATCH
   /maritime-loadings/:id`, `PATCH /maritime-loadings/:id/stage`.

**5. `src/maritime-appointment/`** — Generar Cita (#20c): tipo de cita, puerto/
   recinto, fecha/hora, movimiento asociado. Modelo `MaritimeAppointment` (N
   por `Shipment` — un movimiento puede necesitar más de una cita): `type`
   (`PIS|OTRO`), `portOrFacility` (VarChar libre), `scheduledAt`. `POST`/`GET
   /shipments/:shipmentId/maritime-appointments`.
   - **Deliberadamente NO se reusó el `Appointment` de warehouse** (el que
     respalda `ReferenceAppointments.tsx` / `POST /appointments/from-reference`
     — dock/door scheduling con transportista/chofer/sellos). Ver "Desviación
     — #20c no reusa Appointments" abajo.

Migración única para los 5 dominios: `prisma migrate dev --name
add-maritime-movements-domain` →
`20260712062422_add_maritime_movements_domain` (aplicada limpio, `prisma
migrate status` verde antes y después). Relaciones inversas agregadas:
`Shipment.maritimeRevalidation/maritimeEmptyReturn/maritimeGuaranteeRecovery/
maritimeAppointments`, `Reference.maritimeLoadings`, `ShipmentAsset.
maritimeRevalidationPaymentProofs/maritimeGuaranteeRecoveryProofs`. Registro en
`src/app.module.ts` (bloque `// Movimientos marítimo (importación/exportación)
- SP-11`).

### Frontend (`carmi-digital`, rama `refactor/customs-operation-sp11`)

- **`ReferenceShipments.tsx`** (router de SP-07): la rama `MARITIMO` ahora
  renderiza `ReferenceShipmentsMaritimo` en vez del placeholder. La rama
  `AEREO` sigue apuntando a `ReferenceShipmentsAereo` (SP-10), sin cambios.
- **`ReferenceShipmentsTrafficPending.tsx`** — **eliminado**. Tras este cambio
  ya no lo importaba nadie (AEREO dejó de usarlo desde SP-10); mantenerlo
  habría sido código muerto.
- **`ReferenceShipmentsMaritimo.tsx`** (nuevo, `shipments/`, misma firma `{
  reference, onUpdate }` que `ReferenceShipmentsTerrestre`/`Aereo`): se
  ramifica por `reference.serviceType` (`'IMPORT' | 'EXPORT'`, mismo campo que
  usan `ReferenceHeroHeader`/`ReferenceRegimenCard` en el resto del repo):
  - **IMPORT**: input manual de `shipmentId` (mismo patrón/deviación que
    `ReferenceShipmentsAereo`, SP-10) revela tabs "Revalidación"/"Retorno de
    vacío"/"Recuperación de garantía".
  - **EXPORT**: `MaritimeLoadingBoard` (tablero a nivel Reference, siempre
    visible) + el mismo input manual de `shipmentId` revela tabs "Toma de
    vacío" (`TransportAssignmentPanel` tal cual)/"Ingreso de mercancía"
    (`MaritimeAppointmentPanel`, #20c).
- **`shipments/maritimo/MaritimeRevalidationPanel.tsx`** + `maritime-
  revalidation-schema.ts` — clon exacto de `AirRevalidationPanel`/`air-
  revalidation-schema.ts` (SP-10) adaptado a los hooks/api de maritime-
  revalidation. Mismo flujo de subida de comprobante vía `shipmentAssetsService`.
- **`shipments/maritimo/MaritimeGuaranteeRecoveryPanel.tsx`** + `maritime-
  guarantee-recovery-schema.ts` — mismo patrón, campos de depósito.
- **`shipments/maritimo/MaritimeEmptyReturnPanel.tsx`** — el componente más
  particular: crea el registro (si no existe), pinta la etapa actual con badge,
  botón "Avanzar etapa" para transiciones simples, y para la transición a
  `FUNDS_REQUESTED` expone: (a) un input manual de `operationId` + botón que
  abre `SolicitudFondosDrawer` (SP-14, reusado tal cual, sin tocar su código),
  y (b) un input manual de "ID de la Solicitud de Fondos" que, al guardarse,
  llama `PATCH /maritime-empty-returns/:id` con `fundsRequestId` — solo
  entonces se habilita el botón "Avanzar a Fondos solicitados". Debajo, reusa
  `TransportAssignmentPanel` tal cual para el transporte del contenedor vacío.
- **`shipments/maritimo/MaritimeLoadingBoard.tsx`** + `CreateMaritimeLoadingModal.tsx`
  + `maritime-loading-schema.ts` — clon de `AirManifestBoard`/
  `CreateAirManifestModal` (SP-10), simplificado: 4 etapas sin ninguna
  precondición de campo (a diferencia de `ETA_CONFIRMED` en aéreo), por eso no
  hay equivalente a `CaptureEtaModal`.
- **`shipments/maritimo/MaritimeAppointmentPanel.tsx`** + `maritime-appointment-
  schema.ts` — formulario de Generar Cita (#20c) + lista de citas ya
  generadas para el movimiento.
- Módulos API nuevos (`lib/api/modules/`): `maritime-revalidation.ts`,
  `maritime-guarantee-recovery.ts`, `maritime-empty-return.ts`, `maritime-
  loading.ts`, `maritime-appointment.ts` (mismo patrón que los de SP-10:
  interfaces + service object + `apiOdinClient` + `res.data.data`).
- Hooks nuevos: `hooks/use-maritime-revalidation.ts`, `use-maritime-guarantee-
  recovery.ts`, `use-maritime-empty-return.ts`, `use-maritime-loadings.ts`,
  `use-maritime-appointments.ts` (React Query, misma convención que
  `use-air-manifests.ts`/`use-air-revalidation.ts`).

## Desviación — #20c "Generar Cita" no reusa `Appointments`

El D1 del sub-plan proponía `references/components/tabs/ReferenceAppointments.tsx`
+ `GET /references/:id/appointments` como base. La investigación de
implementación encontró dos problemas que invalidan ese punto de partida:

1. **No existe tal endpoint.** `ReferenceAppointments.tsx` hace `fetch` directo
   a `${API_BASE_URL}/appointments?referenceId=...` (no
   `/references/:id/appointments`), sin servicio ni hook de React Query.
2. **El modelo `Appointment` (warehouse) es de otro dominio.** Tiene
   `type`/`status`/`appointmentMode`/`locationId`/`doorNumber`/`carrierId`/
   `driverId` — es agenda de muelle/puerta con transportista y chofer, **sin
   ningún campo de puerto/recinto aduanero**. Forzar #20c sobre este modelo
   habría exigido extender su enum de tipos y agregarle una columna nueva de
   puerto/recinto a un modelo que ya sirve a un dominio operativo distinto
   (control-tower de almacén).

**Decisión tomada:** modelo nuevo y liviano `MaritimeAppointment` (N por
`Shipment`) con exactamente los 4 campos que pide #20c (tipo, puerto/recinto,
fecha/hora, movimiento asociado), sin ninguna relación con `Appointment`. Es la
misma filosofía que ya usa `TransportAssignment`/`AirRevalidation` (SP-10):
modelo chico, propósito único, sin absorber un dominio ajeno.

## Desviación — selectores manuales (`shipmentId`, `operationId`, folio de fondos)

Mismo patrón/deviación ya documentado por SP-10: ninguno de los 3 endpoints a
nivel Shipment (Revalidación/Retorno de Vacío/Recuperación en import; Toma de
Vacío/Ingreso en export) tiene todavía una lista real de "movimientos
marítimos de esta referencia" cableada (fuera de alcance declarado por SP-10 y
heredado aquí). Se expone un input de texto simple para capturar el
`shipmentId`, igual que `ReferenceShipmentsAereo`. Los paneles reciben el
`shipmentId` como prop y no conocen su origen, así un sub-plan futuro puede
reemplazar el input por una lista real sin tocar ninguno de los 5 paneles
nuevos.

Adicionalmente, `MaritimeEmptyReturnPanel` necesita un `operationId` para abrir
`SolicitudFondosDrawer` (SP-14) — tampoco existe hoy una lista real de
"operaciones de este movimiento" (`ReferenceDetail.operations` llega sin tipar,
`any[]`). Se capturan manualmente tanto el `operationId` (para abrir el drawer)
como el folio/id de la Solicitud de Fondos resultante (para guardarlo como
`fundsRequestId` en `MaritimeEmptyReturn`), en vez de intentar inferirlos
automáticamente. `SolicitudFondosDrawer.tsx` (SP-14) se reusa sin ninguna
modificación.

## Verificación

- **Backend**: `npx prisma migrate status` → up to date, sin drift (1
  migración nueva aplicada limpio, cubre los 5 dominios). `pnpm jest src/
  maritime-revalidation src/maritime-empty-return src/maritime-guarantee-
  recovery src/maritime-loading src/maritime-appointment` → **48/48 tests en
  verde** (10 suites). `eslint` en los 5 módulos nuevos + `app.module.ts` → 0
  errores (import order corregido). `tsc --noEmit` → mismos errores
  preexistentes y no relacionados de siempre (twilio.service.spec, company-
  seals, audited.spec, e2e specs) — cero errores nuevos introducidos por SP-11.
- **Frontend**: `npx tsc --noEmit -p tsconfig.json` → **0 errores en todo el
  proyecto**. `eslint` en los 25 archivos nuevos/tocados → **0 errores, 0
  warnings**. Sin tests de Jest para estos componentes de UI (mismo criterio
  que SP-10: gate frontend = tsc + eslint, no hay suite de componentes en este
  repo para el tab Movimientos).
- **Playwright**: **pendiente de sesión humana** (mismo bloqueo transversal de
  toda la iniciativa, sin `dev_url` configurado para este cliente). Flujo a
  validar cuando haya entorno: en una referencia marítima de importación,
  recorrer Retorno de Vacío (transporte → fumigación/lavado → solicitud de
  fondos → acuse) y, en una de exportación, registrar una carga, avanzar sus
  etapas y generar una cita PIS — sin errores de consola.

## Bloqueos

Ninguno de infraestructura propio de este sub-plan. Hereda el bloqueo
transversal ya documentado por Fase 1/2 completas (Playwright sin `dev_url`).

## Archivos tocados

**Backend (`carmi-odin-api-v2`):**
- `prisma/schema.prisma` (5 modelos + 5 enums + relaciones inversas en
  `Shipment`/`Reference`/`ShipmentAsset`)
- `prisma/migrations/20260712062422_add_maritime_movements_domain/`
- `src/app.module.ts` (registro de los 5 módulos nuevos)
- `src/maritime-revalidation/**` (módulo completo + specs)
- `src/maritime-empty-return/**` (módulo completo + specs)
- `src/maritime-guarantee-recovery/**` (módulo completo + specs)
- `src/maritime-loading/**` (módulo completo + specs)
- `src/maritime-appointment/**` (módulo completo + specs)

**Frontend (`carmi-digital`):**
- `app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx`
  (rama MARITIMO)
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTrafficPending.tsx`
  (eliminado, código muerto tras este cierre)
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsMaritimo.tsx`
  (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/maritimo/MaritimeRevalidationPanel.tsx`
  + `maritime-revalidation-schema.ts` (nuevos)
- `app/(customerPortal)/references/components/tabs/shipments/maritimo/MaritimeGuaranteeRecoveryPanel.tsx`
  + `maritime-guarantee-recovery-schema.ts` (nuevos)
- `app/(customerPortal)/references/components/tabs/shipments/maritimo/MaritimeEmptyReturnPanel.tsx`
  (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/maritimo/MaritimeLoadingBoard.tsx`
  + `CreateMaritimeLoadingModal.tsx` + `maritime-loading-schema.ts` (nuevos)
- `app/(customerPortal)/references/components/tabs/shipments/maritimo/MaritimeAppointmentPanel.tsx`
  + `maritime-appointment-schema.ts` (nuevos)
- `lib/api/modules/maritime-revalidation.ts` (nuevo)
- `lib/api/modules/maritime-empty-return.ts` (nuevo)
- `lib/api/modules/maritime-guarantee-recovery.ts` (nuevo)
- `lib/api/modules/maritime-loading.ts` (nuevo)
- `lib/api/modules/maritime-appointment.ts` (nuevo)
- `hooks/use-maritime-revalidation.ts` (nuevo)
- `hooks/use-maritime-empty-return.ts` (nuevo)
- `hooks/use-maritime-guarantee-recovery.ts` (nuevo)
- `hooks/use-maritime-loadings.ts` (nuevo)
- `hooks/use-maritime-appointments.ts` (nuevo)
