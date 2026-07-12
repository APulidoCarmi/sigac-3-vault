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
