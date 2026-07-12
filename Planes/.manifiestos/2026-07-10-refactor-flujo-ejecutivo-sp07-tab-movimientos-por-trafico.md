# Manifiesto — SP-07: Tab Movimientos — rediseño por tráfico + vínculo flexible con DGO

**Estado: ✅ IMPLEMENTADO (2026-07-11).** Ramas `refactor/customs-operation-sp07`
en ambos repos (`carmi-digital`, `carmi-odin-api-v2`), encadenadas desde
`refactor/customs-operation-sp06` (última rama con trabajo real de la Fase 1).
Diff sin commitear en ambos repos, listo para revisión humana.

## Resumen de lo implementado

### Backend (`carmi-odin-api-v2`, rama `refactor/customs-operation-sp07`)

- **`src/shipments/services/shipments.service.ts` — `findByReference(referenceId)`**
  (usado por `GET /shipments/reference/:referenceId`, el endpoint que consume el
  tab Movimientos vía fetch directo):
  - Se agregó `invoiceShipmentLinks` al `include` de la query (select mínimo:
    `invoice.id`, `invoice.dgoId`, `invoice.dgo.{id, dgoNumber, status}`).
  - En el `.map()` de enriquecimiento (mismo bloque donde ya se agregaban
    `osdReports`/`osdByPartida`), se añadió el cálculo de `linkedDgos`: dedupe
    por `dgo.id` de todos los DGOs alcanzables desde el shipment a través de
    `InvoiceShipmentLink → Invoice.dgoId → Dgo` (un shipment puede tener varias
    facturas del mismo DGO, o de varios DGOs distintos — no se asume 1 a 1). El
    campo crudo `invoiceShipmentLinks` se descarta del payload final
    (destructuring), solo se expone `linkedDgos: Array<{id, dgoNumber, status}>`
    por shipment.
  - **No se creó un endpoint nuevo.** El modelo de vínculo DGO↔movimiento ya
    resuelto por SP-05 (`Invoice.dgoId` + `InvoiceShipmentLink` M:N con
    `Shipment`) no tenía ningún endpoint que lo agregara — se resolvió
    enriqueciendo el endpoint de listado ya consumido por el tab, evitando N+1
    llamadas desde el front para una lista que puede tener cientos de
    movimientos por referencia.
  - Patrón reusado tal cual ya existía en el archivo (líneas ~697/1316/1831) de
    `invoiceShipmentLinks: { include: { invoice: ... } }` seguido de
    `.map(link => link.invoice)`.

### Frontend (`carmi-digital`, rama `refactor/customs-operation-sp07`)

- **`hooks/useReferenceDetail.ts`**: se agregó `trafficTypeId` y `trafficType:
  {id, code, description} | null` a la interfaz `ReferenceDetail`. El backend
  (`references.service.ts`, `findOne`) ya incluía `trafficType` en el `select`
  desde antes de este sub-plan — solo faltaba exponerlo en el tipo del front,
  no hizo falta tocar el service.
- **`app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx`**
  (reescrito): pasó de contener toda la lógica del tab a ser un router delgado
  por `reference.trafficType?.code`:
  - `AEREO` → `ReferenceShipmentsTrafficPending` (`trafficCode="AEREO"`).
  - `MARITIMO` → `ReferenceShipmentsTrafficPending` (`trafficCode="MARITIMO"`).
  - `TERRESTRE` o `trafficType` aún no capturado (compatibilidad con datos D1
    existentes que no tienen el campo poblado) → `ReferenceShipmentsTerrestre`
    (comportamiento por defecto, sin regresión para referencias existentes).
  - `ReferenceDetailShell.tsx` (SP-03) no requirió cambios: sigue importando
    `ReferenceShipments` desde el mismo path, con la misma firma de props.
- **`app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTerrestre.tsx`**
  (nuevo path — `git mv` del `ReferenceShipments.tsx` original, sin reescribir
  su lógica de negocio): contiene el flujo terrestre completo, sin cambios de
  comportamiento salvo:
  - Export renombrado `ReferenceShipments` → `ReferenceShipmentsTerrestre`.
  - Imports relativos corregidos por el nuevo nivel de anidamiento
    (`../modals/...` → `../../modals/...`, `../../utils/...` → `../../../utils/...`).
  - Nueva columna **"DGO"** en las tablas de Entradas y Subdivisiones: badges
    `DGO-{dgoNumber}` por cada DGO en `shipment.linkedDgos` (o "Sin DGO" si el
    array viene vacío) — esta es la trazabilidad flexible pedida por el
    sub-plan, sin imponer una regla 1 a 1. `colSpan` de las filas expandibles
    de OS&D actualizado de 9 a 10 por la columna nueva.
  - Fix incidental: entidades HTML sin escapar (`"Crear Subdivisión"` →
    `&ldquo;Crear Subdivisión&rdquo;`) que causaban el único error real
    (no warning) de `next lint` en el archivo — pre-existente en el D1
    original, corregido al ya estar interviniendo el archivo.
  - Todos los modales reusados sin cambios: `ShipmentCreationModal`,
    `SubdivisionCreationModal`, `TransferItemsModal` (los tres del sub-plan) +
    los modales legacy que ya estaban cableados (`EditInvoiceModal`,
    `BulkActionsModal`, `AssignToWarehouseDialog`, `CuentaAduaneraModal`).
- **`app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTrafficPending.tsx`**
  (nuevo): placeholder explícito para AEREO/MARITIMO, con copy diferenciado y
  referencia textual a SP-10/SP-11 — es el "punto de extensión" que dejan estos
  sub-planes futuros, en vez de forzar el flujo terrestre sobre tráficos donde
  no aplica.

## Decisiones y desviaciones

- **No se creó endpoint `GET /dgo/:id/shipments` ni `GET /shipments/:id/dgos`
  independientes.** Se decidió enriquecer el endpoint de listado ya existente
  (`GET /shipments/reference/:referenceId`) en lugar de exponer nuevos
  endpoints, porque (a) el tab solo necesita ver el vínculo desde el lado del
  movimiento, no navegar desde el DGO; (b) evita N+1 llamadas para listados de
  cientos/miles de movimientos (nota de escala del paraguas). Si un sub-plan
  futuro (p. ej. dentro de `ReferenceDGOTab`) necesita ver "qué movimientos
  cubre este DGO", se recomienda el mismo patrón de dedupe pero consultado
  desde `DgoService` — no se construyó preventivamente por estar fuera del
  alcance declarado de SP-07.
- **`trafficType` ausente (`undefined`) se trata como terrestre** en el router
  del front, no como un tráfico desconocido/bloqueante. Es la única forma de no
  romper referencias D1 existentes que aún no tengan `trafficTypeId` poblado
  (el campo es opcional en el schema). Si el spike de escala confirma que toda
  referencia nueva siempre trae `trafficTypeId`, este fallback deja de ser
  necesario pero no hace daño dejarlo.
- **Ningún archivo de `components/operations`, `customerPortal/operations` ni
  `actions/operations` fue tocado** (frontera confirmada por SP-06, respetada).
- **`components/movimientos/movimiento-modal.tsx` + `movimiento-form/**` no se
  tocaron ni se reusaron**: son el dominio de la vista raíz "Movimientos"
  (backlog logístico sin referencia aún), un flujo distinto y complementario
  (ver investigación previa a la implementación) — el sub-plan pide reusar
  expresamente `ShipmentCreationModal`/`SubdivisionCreationModal`/
  `TransferItemsModal`, que es lo que ya estaba cableado y se mantuvo.
- No se tocó **Export Terrestre** (bloqueado, fuera de alcance declarado) ni la
  creación de operación desde movimientos (ya no aplica, se crea desde DGO
  según SP-06 — el botón "Crear Operación" desde selección múltiple de
  shipments se dejó intacto porque es una funcionalidad D1 preexistente no
  mencionada como a remover en el sub-plan; se documenta por transparencia,
  no se decidió eliminarla sin pedirlo explícitamente).

## Archivos tocados

**Backend (`carmi-odin-api-v2`):**
- `src/shipments/services/shipments.service.ts` (modificado — `findByReference`)

**Frontend (`carmi-digital`):**
- `hooks/useReferenceDetail.ts` (modificado — `trafficTypeId`/`trafficType` en `ReferenceDetail`)
- `app/(customerPortal)/references/components/tabs/ReferenceShipments.tsx` (reescrito — router por tráfico)
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTerrestre.tsx` (nuevo path, `git mv` + columna DGO + fix lint)
- `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsTrafficPending.tsx` (nuevo)

## Verificación

- **Backend**: `npx tsc --noEmit` sin errores en `shipments.service.ts` (los
  errores preexistentes del proyecto están en specs no relacionados —
  `twilio.service.spec.ts`, `seal-resolver.service.spec.ts`,
  `company-patent-signature-config.service.spec.ts`, `audited.spec.ts`, y dos
  e2e-spec — todos pre-existentes, no tocados por este sub-plan). `eslint` en
  el archivo tocado: 0 errores en las líneas del diff (79 errores de
  `prettier/prettier` preexistentes en zonas del archivo no tocadas, líneas
  ~1940-2192, no introducidos por este cambio). `npx jest
  src/shipments/services/shipments.service.spec.ts
  src/shipments/controllers/shipments.controller.spec.ts` → **30/30 tests en
  verde**.
- **Frontend**: `npx tsc --noEmit -p tsconfig.json` → **0 errores en todo el
  proyecto**. `next lint` (eslint) en los 4 archivos tocados → 0 errores (solo
  warnings preexistentes de `any`/variables no usadas, heredados del archivo
  original sin modificar por este sub-plan).
- **Playwright**: **pendiente de sesión humana** (mismo bloqueo transversal de
  toda la Fase 1/2: no hay `dev_url` configurado para este cliente en
  `config/clientes.json`). Flujo a validar cuando haya entorno: en una
  referencia terrestre, crear Entrada, subdividir (no 100%), transferir ítems,
  y ver el vínculo con el/los DGO en la columna nueva — sin errores de consola.

## Bloqueos

Ninguno de infraestructura propio de este sub-plan. Hereda el bloqueo
transversal ya documentado por SP-06 (dependencias `three`/`mammoth` no
resueltas en `node_modules` para `next build` completo con Turbopack) — no
verificado de nuevo aquí porque `tsc --noEmit` y `next lint` ya dan cobertura
estática suficiente sobre los archivos del diff; no se intentó `next build`
para no reabrir ese bloqueo preexistente sin necesidad.
