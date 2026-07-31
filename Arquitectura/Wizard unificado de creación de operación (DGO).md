# Wizard unificado de creación de operación (DGO)

Estado vigente desde 2026-07-28 ([[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]]).
Reemplaza el estado descrito en [[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]]
(que asumía dos wizards paralelos, uno de ellos fuera de alcance).

## Wizard único

`components/operations/CreateOperationModal.tsx` (`carmi-digital`) es el **único** wizard
de creación/edición de operación. El wizard de página `customs-operation/createOperation/`
(4 pasos, basado en shipments/invoices) se eliminó por completo.

**3 pasos, un solo pase (no hay loop por pedimento):**
0. `StepDgoSelection.tsx` — seleccionar uno o más DGO(s) firmados (de una o varias
   referencias, siempre que compartan régimen aduanero homogéneo). 1 DGO seleccionado = 1
   pedimento (`initializeGroupsFromDgos` en `create-operation.store.ts`).
1. `Step2OperationConfig.tsx` — datos de **operación completa**: TC (manual o Banxico del
   día), fechas, tipo de operación, agente aduanal MOMP, empresa de facturación,
   mandatario, y **aduana de despacho** (agregado en este plan — antes vivía,
   incorrectamente, como dato per-pedimento).
2. `Step4PaymentForms.tsx` — botón "Calcular impuestos" (`POST /operations/preview-taxes`)
   + matriz de formas de pago.

## Fuente de verdad de los datos de pedimento

Régimen, clave de pedimento, destino y observaciones **no se capturan en el wizard** — se
leen de solo lectura desde cada DGO (`DgoPedimentoForm` en `ReferenceDGOTab.tsx`, ya
capturados antes de llegar al wizard). **Aduana y patente** son la excepción: son datos de
**operación completa** (una sola vez, no por DGO/pedimento), no per-DGO como se asumía
originalmente en SP-06/SP-19.

## Entry points activos hacia `CreateOperationModal`

- `ReferenceHeroHeader.tsx` / `ShipmentCard.tsx` — abren el modal en Step 0 (elegir DGOs),
  ya no asumen un shipment directo.
- `ReferenceClassicTable.tsx` / `ReferenceShipmentsTerrestre.tsx` — abren el modal
  (revertido desde SP-19, que los había migrado a navegar a la página ahora eliminada).
- `ReferenceDGOTab.tsx` — control global de selección múltiple de DGOs firmados, fuera del
  Accordion, con botón "Crear Operación".
- `ReferenceInventoryTable.tsx` / `OperationClient.tsx` (lista de operaciones) — hallados
  en auditoría de código durante este plan (no estaban en la lista de entry points
  auditados originalmente); migrados también a abrir el modal.

## Proforma por DGO

`OperationProformaDrawer.tsx` acepta `dgoId` (además de `operationId`) — se ve en vivo
desde `ReferenceDGOTab.tsx`/`DgoActionsDrawer` (botón "Ver Proforma"), antes de firmar/
crear la operación. Backend: `GET /dgo/:id/proforma` arma la proforma desde el propio DGO
si no está `locked`, o delega a `GET /operations/:id/proforma` si ya originó un pedimento
(`OperationsService.getProformaFromDgo`/`assembleProformaResponse`). `preview-taxes` acepta
un filtro `dgoId` opcional para agregar solo las invoices de ese DGO.

## Gap conocido: no se puede crear una operación solo-desde-DGO todavía

`POST /operations` (`operations.service.ts` `create()`) exige `shipments` o
`movementInvoices` no vacíos — restricción de esquema
(`OperationInvoice.operationShipmentId` es `NOT NULL`), no solo un guard de código. El
wizard ya no captura shipments (Step 0 es selección de DGOs), así que **el submit final
está bloqueado a propósito** (toast explícito en `CreateOperationModal.handleSubmit`)
hasta que se implemente un sub-plan de backend aparte (migración de Prisma +
`createFromDgos()`). Ver "Pendientes" en la sesión
[[2026-07-28 - Unificar wizard de operación en CreateOperationModal (DGO como origen)]].
