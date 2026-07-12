# Manifiesto — SP-06: Wizard de Crear/Editar Operación → Step 0 "Seleccionar DGO(s)"

**Estado: 🚧 BLOQUEADO — el plan describe un wizard que no es el que existe en el
código. Se detiene la implementación (regla dura de `/implementa`: "si descubres algo
que invalida el plan, detente y repórtalo, no improvises un diseño distinto") y se
propone actualizar el sub-plan antes de escribir código.**

## Ramas
- `carmi-digital`: se creó `refactor/customs-operation-sp06` (encadenada desde
  `refactor/customs-operation-sp05`, working tree previo intacto, nada comiteado).
- `carmi-odin-api-v2`: no se tocó (no se creó rama sp06 porque, según el análisis, este
  sub-plan tal como está escrito no requiere cambios de backend — ver más abajo). Sigue
  en `refactor/customs-operation-sp05`.

## Qué se investigó (antes de tocar código, siguiendo el paso 2 de `/implementa`)
Se leyó completo el wizard IN-SCOPE `app/(customerPortal)/customs-operation/createOperation/`:
`page.tsx`, `context/PedimentoWizardContext.tsx`, `components/StepInventorySelection.tsx`,
`components/stepOperationBasicData.tsx`, `stores/create-operation.store.ts`, y se
verificó (grep + lectura) el backend `src/dgo/**` (SP-05) y `src/operations/controllers/operations.controller.ts`.

## Hallazgos que invalidan el D1 del plan

1. **El Step 0 real de `page.tsx` no es `StepInventorySelection`.** El wizard vivo
   renderiza en `currentStepIndex === 0` un `<div>` placeholder literal: *"Reference
   selection UI needs to be implemented for single-reference flow"* (`page.tsx:429-433`).
   No hay ningún `import` de `StepInventorySelection` en `page.tsx`.

2. **`PedimentoWizardContext.tsx` / `StepInventorySelection.tsx` / `StepIncrementables.tsx`
   / `StepPartidasTable.tsx` / `StepPedimentoHeader.tsx` son código huérfano ("FASE
   5.3").** `grep -rln "PedimentoWizardProvider"` en todo el repo solo encuentra el
   propio archivo que lo define — `PedimentoWizardProvider` **nunca se monta** en ningún
   layout ni página. El wizard real usa exclusivamente `useCreateOperationStore`
   (zustand, `stores/create-operation.store.ts`). Este grupo de archivos usa datos mock
   (`mockPedimentoId = \`ped-draft-${Date.now()}\``) y llamadas a fetch comentadas que
   nunca se conectaron al store real. El D1 del plan los lista como "Reusa" — no se puede
   reusar código que no está en el flujo ejecutado.

3. **La línea "reactivar el cálculo de impuestos… `PedimentoWizardContext.tsx:191`
   (`/api/pedimentos/calculate-reserve`)" es un dato erróneo del plan.** Esa línea
   comentada es una llamada mock a `calculateAndReserveSubdivision` (reserva de
   inventario, Paso 2 del wizard huérfano), no tiene relación con impuestos, y vive en el
   mismo archivo muerto del hallazgo #2.

4. **El cálculo de impuestos real (TaxEngine) no existe en ningún archivo in-scope.**
   Se ubica por completo en el módulo **fuera de alcance** `components/operations/**`:
   `Step3CustomsHeader.tsx` (llama `POST /operations/preview-taxes` con debounce 500ms),
   `Step2FiscalConfig.tsx`, `Step4PaymentForms.tsx`, `TaxCalculationCard.tsx`. El store
   `create-operation.store.ts` sí tiene un campo `customsData.taxCalculation` y un
   passthrough (`state.customsData.taxCalculation = operationData.taxesCalculated`,
   línea 749) pero **nada en el wizard in-scope dispara ese cálculo** — no hay una
   llamada comentada que "reactivar"; sería wiring nuevo, tomando como referencia un
   patrón que vive en el módulo declarado fuera de alcance del refactor completo.

5. **El store real es de una sola referencia, no multi-DGO.** `create-operation.store.ts`
   modela `selectedReference: SelectedReference` (singular) y `sourceShipment` (singular)
   como el flujo activo ("Single reference flow — simplified shipment structure",
   comentario propio del archivo), aunque `OperationBasicData.referenceIds?: string[]`
   sugiere un scaffold multi-referencia abandonado a medias. Construir "seleccionar 2+
   DGOs → 2+ pedimentos" exige decidir de fondo cómo cambia esta forma (¿la operación
   sigue atada a 1 referencia con N DGOs de esa referencia, o puede abarcar DGOs de
   varias referencias?) — es una decisión de diseño de producto, no un ajuste mecánico.

6. **Backend DGO (SP-05) confirma la limitación ya documentada por SP-05**: `GET
   /dgo/reference/:referenceId` solo lista DGOs de **una** referencia (no hay endpoint
   para listar/seleccionar DGOs entre referencias). Y las consultas de Mercancía/Gastos/
   Identificadores (usadas por `stepOperationBasicData.tsx` vía `/operations/prepare/:id`,
   que a su vez llama a `operationsService.prepare(referenceIds, shipmentId)`) siguen
   filtrando por `referenceId`, no por `dgoId` — la limitación heredada de SP-05 que el
   prompt orquestador anticipó. Confirmado, no arreglado (fuera de mi alcance declarado).

## Por qué no se improvisó una solución
Los 4 pasos del plan ("cambiar Step 0 a DGOs", "prellenar desde DGO", "reactivar
impuestos", "soportar multi-DGO→multi-pedimento") requieren, dado lo anterior, diseñar
de cero: (a) qué reemplaza al placeholder del Step 0 (no existe un componente previo que
adaptar), (b) cómo el store pasa de single-reference a N-DGO (posiblemente N-referencia),
(c) cómo se dispara `preview-taxes` desde el wizard in-scope sin tocar el módulo
`operations` declarado fuera de alcance del refactor, y (d) cómo el payload de
`POST /operations` (que hoy es 1 `referenceId` + 1 array `shipments`) se convierte en
N pedimentos. Esto es rediseño de arquitectura, no "ajustar fuente de datos" como dice el
D1. Escribir código sobre esa base habría sido inventar un diseño no revisado por el
humano, exactamente lo que la regla dura de `/implementa` prohíbe.

## Recomendación
Regresar SP-06 a `/plan` para una entrevista corta que resuelva, con datos reales del
código (no los asumidos en el D1 actual):
- Confirmar que el Step 0 se construye desde cero (no hay componente previo que adaptar).
- Definir el modelo de datos del store para multi-DGO (¿multi-referencia también?).
- Decidir el mecanismo de cálculo de impuestos in-scope: ¿llamar `preview-taxes`
  directamente desde un componente nuevo del wizard in-scope (permitido: es consumir un
  endpoint existente, no tocar el módulo `operations`), o es necesario un endpoint nuevo?
- Confirmar si `POST /operations` acepta ya N pedimentos por operación o si eso también
  requiere un cambio de contrato (backend), lo cual añadiría un repo y una rama más al
  alcance del sub-plan.

## Archivos tocados
Ninguno de código de producto. Solo este manifiesto y la rama
`refactor/customs-operation-sp06` creada en `carmi-digital` (vacía, sin commits, working
tree previo de sp05 intacto).

## Pasos del sub-plan
Los 4 `- [ ]` **no se marcan como hechos** — no se implementó nada, para no dejar código
especulativo o parches rápidos sobre una base que el propio análisis muestra que no
aplica tal como está descrita.

## Verificación
No aplica todavía (no hay diff de producto que verificar). Gate estático y Playwright
quedan pendientes de la siguiente iteración, una vez el plan se actualice.
