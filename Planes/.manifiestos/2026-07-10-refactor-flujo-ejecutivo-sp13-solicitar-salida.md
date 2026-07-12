# Manifiesto — SP-13: Solicitar Salida (mini-modal)

Implementado 2026-07-11. Ver sub-plan
[[2026-07-10-refactor-flujo-ejecutivo-sp13-solicitar-salida]].

## Ramas
- `carmi-odin-api-v2`: `refactor/customs-operation-sp13` (encadenada desde
  `refactor/customs-operation-sp11`). Nada comiteado.
- `carmi-digital`: `refactor/customs-operation-sp13` (encadenada desde
  `refactor/customs-operation-sp11`). Nada comiteado.

## Brecha entre el D1 del sub-plan y el estado real del repo (importante)
El D1 decía: reusar `OutboundForm.tsx` y `PedimentoSelectorForExit.tsx` (y
`outbound.controller.ts`) como base del mini-modal. Investigación previa a
escribir código encontró que **ambos componentes están huérfanos**: no los
importa `ShipmentCreationModal.tsx` ni ningún flujo vigente (parecen residuo
de una fase anterior). El flujo OUTBOUND real vive en el formulario unificado
`NewReferenceShipmentsForm` (mismo componente que INBOUND, diferenciado por
`flowType`/`regimenType`/`warehouseRouting`), que llama a `POST/PATCH
/shipments` (`CreateShipmentDto`, `containers: [{containerTypeId,
containerNumber, sealNumber}]`). `outbound.controller.ts` tampoco es el
endpoint de creación: expone `POST /outbound/commit-exit` (confirmar salida
física / cierre de inventario) y `GET /outbound/pending-exit/:pedimentoId`,
no la creación del movimiento.

**Decisión tomada:** no se reusaron `OutboundForm.tsx`/`PedimentoSelectorForExit.tsx`
(hubiera significado enganchar código huérfano en vez de reusar el flujo
vigente). En su lugar se creó un endpoint dedicado
`POST /operations/:id/request-exit` (odin) que arma el `CreateShipmentDto`
heredando los campos del INBOUND server-side y delega en
`ShipmentsService.create` — el mismo método que usa el flujo OUTBOUND normal,
así la regla "salida requiere entrada" (`validateInventoryAvailability`,
`shipments.service.ts`) no se duplica, solo se reafirma a nivel de operación
(existencia de un INBOUND asociado) antes de invocar el `create` común.

## Backend (`carmi-odin-api-v2`)
- `src/operations/dtos/request-exit.dto.ts` (nuevo): `RequestExitDto` —
  `sealNumber`, `containerNumber`, `containerTypeId` (los 3 campos que captura
  el mini-modal) + `userId?` opcional.
- `src/operations/services/operations.service.ts`:
  - Import de `ShipmentsService` y `CreateShipmentDto`; inyectado en el
    constructor (`shipmentsService`).
  - Método nuevo `requestExit(operationId, dto)`: busca la operación con un
    `select` acotado de `shipments[].shipment` (`flowType`, `referenceId`,
    `clientCompanyId`, `transportModeId`, `warehouseLocationId`, `carrierId`,
    `customsBrokerId`); filtra el INBOUND; si no existe lanza
    `BadRequestException` ("la operación no tiene un movimiento de entrada
    asociado" — regla "salida requiere entrada" a nivel de operación); arma
    el `CreateShipmentDto` (`flowType: OUTBOUND`, `warehouseRouting: 'EXIT'`,
    `operationId`, campos heredados del INBOUND, `containers: [{...}]` con
    los 3 campos capturados) y delega en `this.shipmentsService.create(...)`.
  - No se tocó el `select` de `findOne` (usado ampliamente por el front para
    el detalle de operación) — `requestExit` hace su propia consulta acotada
    para no arriesgar ese endpoint ya en uso.
- `src/operations/controllers/operations.controller.ts`: endpoint nuevo
  `POST :id/request-exit` (mismo patrón sin guard que el resto del
  controller, ej. `request-pedimento`/`solicitar-fondos`; no usa
  `JwtAuthGuard` porque ningún otro endpoint de este controller lo usa).
- `src/operations/operations.module.ts`: import de `ShipmentsModule` (sin
  ciclo — `ShipmentsModule` no importa `OperationsModule`).
- `src/operations/services/operations.service.spec.ts`: mock de
  `ShipmentsService` agregado a la suite existente (rompía por el nuevo
  parámetro del constructor) + 3 tests nuevos para `requestExit` (404 sin
  operación, 400 sin INBOUND, creación con herencia correcta de campos).

## Front (`carmi-digital`)
- `lib/api/modules/customs-operation.ts`: `RequestExitDto` (interface) +
  `customsOperationServices.requestExit(operationId, dto)` → `POST
  /operations/:id/request-exit`.
- `app/(customerPortal)/customs-operation/components/SolicitarSalidaModal.tsx`
  (nuevo): mini-modal `Dialog` siguiendo el patrón de
  `RequestPrevioModal.tsx` (referencias/modals) — 3 campos: `containerTypeId`
  (`SearchableSelect` + `useContainerTypes()`, reusado tal cual del step de
  contenedores del formulario unificado), `containerNumber` y `sealNumber`
  (`Input`). `useMutation` + `useToast` (no `sonner`, para igualar el patrón
  de `RequestPrevioModal`). Usuario actual vía `useUser()` →
  `user.user.UserID` como `userId`.
- `app/(customerPortal)/customs-operation/components/OperationHeader.tsx`:
  props nuevas `onRequestExit?`/`canRequestExit?`; botón "Solicitar Salida"
  (ícono `LogOut`, `variant="outline"`) junto a "Iniciar Operación"/"Cancelar
  Despacho".
- `app/(customerPortal)/customs-operation/[id]/page.tsx`:
  - Interface local `Operation` extendida con `shipments?:
    Array<{shipment?:{id,flowType}}>` (el backend ya lo incluye en `GET
    /operations/:id`, no se necesitó tocar el service).
  - Estado `isRequestExitOpen`; `hasInboundShipment` calculado filtrando
    `operation.shipments` por `flowType === 'INBOUND'` — controla
    `canRequestExit` (el botón solo aparece si hay un INBOUND del cual
    heredar).
  - Render de `<SolicitarSalidaModal open onOpenChange operationId
    onSuccess={fetchOperation} />` junto a `<OperationHeader />`.

## Verificación
- **Backend**: `npx tsc --noEmit`: 0 errores nuevos (mismos preexistentes en
  archivos `.spec.ts`/`.e2e-spec.ts` no relacionados, ya documentados en
  manifiestos previos de la cadena: `twilio.service.spec.ts`,
  `seal-resolver.service.spec.ts`,
  `company-patent-signature-config.service.spec.ts`, `audited.spec.ts`, 3
  `*.e2e-spec.ts`). `npx eslint` sobre los 4 archivos tocados/nuevos: 0
  errores (2 warnings preexistentes sin relación en
  `operations.service.ts`). `npx jest` (suite completa): **523 test suites,
  3447 tests, todos verdes** (incluye los 3 nuevos de `requestExit`, sin
  regresión).
- **Front**: `npx tsc --noEmit` sobre todo el repo: 0 errores. `npx eslint`
  sobre los 4 archivos tocados/nuevos: 0 errores (solo warnings preexistentes
  de `any` en código no tocado por esta feature). `next lint` (script `npm
  run lint`) está roto repo-wide en esta versión de Next (falla con "Invalid
  project directory" incluso sin cambios) — no es una regresión de este
  sub-plan, reportado como hallazgo aparte.
- **Playwright**: no ejecutado. Al iniciar la verificación, los puertos 3000
  (`next-server`, build de producción) y 8090 (`node dist/src/main`) ya
  estaban ocupados por procesos de otra sesión/tarea en curso; reiniciarlos
  para levantar el código nuevo (el backend corre desde `dist/`, no en modo
  watch) hubiera interrumpido ese trabajo ajeno. Pendiente de sesión humana:
  abrir una operación con un movimiento INBOUND asociado, click "Solicitar
  Salida", capturar sello/contenedor/tipo, confirmar el movimiento OUTBOUND
  creado y ausencia de errores de consola (criterio de verificación del
  sub-plan).

## Desviaciones / decisiones fuera de lo literal del sub-plan
1. No se reusaron `OutboundForm.tsx`/`PedimentoSelectorForExit.tsx` (D1
   literal) por estar huérfanos — se documentó la brecha arriba y se optó por
   un endpoint backend dedicado que reusa `ShipmentsService.create` (el mismo
   método que sí usa el flujo OUTBOUND vigente), evitando duplicar la validación
   de inventario.
2. La herencia de datos del INBOUND se hizo **server-side**
   (`OperationsService.requestExit`) y no en el front, para no duplicar en el
   cliente el conocimiento de qué campos hereda un OUTBOUND — mantiene la
   lógica de negocio en un solo lugar (backend) y el mini-modal del front
   queda mínimo (solo los 3 campos capturados), como pide el sub-plan.
3. Se agregaron 3 tests unitarios nuevos para `requestExit` (no exigidos
   explícitamente por el sub-plan, pero Playwright no era viable en este
   entorno y la lógica nueva incluye una regla de negocio — "salida requiere
   entrada").
4. No se tocó `findOne` de `OperationsService` (usado por el detalle de
   operación) para no arriesgar ese endpoint ya en producción — `requestExit`
   hace su propio `select` acotado.
