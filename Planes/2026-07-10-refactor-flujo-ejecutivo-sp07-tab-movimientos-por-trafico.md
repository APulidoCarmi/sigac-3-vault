# Sub-plan SP-07: Tab Movimientos — rediseño por tráfico + vínculo flexible con DGO

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #8 del [[Inventario_Pantallas_v3]] (🟡 rediseño de fondo). El tab
Movimientos vive en el plano **físico-logístico** (almacén), distinto del
documental-legal (DGO). Origen: [[2026-03-19 - Análisis de Referencia y Movimientos]]
(M0: la entrada avisa a almacén qué llega y cuándo),
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b) y glosario
"Movimiento" del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]] (taxonomía por
tráfico; vínculo Movimiento↔DGO **flexible, no 1 a 1**). Cubre el reuso de #14
(Crear Entrada), #15 (Subdivisión) y #17 (Transferir), que ya existen (🟢).

## D1 — punto de partida
- **Reusa:** `references/components/tabs/ReferenceShipments.tsx` (tab con sub-tabs
  Entradas INBOUND / Subdivisiones); `components/shipment-creation/ShipmentCreationModal.tsx`
  (#14 Crear Entrada, INBOUND/OUTBOUND; `InboundForm`, `OutboundForm`, `FlowTypeSelector`),
  `SubdivisionCreationModal.tsx`/`SubdivisionForm.tsx` (#15; regla: no subdividir el
  100%), `TransferItemsModal.tsx` (#17); `components/movimientos/movimiento-modal.tsx`
  + `movimiento-form/**`. APIs `GET /references/:id/shipments`,
  `GET /references/:id/shipments/available-for-operation` (`references.controller.ts:342`),
  `GET /shipments/:id/balance`, `POST /shipments/subdivision`, `POST /shipments/:id/transfer`.
- **Refactoriza:** D1 trata el tab **igual para todos los tráficos** y **asume
  vínculo fijo** con la operación. Cambiar a: (a) el tab **adapta qué muestra según
  el tráfico** de la referencia (terrestre = Entrada/Salida; aéreo/marítimo remiten
  a SP-10/SP-11); (b) **dejar ver el vínculo flexible DGO↔movimiento** (un DGO puede
  cubrir varios movimientos o viceversa), sin asumirlo 1 a 1.
- **Crea:** contenedor por tráfico + visualización del vínculo flexible.

## Fuera de alcance
- Los movimientos especializados de aéreo (SP-10) y marítimo (SP-11).
- **Export Terrestre** (bloqueado, falta sesión de discovery).
- La creación de la operación desde movimientos (ya no aplica; se crea desde DGO, SP-06).

## Pasos
- [ ] Adaptar el tab para renderizar el set de movimientos correcto según tráfico.
- [ ] Reusar Entrada/Subdivisión/Transferir para terrestre importación (ya existen).
- [ ] Mostrar el vínculo flexible DGO↔movimiento (trazabilidad, no regla fija).

## Riesgos y side effects
- Depende del modelo de vínculo DGO↔movimiento definido en SP-05.

## Criterios de verificación
- Gate estático verde. Playwright: en una referencia terrestre, crear Entrada,
  subdividir (no 100%), transferir ítems, y ver el vínculo con el/los DGO; sin
  errores de consola.

## Estado
📋 Por implementar.
