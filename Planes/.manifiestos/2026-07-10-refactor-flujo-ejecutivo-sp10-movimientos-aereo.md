# Manifiesto — SP-10: Movimientos especializados — Aéreo (Importación)

**Estado: ✅ IMPLEMENTADO (2026-07-12).** Ramas `refactor/customs-operation-sp10`
en ambos repos (`carmi-digital`, `carmi-odin-api-v2`), encadenadas desde
`refactor/customs-operation-sp09` (última rama con trabajo real de SP-07/08/09).
Diff sin commitear en ambos repos, listo para revisión humana.

## Resumen de lo implementado

### Backend (`carmi-odin-api-v2`, rama `refactor/customs-operation-sp10`)

Todo greenfield — ninguna de las 3 features existía ni parcialmente. Se
siguió el patrón de módulo ya establecido por `src/previo/**`/`src/dgo/**`
(módulo + controller + dto + service + spec, `PrismaService` inyectado
directo, respuesta uniforme `{ success, data, timestamp }`).

**1. `src/transport-assignment/`** — Asignación de Transporte (reusable, ver
   sección de contrato de reuso abajo).
   - Modelo `TransportAssignment` (schema `warehouse`, 1:1 con `Shipment`
     vía `shipmentId` único): `carrierName`, `contactName/Phone/Email`,
     `route` (editables por el ejecutivo) + `arrivalState`/`loadState`/
     `departureState` (`TransportSemaphoreState`: `PENDING|GREEN|RED`,
     **solo lectura** para el ejecutivo — no hay endpoint de escritura para
     estos 3 campos, es responsabilidad de un rol/sub-plan futuro "trámite
     y despacho" no diseñado aún).
   - `PUT /shipments/:shipmentId/transport-assignment` (upsert, DTO
     `AssignTransportDto` sin campos de semáforo) / `GET
     /shipments/:shipmentId/transport-assignment` (registro o `null`).
   - Migración: `20260712053752_add_transport_assignment_domain`.

**2. `src/air-revalidation/`** — Revalidación Aérea.
   - Modelo `AirRevalidation` (schema `warehouse`, 1:1 con `Shipment`):
     `status` (`AirRevalidationStatus`: `PENDING|DOCUMENTS_SUBMITTED|PAID|
     COMPLETED`), `paymentAmount`, `paymentReference`, `paymentProofAssetId`
     (FK a `ShipmentAsset` ya existente — **no se creó storage nuevo**, el
     comprobante se sube con el endpoint ya existente `POST
     /shipments/:id/assets` tipo `DOCUMENT` y solo se referencia el id aquí),
     `registeredBy/At`, `notes`.
   - `PUT /shipments/:shipmentId/air-revalidation` (upsert, valida que
     `paymentProofAssetId` pertenezca al mismo shipment) / `GET
     /shipments/:shipmentId/air-revalidation` (incluye `paymentProofAsset:
     {id,title,url,fileId}|null`).
   - Migración: `20260712053825_add_air_revalidation_domain`.

**3. `src/air-manifest/`** — Manifiesto de Carga + Tablero de planeación
   diaria.
   - Modelo `AirManifest` (schema `warehouse`, N:1 con `Reference` — el
     Manifiesto, no el Shipment, es la unidad de tracking a nivel Referencia
     en aéreo, según el sub-plan): `masterGuideNumber` (regex 12 dígitos),
     `houseGuideNumber` (regex 10-11 dígitos, opcional), `carrierSource`
     (`WAREHOUSE|FEDEX|DHL`), `stage` (`AirManifestStage`, 7 valores en
     orden estricto: `RECEIVED→ETA_CONFIRMED→ARRIVAL_CONFIRMED→UNLOADED→
     DECONSOLIDATED→ASSIGNED_TO_PREVIO_TABLE→READY_FOR_PREVIO`, con un
     timestamp dedicado por etapa alcanzada), `eta`, `weightKg` (nullable —
     "llega después de la báscula del almacén"), `notes`.
   - `POST /references/:referenceId/air-manifests` (crea en `RECEIVED`) /
     `GET /references/:referenceId/air-manifests` (lista, tablero) / `PATCH
     /air-manifests/:id` (edita campos, incluido `weightKg` post-báscula) /
     `PATCH /air-manifests/:id/stage` (transición **estrictamente
     secuencial**, sin saltos ni retrocesos; `ETA_CONFIRMED` exige `eta` ya
     seteado vía el PATCH normal antes de transicionar).
   - Migración: `20260712053905_add_air_manifest_domain`.

Registro: `TransportAssignmentModule`, `AirRevalidationModule`,
`AirManifestModule` agregados a `src/app.module.ts`. Relaciones inversas
mínimas agregadas: `Shipment.transportAssignment`, `Shipment.airRevalidation`,
`ShipmentAsset.airRevalidationPaymentProofs`, `Reference.airManifests`.

**Desviación documentada**: el spec original pedía copiar el patrón de auth
de `previo.module.ts`. Al implementar se encontró que ni `PrevioModule` ni
`DgoModule` ni `ShipmentInbondController` tienen guard de auth real cableado
(este último con un `// TODO: Extract userId from JWT` y `'system'`
hardcodeado). Por la regla dura de "cero TODOs sin resolver", se usó en su
lugar el patrón de auth real y funcional de `ReferencesController`
(`IdentityJwtGuard` + `@CurrentUser()`), consistente con módulos adyacentes
a Shipment/Reference que sí protegen sus rutas.

### Frontend (`carmi-digital`, rama `refactor/customs-operation-sp10`)

- **`app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx`**
  (router de SP-07): la rama `AEREO` ahora renderiza
  `ReferenceShipmentsAereo` en vez del placeholder
  `ReferenceShipmentsTrafficPending`. La rama `MARITIMO` queda intacta
  (sigue con el placeholder, es responsabilidad de SP-11).
- **`ReferenceShipmentsAereo.tsx`** (nuevo, mismo directorio `shipments/`,
  misma firma `{ reference, onUpdate }` que `ReferenceShipmentsTerrestre`):
  compone `AirManifestBoard` (tablero principal) + un input manual de
  `shipmentId` que, al tener valor, revela `TransportAssignmentPanel` y
  `AirRevalidationPanel` en tabs ("Transporte"/"Revalidación").
- **`AirManifestBoard.tsx` + `CreateAirManifestModal.tsx` +
  `CaptureEtaModal.tsx`** (`shipments/aereo/`): tabla de manifiestos con
  badge de etapa en español, botón "Nuevo manifiesto" (zod valida longitud
  de guías), botón "Avanzar etapa" (calcula la siguiente del array
  ordenado; si la siguiente es `ETA_CONFIRMED` y `eta` es `null`, abre antes
  `CaptureEtaModal`).
- **`AirRevalidationPanel.tsx`** (`shipments/aereo/`): formulario de
  status/pago/comprobante, reusa `shipmentAssetsService.create` (tipo
  `DOCUMENT`) para el upload antes de referenciar el `assetId`.
- **`components/shipments/transport-assignment/`** (path neutral, fuera de
  `references/**` — ver contrato de reuso abajo): `TransportAssignmentPanel.tsx`,
  `SemaphoreIndicator.tsx`, `schema.ts`.
- Módulos API nuevos (`lib/api/modules/`): `transport-assignment.ts`,
  `air-revalidation.ts`, `air-manifest.ts` (patrón de `shipment-inbond.ts`:
  interfaces + service object + `apiOdinClient` + `res.data.data`).
- Hooks nuevos: `hooks/use-transport-assignment.ts`,
  `hooks/use-air-revalidation.ts`, `hooks/use-air-manifests.ts` (React Query,
  convención de `hooks/use-shipments.ts`).

**Desviación documentada — selector manual de `shipmentId`**: Asignación de
Transporte y Revalidación Aérea son endpoints a nivel `Shipment`, pero SP-10
no crea ni conecta una lista real de "shipments aéreos de esta referencia"
(fuera de alcance declarado: "aquí solo el tracking"). El tab expone un input
de texto simple ("Introduce el ID del movimiento aéreo") en vez de un
selector cableado a una lista real. Los paneles (`TransportAssignmentPanel`/
`AirRevalidationPanel`) están completamente desacoplados de esta limitación
— reciben `shipmentId` como prop y no conocen el origen del valor — así que
un sub-plan futuro puede reemplazar el input por una lista real sin tocar
ninguno de los dos paneles.

## Contrato de reuso para SP-11 (Asignación de Transporte, movimientos marítimo)

**No releer el código — usar tal cual esta forma.**

Componente: `components/shipments/transport-assignment/TransportAssignmentPanel.tsx`
(path neutral, sin imports de `references/**`, importable directo desde
cualquier tab de tráfico).

```ts
interface TransportAssignmentPanelProps {
  shipmentId: string;
  readOnly?: boolean; // true: oculta el form de edición, solo muestra datos + semáforo (roles de solo consulta)
  onAssigned?: () => void; // callback tras un assign (PUT) exitoso, para que el padre haga refetch si lo necesita
}
```

Componente auxiliar reusable también: `components/shipments/transport-assignment/SemaphoreIndicator.tsx`:
```ts
interface SemaphoreIndicatorProps {
  label: string;
  state: 'PENDING' | 'GREEN' | 'RED';
}
```
(colores: `PENDING`=gris/muted, `GREEN`=`text-green-600`/`bg-green-500`,
`RED`=`text-destructive`/`bg-destructive`, consistente con el patrón de
disponibilidad ya usado en `ReferenceShipmentsTerrestre.tsx`).

Backend consumido por el panel (ya implementado, genérico por `Shipment`,
**sin ningún acoplamiento a tráfico** — SP-11 no necesita tocar backend para
reusarlo):
- `GET /shipments/:shipmentId/transport-assignment` → registro o `null`.
- `PUT /shipments/:shipmentId/transport-assignment` → `AssignTransportDto
  { carrierName: string; contactName?: string; contactPhone?: string;
  contactEmail?: string; route?: string }`.
- El semáforo (`arrivalState`/`loadState`/`departureState`,
  `'PENDING'|'GREEN'|'RED'`) es de solo lectura vía este mismo GET — no hay
  endpoint de escritura todavía (rol "trámite y despacho", fuera de alcance
  de SP-10 y SP-11 por igual, hasta que exista ese sub-plan).

Hook a reusar: `hooks/use-transport-assignment.ts` (React Query,
`useQuery`/`useMutation` ya cableados a los 2 endpoints).

**Para SP-11**: basta con `import { TransportAssignmentPanel } from
"@/components/shipments/transport-assignment/TransportAssignmentPanel"` y
pasarle el `shipmentId` real del movimiento marítimo — cero cambios de
código en el componente. Si el tab marítimo necesita solo-lectura (ej. un
rol de consulta), usar `readOnly`.

## Verificación

- **Backend**: `npx prisma migrate status` → up to date, sin drift (3
  migraciones nuevas aplicadas limpio). `pnpm test src/transport-assignment
  src/air-revalidation src/air-manifest` → **30/30 tests en verde** (6
  suites). `eslint` en los 3 módulos nuevos → 0 errores.
- **Frontend**: `npx tsc --noEmit -p tsconfig.json` → **0 errores en todo el
  proyecto**. `eslint` en los 11 archivos nuevos/tocados → **0 errores, 0
  warnings**.
- **Playwright**: **pendiente de sesión humana** (mismo bloqueo transversal
  de toda la Fase 1/2: sin `dev_url` configurado para este cliente). Flujo a
  validar cuando haya entorno: en una referencia aérea de importación, crear
  un manifiesto, avanzar sus etapas (incluida la captura de ETA), asignar
  transporte y ver el semáforo de solo lectura, y registrar una revalidación
  con comprobante de pago — sin errores de consola.

## Bloqueos

Ninguno de infraestructura propio de este sub-plan. Hereda el bloqueo
transversal ya documentado por SP-06/07/08/09 (Playwright sin `dev_url`).

## Archivos tocados

**Backend (`carmi-odin-api-v2`):**
- `prisma/schema.prisma` (3 modelos + 4 enums + relaciones inversas)
- `prisma/migrations/20260712053752_add_transport_assignment_domain/`
- `prisma/migrations/20260712053825_add_air_revalidation_domain/`
- `prisma/migrations/20260712053905_add_air_manifest_domain/`
- `src/app.module.ts` (registro de los 3 módulos nuevos)
- `src/transport-assignment/**` (módulo completo + specs)
- `src/air-revalidation/**` (módulo completo + specs)
- `src/air-manifest/**` (módulo completo + specs)

**Frontend (`carmi-digital`):**
- `app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx` (rama AEREO)
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsAereo.tsx` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/AirManifestBoard.tsx` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/CreateAirManifestModal.tsx` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/CaptureEtaModal.tsx` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/air-manifest-schema.ts` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/AirRevalidationPanel.tsx` (nuevo)
- `app/(customerPortal)/references/components/tabs/shipments/aereo/air-revalidation-schema.ts` (nuevo)
- `components/shipments/transport-assignment/TransportAssignmentPanel.tsx` (nuevo)
- `components/shipments/transport-assignment/SemaphoreIndicator.tsx` (nuevo)
- `components/shipments/transport-assignment/schema.ts` (nuevo)
- `lib/api/modules/transport-assignment.ts` (nuevo)
- `lib/api/modules/air-revalidation.ts` (nuevo)
- `lib/api/modules/air-manifest.ts` (nuevo)
- `hooks/use-transport-assignment.ts` (nuevo)
- `hooks/use-air-revalidation.ts` (nuevo)
- `hooks/use-air-manifests.ts` (nuevo)
