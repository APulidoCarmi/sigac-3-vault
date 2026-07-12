# Manifiesto — SP-14: Solicitud de Fondos (3 categorías)

Implementado 2026-07-12. Ver sub-plan
[[2026-07-10-refactor-flujo-ejecutivo-sp14-solicitud-fondos]].

## Ramas
- `carmi-odin-api-v2`: `refactor/customs-operation-sp14` (encadenada desde
  `refactor/customs-operation-sp12`). Nada comiteado.
- `carmi-digital`: `refactor/customs-operation-sp14` (encadenada desde `sp12`).
  Nada comiteado.

## Brecha entre el D1 del sub-plan y el estado real del repo (importante)
El D1 decía: "reusa/refactoriza `ReferenceOperations.tsx:198-215`
(`handleSolicitarFondos`: **POST vacío** a `/operations/:id/solicitar-fondos` +
`alert()` nativo)". Al investigar el backend (`operations.controller.ts:202-213`,
`operations.service.ts:solicitarFondos` líneas 3267-3330) resultó que **no es un
POST vacío**: reenvía `LegacySystemData` (systemName `funding-requests`) al
Carmi DB legacy (`POST ${CARMI_DB_API}/funding-requests`). Es un puente de
migración para operaciones que vinieron del sistema legacy — no calcula nada,
solo reenvía datos pre-cargados, y **lanza 400 si la operación no tiene ese
registro legacy** (el caso normal para operaciones nativas de Odin, que es
justamente el público de este sub-plan).

**Decisión tomada:** no se tocó ni se eliminó `operations.controller.ts:202`/
`operations.service.ts:solicitarFondos` (sigue sirviendo a operaciones
migradas, fuera del alcance declarado). Se construyó un módulo nuevo e
independiente (`FundsRequest`/`FundsRequestItem`) para la solicitud nativa de
Odin, y el botón "Solicitar Fondos" del front ahora abre ese módulo nuevo en
vez de llamar al endpoint legacy directamente. Si en el futuro se requiere que
la solicitud nativa también dispare el reenvío a Carmi DB, es un enganche
adicional explícito (no incluido aquí, no estaba en el alcance).

## Backend (`carmi-odin-api-v2`)

### Schema (migración vía CLI: `add-funds-request-domain`)
- `FundsRequest` (`prisma/schema.prisma`, cerca de `OperationGlobalExpense`):
  cabecera de una solicitud — `operationId`, `requestNumber` (folio `SF-00001`
  consecutivo, mismo patrón que `Operation.operationNumber`), `type`
  (`INITIAL`/`COMPLEMENTARY`), `status` (`DRAFT`/`SUBMITTED`),
  `expensesAmount`/`taxesAmount`/`feesAmount`/`totalAmount` (congelados al
  enviar), `calculationSnapshot` (Json con el detalle del cálculo), `notes`,
  `submittedAt`.
- `FundsRequestItem`: renglones — `category` (`EXPENSES`/`TAXES`/`FEES`),
  `description`, `amount`, `sourceRef` (trazabilidad: id de pedimento para
  TAXES, método para FEES, libre para EXPENSES).
- `Operation.fundsRequests FundsRequest[]` (relación nueva).
- Migración: `prisma/migrations/20260712060600_add_funds_request_domain/`.

### Módulo nuevo `src/funds-requests/`
- `funds-requests.module.ts` — registrado en `app.module.ts` (import +
  entrada en `imports[]`, junto a `OperationsModule`).
- `funds-requests.controller.ts` — rutas bajo
  `operations/:operationId/funds-requests`:
  - `GET /` — historial de solicitudes (`FundsRequestsService.list`).
  - `GET /draft` — preview en vivo (recalcula impuestos/honorarios, trae
    gastos comprobados ya capturados en el draft abierto). No persiste nada.
  - `POST /expenses` — captura un renglón manual de gasto comprobado (crea el
    draft si no existe).
  - `DELETE /expenses/:itemId` — elimina un renglón de gasto comprobado del
    draft abierto.
  - `POST /submit` — recalcula impuestos/honorarios **del lado del servidor**
    (nunca confía en montos del cliente), congela los 3 montos + total, marca
    `SUBMITTED` (`INITIAL` si es la primera, `COMPLEMENTARY` si ya hay una
    previa enviada para la operación).
- `services/funds-requests.service.ts` — CRUD/orquestación descrita arriba.
- `services/funds-request-calculator.service.ts` — cálculo de las 2 categorías
  automáticas:
  - **Impuestos**: suma `Pedimento.liquidacionEfectivo` (sección "efectivo"
    del cuadro de liquidación, campo vigente — no deprecado, ver comentario en
    `prisma/migrations/20260314000000_add_pedimento_structured_fields`) de
    todos los pedimentos vinculados a la operación vía `OperationPedimento`.
    Alternativa no usada: `OperationTax` (snapshot de impuestos a nivel
    Operación) — se prefirió `liquidacionEfectivo` por ser lo que el sub-plan
    pide literalmente ("desde la sección 'efectivo' del cuadro de
    liquidación").
  - **Honorarios**: NO existía un motor de cálculo aplicado a una Operación
    (confirmado por investigación — `CompanyTariffConfiguration` /
    `TerrestrialTariff`/`AirTariff`/`MaritimeTariff`/`CompanyTariffService` son
    el catálogo de configuración, pero ningún service los aplicaba a una
    Operación concreta; el único punto cercano,
    `OperationCalculator.calculateHonorarios` en
    `src/tariff-calculation/calculators/operation-calculator.ts:198`, es un
    stub preexistente (`TODO: Implementar`, retorna `amount: 1` fijo) — **no
    se tocó**, es un módulo distinto (migración de stored procedures SIGAC 2)
    con TODOs propios y fuera del alcance declarado de este sub-plan). Se
    construyó `FundsRequestCalculatorService.calculateFees` desde cero,
    evaluando los 3 mecanismos del tarifario y tomando el **mayor** candidato
    aplicable (regla conservadora explícita, documentada en el código —
    decisión propia ante ausencia de una regla de combinación especificada en
    el sub-plan):
    - *Porcentaje*: `feesPercentage`/`feesMinimum` del tarifario activo
      (`PRICE`, `approvalStatus` `INTERNAL_APPROVED`/`CLIENT_APPROVED`) del
      `billingCompanyId ?? clientCompanyId` de la Operación, según modo
      (`TERRESTRIAL`/`AIR`/`MARITIME`, resuelto por
      `Reference.trafficType.code/description`) e IMPORT/EXPORT.
    - *Peso*: `TerrestrialTariff.consolidatedRanges` (rango por libra), solo
      para modo terrestre. Peso de la operación = suma de
      `Operation → OperationShipment → Shipment.grossWeight` (convertido a
      libras si `grossWeightUnit` empieza con "K"). **Nota:** el intento
      existente de peso en `OperationCalculator.calculateWeightImportArrival`
      está roto (lee `reference.grossWeight`/`unitComId`, campos que **no
      existen** en el modelo `Reference` — confirmado, `Reference` no tiene
      esos campos; el método compila solo porque su helper
      `getReferencesByOperation` está tipado `Promise<any[]>`). Se evitó
      repetir ese bug usando `Shipment.grossWeight` (campo real) en su lugar.
    - *Criterio*: `CompanyTariffService` del tarifario activo cuyo
      `serviceUnitCriterionId` apunta a un `ServiceUnitCriterion` con
      `code = 22` ("Honorarios", nombre documentado en el comentario de
      `operation-calculator.ts:196`), aplicando `amount * factor` con piso
      `minimum` y techo `maximum` (si > 0).
    - Si el cliente no tiene tarifario `PRICE` activo para el modo, regresa
      `amount: 0` con `warning` — **no bloquea** la solicitud (gastos e
      impuestos pueden seguir su curso; honorarios en 0 es visible en el
      drawer).
  - **Gastos comprobados**: NO se recicló `GlobalExpense`/
    `OperationGlobalExpense` (esos son incrementables/decrementables de
    *valoración aduanera* del pedimento — dominio distinto, `type` es
    `INCREMENTABLE`/`DECREMENTABLE`/`CUENTA_ADUANERA`, no hay una categoría
    "comprobado" en ese catálogo). Se implementó como captura manual dentro
    del draft (`FundsRequestItem` categoría `EXPENSES`), que es exactamente lo
    que pide el sub-plan ("integración" con Retorno de Vacío queda para SP-11,
    que puede sumar sus propios renglones a través del mismo endpoint
    `POST .../funds-requests/expenses`, o directamente vía el modelo).
- Tests: `funds-request-calculator.service.spec.ts` (10 tests) y
  `funds-requests.service.spec.ts` (9 tests) — 19 tests nuevos, todos verdes.

## Front (`carmi-digital`)
- `app/(customerPortal)/references/components/tabs/ReferenceOperations.tsx`:
  - Import nuevo: `SolicitudFondosDrawer` (línea junto a
    `OperationProformaDrawer`).
  - Estado `fundingLoadingId` → renombrado/reemplazado por
    `fundsRequestOperationId` (controla apertura del drawer, no un spinner de
    POST).
  - `handleSolicitarFondos` (antes líneas 198-221): ya no hace `fetch` ni
    `alert()` — solo `setFundsRequestOperationId(operationId)` para abrir el
    drawer. Comentario explica la razón (ver sección "Brecha" arriba).
  - Botón "Solicitar Fondos" del `DropdownMenuItem` (antes líneas 468-478):
    simplificado, ya no depende de `fundingLoadingId`/`Loader2` (el loading
    ahora vive dentro del drawer, en el botón "Enviar solicitud").
  - Import `Loader2` eliminado (quedó sin uso tras el cambio anterior).
  - Render nuevo al final: `<SolicitudFondosDrawer operationId={...} open={...}
    onClose={...} />`, hermano de `<OperationProformaDrawer />`.
- Nuevo:
  `app/(customerPortal)/references/components/drawers/SolicitudFondosDrawer.tsx`
  — `Sheet` (mismo patrón que `OperationProformaDrawer.tsx`: `useQuery` para
  el draft, `useMutation` + `useToast` para agregar/quitar gasto y para
  enviar). 3 secciones (Gastos comprobados editable, Impuestos y Honorarios de
  solo lectura) + total, badge Inicial/Complementaria. Usuario actual vía
  `useSelector(selectUser)` de `store/userSlice` (mismo patrón que
  `OperationProformaDrawer.tsx`), `user.UserID` como `createdBy`.
- No se tocó `app/(customerPortal)/finance/components/Funds-stepper.tsx` ni
  `app/actions/finance/get-SolicitudFondos.ts` — son el flujo de tesorería del
  customer portal (pago de fondos ya solicitados contra el sistema legacy
  Carmi DB), dominio distinto confirmado por diseño explícito del sub-plan.
  Dato de esa investigación: el legacy modela las mismas 3 categorías + una
  4ta ("Garantía", `TipoAntId` 1-4: Impuestos/Gastos_Comprobados/Honorarios/
  Garantía) — no se replicó "Garantía" aquí porque no está en el alcance del
  sub-plan (solo 3 categorías, decisión ya tomada en el paraguas #4).

## Para el subagente de SP-11 (Retorno de Vacío / movimientos marítimo)
- El endpoint para sumar gastos comprobados es
  `POST /operations/:operationId/funds-requests/expenses` (crea el draft si
  no existe) — puede reusarse para enganchar los gastos de Retorno de Vacío
  sin duplicar el modelo.
- El estado de "Solicitud de Fondos" para un botón tipo "Generar Cita" puede
  consultarse con `GET /operations/:operationId/funds-requests` (historial;
  `status: 'SUBMITTED'` = ya se envió al menos una) o
  `GET /operations/:operationId/funds-requests/draft` (preview en vivo, con
  `type: 'INITIAL'|'COMPLEMENTARY'`).
- `ReferenceOperations.tsx` quedó con el drawer nuevo integrado; si SP-11
  necesita disparar la solicitud de fondos desde otra pantalla (no el tab
  Operaciones), reutiliza `SolicitudFondosDrawer` tal cual (recibe
  `operationId`/`open`/`onClose`, no depende de nada específico del tab).

## Verificación
- **Backend**: `npx prisma migrate dev --name add-funds-request-domain`
  aplicado limpio contra la DB del entorno (`carmi_dev`). `npx tsc --noEmit`:
  sin errores nuevos (mismos 15 preexistentes ya documentados en el manifiesto
  de SP-12: `twilio.service.spec.ts`, `seal-resolver.service.spec.ts`,
  `company-patent-signature-config.service.spec.ts`, `audited.spec.ts`, 3
  `*.e2e-spec.ts`). `npx eslint src/funds-requests src/app.module.ts`: 0
  errores. `npx jest src/funds-requests src/operations`: 9 suites, 76 tests,
  todos verdes (19 nuevos de este sub-plan + 57 preexistentes de
  `src/operations` sin regresión).
- **Front**: `npx tsc --noEmit`: 0 errores en todo el repo. `npx eslint` sobre
  `ReferenceOperations.tsx` y `SolicitudFondosDrawer.tsx`: 0 errores.
- **Playwright**: no ejecutado — el entorno de esta sesión no tiene el flujo
  de front levantado con datos reales (operación con pedimento liquidado y
  tarifario de cliente activo) para un E2E visual significativo. Pendiente de
  sesión humana: crear una solicitud con las 3 categorías recalculadas y una
  complementaria, confirmar toast y ausencia de errores de consola (criterio
  de verificación del sub-plan).

## Desviaciones / decisiones fuera de lo literal del sub-plan
1. El D1 asumía un backend "vacío"; en realidad había un puente legacy
   funcional para otro caso de uso. Se documentó arriba y se dejó intacto.
2. Regla de combinación de los 3 mecanismos de honorarios (porcentaje/peso/
   criterio) no estaba especificada — se implementó "el mayor candidato gana"
   como regla conservadora explícita y documentada en código; puede requerir
   ajuste de negocio en revisión humana.
3. No se reutilizó `GlobalExpense`/`OperationGlobalExpense` para "gastos
   comprobados" — es un dominio distinto (valoración aduanera del pedimento).
   Se optó por captura nueva y propia dentro de `FundsRequestItem`.
4. Se agregaron 19 tests nuevos (no pedidos explícitamente) dado que
   Playwright no es viable en este entorno y la lógica de cálculo (impuestos/
   honorarios) es sensible (montos de dinero) — se consideró indispensable
   cubrirla con unit tests.
5. No se tocó `OperationCalculator.calculateHonorarios` (el stub existente en
   `tariff-calculation`) — pertenece a un módulo de migración de stored
   procedures distinto, con TODOs propios preexistentes y fuera del alcance
   declarado (`src/operations` sí, pero `src/tariff-calculation` no estaba
   mencionado y resolver sus TODOs habría expandido el alcance
   considerablemente).
