# Sub-plan SP-12: Tab Operaciones — agrupador vía DGO

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #11 del [[Inventario_Pantallas_v3]] (🟡 ajustar consulta). La Función 3 de
la Referencia: ver las operaciones que agrupan sus DGO/pedimentos. La Operación
**agrupa, no crea**; puede agrupar varias referencias, y una referencia puede tener
DGO/pedimentos repartidos en varias operaciones. Origen: glosario "Operación" del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]] y
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b).

## D1 — punto de partida
- **Reusa:** `references/components/tabs/ReferenceOperations.tsx` (tabla folio /
  status / pedimento / valor; acciones Inicio de Operación / Editar / Proforma /
  Fondos). API `GET /references/:id/operations`.
- **Refactoriza:** la **consulta** debe traer **todas las operaciones vinculadas vía
  DGO** a esta referencia, no solo las nacidas exclusivamente de ella.
- **Crea:** nada estructural (ajuste de fuente de datos/consulta).

## Fuera de alcance
- Crear/editar operación (SP-06), Solicitar Fondos (SP-14), Proforma (SP-15) —
  aunque sus botones viven en este tab.

## Pasos
- [x] Ajustar la consulta a "operaciones vinculadas vía DGO".
- [x] Verificar que la tabla refleja operaciones consolidadas (multi-referencia).

## Riesgos y side effects
- Depende del modelo DGO (SP-05). Coordinar con Carlos (listado de operaciones, M8).

## Criterios de verificación
- Gate estático verde. Playwright: una referencia cuyos DGO están repartidos en 2
  operaciones muestra ambas; sin errores de consola.

## Estado
✅ Implementado en `refactor/customs-operation-sp12` (back: `carmi-odin-api-v2`). No se
tocó el front: `ReferenceOperations.tsx` ya soportaba multi-referencia (SPEC-007) y no
requería cambios estructurales, solo la fuente de datos (backend). Ver manifiesto:
[[.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]].

**Nota 2026-07-28:** [[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]]
tocó `ReferenceOperations.tsx` (el mismo archivo que este sub-plan reusa) para eliminar
el item "Ver Proforma" del dropdown de acciones — la vista de proforma a nivel-operación
completa se elimina (Decisión 8 de ese plan; la proforma se reubica a nivel DGO). No
afecta la consulta/agrupación vía DGO que este sub-plan implementó (sigue vigente), solo
retira un botón que ya estaba explícitamente fuera de alcance aquí (línea 23: "Proforma
(SP-15) ... aunque sus botones viven en este tab").
