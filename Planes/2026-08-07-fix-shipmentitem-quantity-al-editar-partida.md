# Plan: Resincronizar ShipmentItem.quantity al editar una partida de factura

## Contexto

Origen: pedido directo del usuario (sesión 2026-08-07), encontrado
verificando el plan
[[2026-08-06-fix-movimiento-guia-bultos-numero-factura]] (movimiento
automático de Guía House). El usuario editó `piezas`/`bultos` de una Guía y
luego la cantidad de una partida de su factura vinculada, y la columna
"Disponibilidad" de `ReferenceShipmentsAereo.tsx` siguió mostrando el valor
viejo.

Área de código auditada (`carmi-odin-api-v2`):

- `src/invoice-items/services/invoice-items.service.ts`:
  - `.create()` (L126-235 aprox.): al dar de alta una partida sobre una
    factura ya existente, si la factura está vinculada a una Guía con
    Shipment (`invoice.guia?.shipment?.id`), hace
    `tx.shipmentItem.upsert({ shipmentId: guiaShipmentId, invoiceItemId, quantity: dto.quantity })`
    — comentario explícito: "sin esto, ShipmentItem nunca se crea y
    Subdividir siempre ve 0 unidades disponibles".
  - `.update()` (L30-89): actualiza todos los campos editables de la
    partida, incluido `quantity`, pero **nunca** replica el upsert de
    `ShipmentItem` que sí hace `.create()`. Es el método real detrás de
    `PATCH /invoice-items/:id`, que es la vía que usa el frontend para
    editar una partida ya existente (confirmado: `AddPartidaForm.tsx:248`
    llama `callApi('/invoice-items/${existingItemId}', buildUpdateDto(), 'PATCH')`).
- `src/shipments/services/shipment-item-transaction.service.ts`:
  - `.getBalance(shipmentId, invoiceItemId)` (L53-81): para shipments
    físicos (INBOUND), `disponible = ShipmentItem.quantity - suma(salidas
    en ShipmentItemTransaction)`. Confirmado que **no hay que tocar
    `ShipmentItemTransaction`** al corregir esto — actualizar
    `ShipmentItem.quantity` es suficiente para que la disponibilidad se
    recalcule bien en la siguiente lectura.
- `src/invoices/services/invoices.service.ts` — `.update()` (L736-881): si
  el DTO trae `items`, borra TODAS las `InvoiceItem` de la factura
  (`deleteMany`) y crea partidas nuevas con IDs distintos
  (`createMany`). Como `ShipmentItem.invoiceItemId` tiene
  `onDelete: Cascade` (`schema.prisma:7576`), ese borrado elimina en
  cascada cualquier `ShipmentItem` ligado, sin recrearlo para los IDs
  nuevos — **bug real, pero el frontend actual nunca lo dispara**:
  `InvoiceFormModal.tsx:848-852` tiene un comentario explícito
  documentando por qué en modo edición nunca manda `items` a
  `PATCH /invoices/:id` (edición de partidas se hace aparte, vía
  `AddPartidaForm` → `POST`/`PATCH /invoice-items`).

## Decisiones tomadas

1. **Alcance: solo `InvoiceItemsService.update()`.** Es la única vía real
   por la que el frontend edita una partida existente. El bug de cascada en
   `InvoicesService.update()` (líneas 798-838) es real pero no se toca en
   este plan — el usuario decidió no endurecerlo ahora porque nada en el
   frontend actual lo dispara; queda anotado como riesgo/deuda conocida
   para revisar en el futuro si algún flujo llega a mandar `items[]` en modo
   edición.
   - Porqué: acotar el cambio al camino real que causó el bug reportado,
     evitando tocar una lógica más compleja (reconciliar
     `ShipmentItem`/`ShipmentItemTransaction`/`OsdLog` al reemplazar TODAS
     las partidas) sin necesidad inmediata.
2. **Sin backfill**: los `ShipmentItem.quantity` ya desincronizados en
   producción (como el caso de prueba del usuario: `quantity: 14` con la
   partida real ya en 16) no se corrigen con este plan. Mismo criterio que
   [[2026-08-06-fix-movimiento-guia-bultos-numero-factura]] aplicó para
   `bultos`/`shipmentCode`: el fix aplica solo hacia adelante, a partir de
   la próxima edición de cada partida.
   - Porqué: decisión explícita del usuario, consistente con el precedente
     inmediato de la sesión anterior.
3. **Mismo patrón que `.create()`**: el fix en `.update()` replica
   exactamente el `upsert` de `ShipmentItem` que ya existe en `.create()`
   (mismo `where` por `shipment_item_unique_constraint`, mismo guard
   `if (guiaShipmentId)`), en vez de inventar una lógica nueva. Solo corre
   cuando `dto.quantity !== undefined` (si no viene en el DTO, no hay nada
   que resincronizar).
   - Porqué: minimiza riesgo — es un patrón ya probado en producción, no
     código nuevo sin precedente.

## Fuera de alcance

- Endurecer `InvoicesService.update()` (reemplazo completo de `items[]`,
  líneas 798-838) contra el bug de cascada (`onDelete: Cascade` en
  `ShipmentItem.invoiceItemId`) — ver Riesgos.
- Backfill de `ShipmentItem.quantity` para partidas ya editadas antes de
  este fix.
- Sincronizar `ShipmentItem` para partidas de facturas **no** vinculadas a
  una Guía (flujo terrestre/manual, donde `ShipmentItem.quantity` puede
  representar una asignación parcial elegida por el usuario al crear el
  Shipment, no necesariamente igual a `InvoiceItem.quantity` completo) — el
  guard `if (guiaShipmentId)` ya excluye ese caso, igual que `.create()`.
- Tocar `ShipmentItemTransaction` o el cálculo de `getBalance()` — no hace
  falta (ver Contexto).

## Pasos

- [x] **Backend — `InvoiceItemsService.update()`** (`invoice-items.service.ts:30-89`):
      después de `tx.invoiceItem.update(...)` (o el equivalente actual sin
      `tx`, revisar si el método ya corre dentro de una transacción o hay
      que envolverlo), si `dto.quantity !== undefined`, resolver
      `invoice.guia?.shipment?.id` (requiere incluir `guia.shipment.id` en
      el `include`/query de la factura, igual que hace `.create()` vía
      `invoice.guia.shipment.id` en su lookup inicial) y hacer:
      ```
      tx.shipmentItem.upsert({
        where: { shipment_item_unique_constraint: { shipmentId: guiaShipmentId, invoiceItemId: id } },
        create: { shipmentId: guiaShipmentId, invoiceItemId: id, quantity: dto.quantity },
        update: { quantity: dto.quantity },
      })
      ```
      solo si `guiaShipmentId` existe (mismo guard que `.create()`).
- [x] **Backend — specs**: agregar casos en
      `invoice-items.service.spec.ts` (o crearlo si no existe) cubriendo:
      editar `quantity` de una partida de una factura vinculada a Guía con
      Shipment → se llama `shipmentItem.upsert` con la nueva `quantity`;
      editar `quantity` de una partida de factura **sin** Guía/Shipment →
      no se llama `shipmentItem.upsert`; editar un campo que no sea
      `quantity` → no se llama `shipmentItem.upsert`.
- [ ] **Verificación end-to-end manual**: repetir el caso real del usuario
      — guía con factura ya vinculada a un movimiento, editar la cantidad
      de una partida vía el formulario de facturas
      (`AddPartidaForm`/`GuiaInvoicesModal`), confirmar en BD que
      `ShipmentItem.quantity` cambió, y que "Disponibilidad" en
      `ReferenceShipmentsAereo.tsx` refleja el nuevo valor tras refrescar.

## Riesgos y side effects a vigilar

- **Deuda conocida, no corregida aquí**: `InvoicesService.update()`
  (líneas 798-838) sigue teniendo el bug de cascada
  (`deleteMany`+`createMany` de `InvoiceItem` → cascade delete de
  `ShipmentItem` vía `onDelete: Cascade` en `schema.prisma:7576`, sin
  recrear el `ShipmentItem` para los IDs nuevos). Hoy no se dispara porque
  `InvoiceFormModal.tsx` evita mandar `items` en modo edición a propósito
  — pero cualquier otro caller futuro de `PATCH /invoices/:id` con `items`
  poblado (o un cambio en el frontend que reintroduzca ese envío) volvería
  a romper la disponibilidad de todas las partidas de esa factura de un
  jazo, y además perdería el historial de `ShipmentItemTransaction`/`OsdLog`
  ligado a los `InvoiceItem` borrados. Si se decide atacarlo después,
  necesita su propio plan (reconciliar por `lineNumber`/`partNumber` en vez
  de borrar y recrear).
- El `upsert` nuevo en `.update()` corre dentro de la misma transacción que
  ya usa el método (si `.update()` no está en un `$transaction` hoy, hay
  que decidir si envolverlo — verificar antes de implementar; si no está
  en `tx`, usar `this.prisma` directo, igual que otras escrituras sueltas
  del mismo archivo, no forzar una transacción nueva solo por este fix).
- El guard `if (guiaShipmentId)` significa que partidas de facturas
  terrestres/manuales (sin Guía) no se tocan — comportamiento intencional
  (ver Fuera de alcance), pero vale dejarlo explícito en el código con un
  comentario para que no se lea como un descuido.

## Criterios de verificación

- Editar la `quantity` de una partida de una factura vinculada a una Guía
  con Shipment ya generado → `ShipmentItem.quantity` en BD refleja el nuevo
  valor.
- La columna "Disponibilidad" en el tab Guías de una Referencia
  (`ReferenceShipmentsAereo.tsx`) muestra el nuevo total tras refrescar
  (`fetchBalances()`).
- Editar un campo de la partida que no sea `quantity` no dispara ningún
  `shipmentItem.upsert` de más.
- Editar una partida de una factura sin Guía/Shipment no falla ni crea un
  `ShipmentItem` inesperado.
- Suite de backend (`invoice-items.service.spec.ts`) en verde; `/verify`
  (lint + typecheck + tests) antes de cerrar.
