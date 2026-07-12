# Sub-plan SP-06: Wizard de Crear/Editar Operación → Step 0 "Seleccionar DGO(s)"

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #21 del [[Inventario_Pantallas_v3]] (🟡 cambiar Step 0). La Operación
**agrupa** uno o más DGO/pedimentos — no los crea ni transforma. Origen:
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b: la operación
nace de entrada/subdivisión, refinado luego a "desde el DGO"), glosario
"Operación"/"DGO" del [[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]].

**Decisión de la entrevista (aclara una ambigüedad del Documento):** el refactor
opera sobre el wizard **IN-SCOPE de 4 pasos** `customs-operation/createOperation/page.tsx`,
NO sobre el de 6 pasos `components/operations/CreateOperationModal.tsx` (ese está en
`components/operations` = fuera de alcance; el Documento lo describía por error).

## D1 — punto de partida
- **Reusa:** `customs-operation/createOperation/page.tsx:18-40` (4 pasos: Referencias
  → Datos Básicos → Transporte → Despacho); store `stores/create-operation.store.ts`;
  cuerpos `createOperation/components/StepInventorySelection.tsx`, `stepOperationBasicData.tsx`
  (`GET /operations/prepare`), `stepTransportConfig.tsx`, `stepCustomsInfo.tsx`,
  `StepIncrementables.tsx`, `StepPartidasTable.tsx`, `StepPedimentoHeader.tsx`; context
  `createOperation/context/PedimentoWizardContext.tsx`. APIs `GET /operations/prepare`,
  `POST /operations` (`lib/api/modules/customs-operation.ts:193`), `PATCH /operations/:id`.
- **Refactoriza:**
  - **Step 0** (`StepInventorySelection`, hoy selección de inventario/movimientos)
    → **"Seleccionar DGO(s)"**: la operación se arma eligiendo DGOs; el número de DGOs
    determina el número de pedimentos.
  - **Reactivar el cálculo de impuestos** hoy comentado en
    `PedimentoWizardContext.tsx:191` (`/api/pedimentos/calculate-reserve`).
  - Como el DGO ya trae los campos de pedimento, los pasos de config/encabezado
    llegan **prellenados** (confirmar/ajustar, no capturar desde cero).
- **Crea:** nada estructural; ajustar fuente de datos (de movimientos → DGOs).

## Fuera de alcance
- El wizard de 6 pasos `components/operations/**` (fuera de alcance).
- El vínculo Movimiento↔DGO como mecanismo de creación (ya no aplica; es solo dato
  de trazabilidad en el tab Movimientos, SP-07).

## Pasos
- [ ] Cambiar Step 0 a "Seleccionar DGO(s)" y ajustar el store/context a DGOs.
- [ ] Prellenar config fiscal / encabezado desde el DGO seleccionado.
- [ ] Reactivar el cálculo de impuestos comentado y validarlo.
- [ ] Soportar multi-DGO → multi-pedimento bajo una operación.

## Riesgos y side effects
- **No tocar `components/operations`** (homónimo fuera de alcance).
- Depende de SP-05 (DGO) — no arrancar antes de tener el modelo DGO.

## Criterios de verificación
- Gate estático verde. Playwright: crear una operación seleccionando 2 DGOs → 2
  pedimentos, config prellenada del DGO, impuestos calculados; sin errores de consola.

## Estado
🚧 Bloqueado — el D1 describe archivos/flujo que no corresponden al wizard real
(`page.tsx` Step 0 es un placeholder sin implementar, no `StepInventorySelection`;
`PedimentoWizardContext.tsx` es código huérfano nunca montado; no hay "cálculo de
impuestos comentado" que reactivar — el TaxEngine real vive solo en el módulo fuera de
alcance `components/operations/**`). Detalle completo, hallazgos y recomendación de
qué resolver en una nueva ronda de `/plan` en el manifiesto:
[[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]] (ver
`Planes/.manifiestos/`). Ningún paso `- [ ]` se marcó como hecho; no se escribió código
de producto. Rama `refactor/customs-operation-sp06` creada en `carmi-digital` (vacía).
