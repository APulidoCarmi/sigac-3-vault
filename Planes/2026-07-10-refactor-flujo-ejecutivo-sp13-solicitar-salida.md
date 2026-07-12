# Sub-plan SP-13: Solicitar Salida (mini-modal)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #22 del [[Inventario_Pantallas_v3]] (🔵 nueva). Avisar a almacén que algo va
a salir, desde dentro de la Operación. Origen:
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b): "cuando yo
tenga la operación... 'solicitar salida' y ahí generar el movimiento... un minimodal
con esos campos bien poquitos" — la mayoría de los datos se heredan del movimiento de
entrada; solo se capturan **sello, contenedor y tipo de contenedor**.

## D1 — punto de partida
- **Reusa:** el flujo **OUTBOUND** de `components/shipment-creation/ShipmentCreationModal.tsx`
  (`OutboundForm.tsx`, `PedimentoSelectorForExit.tsx`) y el `outbound.controller.ts` de
  odin — la salida ya existe en D1; aquí se reduce a un mini-modal.
- **Refactoriza:** convertir el flujo OUTBOUND extenso en un **mini-modal** disparado
  **desde la Operación**, con solo sello/contenedor/tipo y el resto heredado
  automáticamente de la entrada.
- **Crea:** el mini-modal Solicitar Salida.

## Fuera de alcance
- La creación de la operación (SP-06).

## Pasos
- [x] Añadir la acción "Solicitar Salida" dentro de la Operación.
- [x] Mini-modal con sello/contenedor/tipo; heredar el resto de la entrada.
- [x] Generar el movimiento de salida (regla: salida requiere entrada).

## Riesgos y side effects
- No duplicar el flujo OUTBOUND completo; reusar sus datos heredados.
- **Desviación documentada:** `OutboundForm.tsx` y `PedimentoSelectorForExit.tsx`
  resultaron ser componentes huérfanos (no conectados a `ShipmentCreationModal.tsx`
  ni a ningún flujo vigente). El flujo OUTBOUND real vive en el formulario
  unificado (`NewReferenceShipmentsForm`) que llama a `POST/PATCH /shipments`
  (`flowType: 'OUTBOUND'`, `containers[]`). Por eso el mini-modal no reusa esos
  dos componentes: en vez de eso, se creó un endpoint dedicado
  `POST /operations/:id/request-exit` (odin) que arma el `CreateShipmentDto`
  heredando los campos del INBOUND (vía `OperationsService.requestExit`,
  inyectando `ShipmentsService`) y delega en el mismo `ShipmentsService.create`
  que usa el flujo normal — así no se duplica la validación
  "salida requiere entrada" (`validateInventoryAvailability`), que ya vive ahí.

## Criterios de verificación
- Gate estático verde (backend: typecheck + lint + 523 test suites / 3447 tests
  OK, incluye 3 tests nuevos de `requestExit`; front: typecheck limpio, eslint
  limpio en archivos tocados — `next lint` está roto repo-wide en esta versión
  de Next, independiente de este cambio).
- Playwright: **pendiente** — los servidores dev de front/back ya corrían en
  el entorno como builds de producción (`next start` / `dist/`) de otra sesión;
  no se reiniciaron para no interferir con otro trabajo en curso. Queda
  pendiente validar manualmente el flujo end-to-end (abrir una operación con
  INBOUND, "Solicitar Salida", capturar sello/contenedor/tipo, verificar el
  movimiento OUTBOUND creado y ausencia de errores de consola).

## Estado
✅ Implementado (código completo, gates estáticos verdes). Playwright pendiente
de ejecución manual (ver criterios de verificación).
