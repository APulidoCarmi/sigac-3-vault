# Plan: Bandeja de Entrada — "Ligar" y "Crear referencia" en "Por identificar"

## Contexto

Pedido directo del usuario (Ángel) en sesión de trabajo del 2026-07-14,
detectado durante la verificación manual de
[[2026-07-14-bandeja-de-entrada-endpoint-real|el plan de GET /inbox]]: al
revisar la sección "Por identificar" de la Bandeja de Entrada, sus dos
botones por fila (`UnidentifiedRow` en
`carmi-digital/app/(customerPortal)/references/components/inbox/InboxDashboard.tsx`)
están rotos:

- **"Ligar"** apunta a `/inbox/identify/${id}?kind=${kind}` — ruta que **no
  existe** en el front (placeholder muerto).
- **"Crear referencia"** apunta a `/references/createReference` sin pasar
  ningún dato del movimiento/guía (`clientId`, `shipmentId`, `waybillId`).

`kind` puede ser `"shipment"` (un `Shipment` con `referenceId = null`, mismo
criterio que la pestaña "sin referencia" de `/movimientos`) o `"waybill"`
(una fila de `UnidentifiedWaybill`, poblada solo cuando
`BolReconciliationService` recibe una guía por email que no puede
emparejar automáticamente — SP-17 pieza 3).

Área de código: front `carmi-digital`
(`InboxDashboard.tsx`; `app/(customerPortal)/movimientos/components/LinkShipmentModal.tsx`,
que se **mueve y generaliza**; `references/createReference/page.tsx`, que
**ya soporta** los query params `clientId`/`shipmentId`/`waybillId` desde
antes — no se toca su lógica interna). Backend `carmi-odin-api-v2`
(`src/inbox/dtos/inbox-item.types.ts` / `inbox.service.ts`, para exponer
`clientCompanyId` en los items `shipment`; los endpoints de vinculación ya
existen y no se tocan — ver Decisiones).

## Decisiones tomadas

1. **Backend de vinculación: 100% reusado, no se crea nada nuevo.**
   Investigación previa a este plan confirmó que ya existen y funcionan:
   - `POST /shipments/:id/link-to-reference` (caso `shipment`,
     `shipments.controller.ts` + `ShipmentsService.linkShipmentToReference`).
   - `POST /email-documents/unidentified-waybills/:id/link-existing` (caso
     `waybill`, ya expuesto por `UnidentifiedWaybillsController`).
   - `POST /email-documents/unidentified-waybills/:id/create-reference`
     (marca la guía resuelta tras crear una referencia — ya invocado por
     `createReference/page.tsx` vía el query param `waybillId`, sin tocar).
   Este plan es puramente de **frontend + un campo nuevo en la respuesta
   de `GET /inbox`**.
2. **UX de "Ligar": modal in-place**, igual que ya funciona hoy en
   `/movimientos` — no se navega a otra pantalla.
3. **Restricción de negocio de `link-existing` (waybill) se deja tal
   cual**: si la referencia elegida no tiene ningún `Shipment` todavía, el
   backend lanza `BadRequestException` y el usuario ve el error y elige
   otra referencia. No se ajusta esa regla en este plan.
4. **Generalizar `LinkShipmentModal` en vez de duplicarlo** (petición
   explícita del usuario: "debería ser el mismo flujo reutilizable para no
   duplicar código"). Se **mueve** de
   `app/(customerPortal)/movimientos/components/LinkShipmentModal.tsx` a
   un lugar compartido (`components/shipment-linking/` o similar) y se
   parametriza con un modo `entityType: 'shipment' | 'waybill'`:
   - `shipment` → sigue llamando `useLinkShipmentToReference()` (sin
     cambios de comportamiento).
   - `waybill` → llama un hook nuevo (`useLinkWaybillToReference()`, no
     existe hoy) contra `POST .../unidentified-waybills/:id/link-existing`.
   Ambos casos comparten la misma UI de búsqueda/selección de referencia
   (`SearchableSelect`, lógica de cliente vs "todas las referencias").
5. **`UnidentifiedWaybill` no tiene cliente asociado** (a diferencia de
   `Shipment.clientCompanyId`) — confirmado en el schema. Para
   "Crear referencia" desde una guía: el wizard abre con `?waybillId=...`
   **sin** `clientId`, y el usuario elige el cliente manualmente (el
   wizard ya soporta ese caso; no se le agrega nada).
6. **`GET /inbox` necesita exponer `clientCompanyId`** en los items
   `kind: 'shipment'` de la sección "Por identificar" (hoy `UnidentifiedItemDto`
   no lo trae, solo un `clientName` de texto ya formateado) — es lo que el
   modal generalizado y el link "Crear referencia" necesitan para
   precargar cliente/armar la URL. Para `kind: 'waybill'` este campo va
   `null` (Decisión #5).
7. **Refresco tras éxito**: al ligar (o volver del wizard tras crear
   referencia) exitosamente, se **recarga la sección "Por identificar"
   desde el backend** (`GET /inbox?unidentifiedPage=1`, reusando la misma
   página en la que estaba el usuario) — no se hace un merge optimista en
   memoria.
8. **Refetch de "Por identificar" (confirmado tras revisión de código,
   2026-07-14): no existe hoy una función reusable que recargue solo esa
   sección.** `InboxDashboard.tsx` solo tiene `loadAll()` (recarga las 5
   secciones, declarada dentro de un `useEffect`, no exportada) y
   `handleLoadMore(sectionKey)` (solo apendea, no reemplaza). Se **extrae
   una función nueva** para recargar-y-reemplazar únicamente
   `sections.unidentified` desde `GET /inbox?unidentifiedPage=1`, en vez de
   reusar `loadAll()` (que recargaría las 5 secciones de más).
9. **Prop de identificador del modal generalizado: `entityId` genérico**
   (no `shipmentId`/`waybillId` separados). El único consumidor
   (`movimientos/page.tsx`) se actualiza para pasar `entityId` en vez de
   `shipmentId`.
10. **`fetchClientInfo` dentro del modal usa `fetch` plano en vez de
    `odinFetch`/`withAuthHeaders`** (inconsistencia ya existente en el
    código actual) — se preserva tal cual al mover/generalizar, no se
    corrige en este plan (consistente con "Fuera de alcance": rediseño del
    modal).

## Fuera de alcance

- Cambiar la regla de negocio de `linkExisting` (waybill) que exige que la
  referencia destino ya tenga un shipment.
- Inferir/prellenar el cliente de una guía sin identificar a partir de
  `extractedJson` u otra heurística — sin cliente precargado, punto.
- Cualquier cambio al wizard `createReference/page.tsx` más allá de
  construir correctamente la URL con la que se le navega — su lógica de
  `urlShipmentId`/`urlWaybillId` ya existe y no se toca.
- Rediseño visual del modal (`LinkShipmentModal`) — se mueve/generaliza tal
  cual luce hoy.
- Acciones nuevas de "descartar" (`DISMISSED`) una guía sin identificar —
  no pedidas, no se agregan.

## Pasos

- [x] Backend: agregar `clientCompanyId: string | null` a
  `UnidentifiedItemDto` (`src/inbox/dtos/inbox-item.types.ts`) y poblarlo
  en `InboxService.getUnidentified()` — `shipment.clientCompanyId` para
  items `shipment`, `null` para items `waybill`.
- [x] Front: crear `useLinkWaybillToReference()` (hook de mutación, mismo
  patrón que `useLinkShipmentToReference()` en `hooks/use-shipments.ts`)
  contra `POST /email-documents/unidentified-waybills/:id/link-existing`.
- [x] Front: mover `LinkShipmentModal.tsx` a una ubicación compartida y
  generalizarlo para aceptar `entityType: 'shipment' | 'waybill'` y un prop
  `entityId: string` genérico (reemplaza `shipmentId`), despachando al hook
  de mutación correspondiente según el modo; ajustar el único import
  existente en `movimientos/page.tsx` a la nueva ruta y al prop `entityId`
  (sin cambiar su comportamiento actual).
- [x] Front: en `InboxDashboard.tsx` (`UnidentifiedRow`/`InboxDashboard`),
  conectar el botón "Ligar" para abrir el modal generalizado con
  `entityType`/`id`/`clientCompanyId` de la fila, y corregir el href de
  "Crear referencia" para armar
  `/references/createReference?clientId=&shipmentId=` (kind shipment) o
  `/references/createReference?waybillId=` (kind waybill, sin `clientId`).
- [x] Front: tras un "Ligar" exitoso, recargar la sección "Por identificar"
  (`GET /inbox?unidentifiedPage=1`, reusando la lógica ya existente de
  fetch por sección) y cerrar el modal.
- [ ] Verificación manual end-to-end con datos reales: ligar un movimiento
  (`kind=shipment`) a una referencia existente desde el Inbox y confirmar
  que desaparece/la sección se refresca; abrir "Crear referencia" desde un
  movimiento y confirmar que el wizard llega con `shipmentId`/`clientId`
  correctos y que al terminar, el movimiento queda vinculado.

## Riesgos y side effects a vigilar

- Mover `LinkShipmentModal.tsx` de carpeta rompe su único import actual
  (`movimientos/page.tsx`) si no se actualiza en el mismo paso — revisar
  que no queden imports colgantes.
- Generalizar el modal no debe cambiar el comportamiento ya validado en
  producción de `/movimientos` para `kind=shipment` — cualquier diferencia
  de UI/mensaje entre los dos modos debe ser intencional, no un efecto
  secundario del refactor.
- `UnidentifiedItemDto.clientCompanyId` es un campo nuevo en la respuesta
  de `GET /inbox` — no debe romper al front actual que no lo esperaba (es
  aditivo, opcional).
- El caso "sin cliente" del modal (`!clientCompanyId`) ya tiene un flujo
  de búsqueda entre todas las referencias — confirmar que ese camino
  también sirve para `kind=waybill` sin cliente (que será el caso más
  común para guías).

## Criterios de verificación

- Puertas estáticas (lint/typecheck) verdes en `carmi-digital` (`/verify`).
- Playwright MCP, revisando consola del navegador:
  - Clic en "Ligar" sobre una fila `kind=shipment` abre el modal in-place
    (sin navegar), permite buscar/seleccionar una referencia y al confirmar
    llama `POST /shipments/:id/link-to-reference` (200) y la sección
    "Por identificar" se refresca sin ese movimiento.
  - Clic en "Crear referencia" sobre una fila `kind=shipment` navega al
    wizard con `?clientId=&shipmentId=` correctos; al completar el wizard,
    el movimiento queda vinculado (confirmar en `/movimientos` o
    revisitando el Inbox).
  - Sin errores nuevos de consola en ninguno de los dos flujos.
  - `/movimientos` sigue funcionando igual que antes tras mover el modal
    (regresión).
