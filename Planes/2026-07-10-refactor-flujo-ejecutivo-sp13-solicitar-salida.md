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
- [ ] Añadir la acción "Solicitar Salida" dentro de la Operación.
- [ ] Mini-modal con sello/contenedor/tipo; heredar el resto de la entrada.
- [ ] Generar el movimiento de salida (regla: salida requiere entrada).

## Riesgos y side effects
- No duplicar el flujo OUTBOUND completo; reusar sus datos heredados.

## Criterios de verificación
- Gate estático verde. Playwright: desde una operación, Solicitar Salida capturando
  solo 3 campos y verificar el movimiento de salida creado; sin errores de consola.

## Estado
📋 Por implementar.
