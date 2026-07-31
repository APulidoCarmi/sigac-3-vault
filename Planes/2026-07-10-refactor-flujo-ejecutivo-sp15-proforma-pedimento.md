# Sub-plan SP-15: Ver Proforma / Pedimento (Anexo 22) + limpieza de código muerto

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #24 del [[Inventario_Pantallas_v3]] (🟢 ya existe). Revisar el pedimento
(resumen + documento Anexo 22). Origen: D1 y glosario "Pedimento" del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]]. Sub-plan de bajo riesgo: sobre todo
**reuso + limpieza**.

## D1 — punto de partida
- **Reusa:** `references/components/drawers/OperationProformaDrawer.tsx` →
  `drawers/pedimento/PedimentoHtmlViewer.tsx`, `ProformaHeader.tsx`, `ProformaPartidas.tsx`,
  `ProformaLiquidation.tsx`, `ProformaAnnexes.tsx`. API `GET /operations/:id/proforma`
  (`operations.controller.ts:130`).
- **Refactoriza:** mínimo — alinear los datos mostrados con el DGO (SP-05) y los
  renombres (incrementables/decrementables, identificadores).
- **Limpia (código muerto confirmado):** `drawers/pedimento/PedimentoProformaDocument.tsx`,
  `drawers/pedimento/PedimentoHbsDocument.tsx` (sin importadores) y el tab muerto
  `references/components/tabs/ReferencePedimento.tsx` (no montado).

## Fuera de alcance
- Generación del pedimento (vive en la Operación/DGO).

## Pasos
- [x] Verificar que el viewer refleja los datos del DGO y renombres.
- [x] Eliminar `PedimentoProformaDocument.tsx`, `PedimentoHbsDocument.tsx`,
      `ReferencePedimento.tsx`.

## Riesgos y side effects
- Confirmar con graphify que los 3 archivos siguen sin importadores antes de borrar.

## Criterios de verificación
- Gate estático verde (sin referencias rotas tras el borrado). Playwright: abrir la
  proforma/pedimento de una operación y ver el Anexo 22; sin errores de consola.

## Estado
✅ Implementado. Ver manifiesto:
[[.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp15-proforma-pedimento]].

**Actualizado 2026-07-28** por
[[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]] (Decisión 8/9): la
proforma deja de vivir solo a nivel Operación (`GET /operations/:id/proforma`) — se
extiende a nivel DGO individual, para verse en vivo mientras se captura el DGO, antes de
firmar/crear la operación. Reusa el mismo `OperationProformaDrawer.tsx` que este sub-plan
dejó como base (no se crea uno nuevo, corrigiendo una decisión intermedia del plan de
2026-07-28 que inicialmente proponía duplicarlo). Cambios concretos: se eliminó la
invocación a nivel-operación completa en `ReferenceOperations.tsx` (tarea 8 de ese plan,
completada); pendiente en ese mismo plan (tareas 12-14, backend + frontend, aún no
implementadas al cierre de esta nota) adaptar `OperationProformaDrawer.tsx` para aceptar
`dgoId` (además de `operationId`) e integrarlo en `ReferenceDGOTab.tsx`/`DgoActionsDrawer`.
El armado de `ProformaData` se extrae a un método de servicio reusable en
`carmi-odin-api-v2`, parametrizado por datos crudos en vez de por `operationId` — la
limpieza de código muerto de este sub-plan (D1, "Limpia") no se revierte.
