# Sub-plan SP-16: Stepper de Despacho — 7 pasos, Shipper condicional, unificación de despacho backend (REDISEÑADO)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Historial de esta sección (no borrar el rastro)

- **2026-07-12, primer intento de `/implementa`:** se detuvo en fase de diseño/verificación por 3 premisas
  falsas del D1 original. Diagnóstico en `Planes/.manifiestos/2026-07-10-refactor-flujo-ejecutivo-sp16-despacho-stepper.md`.
- **2026-07-12, replanteo (este documento):** el usuario resolvió 3 de las 4 decisiones pendientes
  (retirar GLOSA del stepper de verdad porque vive en Zeus/Expediente vía SP-04/SP-05; Shipper como
  paso condicional por umbral de pago del pedimento; delegó al subagente de diseño la 4ª decisión —
  qué controller/sistema de despacho prevalece). Este documento reemplaza el D1 anterior con la
  arquitectura resultante de la investigación de código real (branch `refactor/customs-operation-sp18`
  en ambos repos, sin commitear).

## Contexto

Pantalla #25 del [[Inventario_Pantallas_v3]] (🟡 modificar). El despacho es un stepper orquestado por
Temporal. Origen: glosario "Despacho" y Brechas #1/#2/#3 del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]], [[2026-06-25 - Daily Scrum - flujo de importación y
checklist dinámico]] y [[2026-06-30 - Prueba operación real - glosa, pago y DODA]].

**Decisiones de producto (ya tomadas, no se vuelven a preguntar):**
1. GLOSA desaparece del stepper de despacho → 7 pasos visibles en el front.
2. La glosa de facturas vive en el Expediente vía Zeus (SP-04 `GlosaEngineService` +
   `assertInvoicesGlossedOk`), no en el arranque de despacho.
3. Shipper es un paso condicional: se añade al stepper solo si el monto de liquidación en efectivo
   del pedimento supera USD $2,500.
4. (Delegada a este subagente) Qué sistema/controller de despacho backend prevalece — resuelta abajo.

## D1 corregido — hallazgos verificados contra el código real

### A. Hay tres cosas distintas que todas se llaman "glosa" — hay que desambiguarlas

1. **Glosa de facturas (SP-04/SP-05, Expediente/Zeus).** `GlosaEngineService` +
   `ReferenceDocumentsService.assertInvoicesGlossedOk(referenceId)`. Se invoca en
   `operations.service.ts` `create()` línea ~915, **solo si la operación se creó con
   `selectedDgoIds.length > 0`**, y en el endpoint de solo-lectura
   `GET /reference-documents/reference/:id/can-start-operation` (wizard `StepDgoSelection.tsx`). **No
   se invoca en `startDispatchWorkflow()`.** Esta es la glosa que el usuario confirma que vive en el
   Expediente y que NO debe gatear el despacho — se confía en que ya se resolvió antes.
2. **GLOSA como step INTERNO del workflow Temporal (`StepName.GLOSA`, `dispatch.workflow.ts` +
   `dispatch.activities.ts::processGlosaStep`).** Esto **no es la glosa de facturas** — es una
   comparación campo a campo entre el pedimento generado y los datos capturados (Reference/Invoices/
   Items/Operation), vía `GET :id/dispatch/pedimento/glosa` → `PedimentoGlosaResponseDto`. El
   workflow la ejecuta automáticamente (`stepConfig.type === 'INTERNAL'`) y **si `hasErrors` o
   `!canProceed`, lanza error y el step queda `FAILED`, pausando el workflow hasta que el usuario
   fuerce o corrija.** Es decir: **este gate SÍ existe hoy y SÍ bloquea el despacho a nivel
   backend**, independientemente de si el front le muestra una pestaña.
3. **`StepPedimento.tsx` (front, exporta `StepGlosa`)** es la pestaña de UI que hoy renderiza el
   resultado de (2) y permite regenerar el pedimento (`/dispatch/regenerate-pedimento`). Sigue
   activa hoy — **no está huérfana** (corrección al D1 original).

**Consecuencia de diseño:** retirar la pestaña "Glosa" del stepper del front (decisión de producto)
**es seguro** precisamente porque el gate real (2) vive dentro del workflow como step interno
automático, no depende de que el front tenga una pestaña dedicada. El plan es: el workflow sigue
ejecutando `GLOSA` como antes (mismo `processGlosaStep`, mismo bloqueo ante discrepancias), pero el
front dejará de listarlo como pestaña navegable en `DISPATCH_STEPS` — se mostrará, si acaso, como un
estado transitorio/automático entre "Manifestación" y "Validación" (spinner "Validando pedimento…"),
reutilizando el badge de progreso que `OperationStepsTabs.tsx` ya pinta para steps `IN_PROGRESS`, sin
tab clicable ni contenido propio. Si el step queda `FAILED` (discrepancias), se debe mostrar un panel
de error inline (reaprovechando el contenido hoy en `StepPedimento.tsx`) para que el usuario pueda
forzar/reintentar — **no se elimina la posibilidad de intervención manual, solo el tab dedicado**.

### B. Los dos sistemas de despacho backend — confirmado el diagnóstico, con matiz importante

- **Sistema vivo (Temporal/`DispatchStep`):** tabla `DispatchStep` (`prisma/schema.prisma:2339`),
  enum `StepName` (8 valores, `prisma/schema.prisma:11627`), `dispatch.workflow.ts`,
  `dispatch.activities.ts`, `dispatch-steps.constant.ts` (`DISPATCH_STEPS_CONFIG`/`ORDERED_STEPS`,
  única fuente de verdad de orden y tipo INTERNAL/EXTERNAL por step).
- **Sistema legacy (JSON blob):** enum `DispatchStepId` (11 valores, `dispatch.dto.ts`), persistido en
  `operation.operationMetadata.dispatch` (no en `DispatchStep`). Vive dentro de
  `operation-dispatch.service.ts` (5972 líneas) en un grupo de métodos concreto:
  `completeModulacion()`, `completeShipper()`, `getDispatchStatus()` (el que lee del JSON blob, NO el
  usado hoy por el front). **Estos 3 métodos + sus tipos (`OperationDispatchMetadata` en
  `dispatch.dto.ts`, `DispatchStepId`) son el sistema a eliminar.**
- **Hallazgo nuevo, decisivo:** `operation-dispatch.service.ts` **no es puramente legacy** — la
  mayoría de sus métodos (`completeEDocuments`, `completeCove`, `completeManifestacion`,
  `enviarValidacionCaarem`, `enviarPagoCaarem`, `enviarDodaCaarem`, `enviarModulacionCaarem`,
  `getPedimentoGlosa`, `regeneratePedimento`, `getOperationCoves`, `getOperationPedimento`,
  `confirmarValidacionCaarem`) son la capa real de construcción de payloads y llamadas a Bifrost, y
  **son invocados directamente por `dispatch.activities.ts::callBifrostApi`** (acceso vía índice de
  propiedad privada, ej. `operationDispatchService['completeEDocuments'](...)`). Es decir: el
  workflow Temporal (autoritativo) **ya depende de este servicio** para su lógica de negocio real —
  no hay que reescribir esa lógica, solo purgar la parte que maneja el JSON blob paralelo.

### C. Confirmación de integración con Bifrost (para no romperla)

- Camino de ida: `dispatch.workflow.ts` (steps EXTERNAL) → `callBifrostApi` activity
  (`dispatch.activities.ts`) → delega en los métodos reales de `OperationDispatchService`
  (`enviarValidacionCaarem`, `enviarPagoCaarem`, `enviarDodaCaarem`, `enviarModulacionCaarem`,
  `completeEDocuments/Cove/Manifestacion`) que llaman a Bifrost vía `BifrostClientService`
  (`src/common/services/bifrost-client.service.ts`) y persisten `bifrostId` en el registro
  `DispatchStep` correspondiente.
- Camino de vuelta (webhook): `POST /api/webhooks/bifrost` (`webhook.controller.ts`) — busca el
  `DispatchStep` **por `bifrostId`** (campo que solo existe en la tabla `DispatchStep`, no en el JSON
  blob legacy), valida HMAC + 3-way validation, y llama a
  `temporalService.signalStepCompleted(...)`, que resuelve el `condition()` del workflow.
- **Conclusión dura:** el sistema legacy (JSON blob) es **estructuralmente incompatible** con el
  webhook real de Bifrost — no tiene forma de correlacionar una respuesta asíncrona porque no
  persiste `bifrostId`. `completeShipper()`/`completeModulacion()` (legacy) nunca podrían conectarse
  al flujo real de Bifrost tal como están escritos hoy. Esto por sí solo decide la pregunta 4: **el
  sistema Temporal/`DispatchStep` es el único que puede sostener la exigencia del usuario de "que la
  conexión a Bifrost siga funcionando"**; el legacy nunca estuvo conectado a Bifrost de verdad.
- Salud: `src/health/indicators/bifrost.health.ts` hace un GET liviano a `${bifrostBaseUrl}/health`
  vía `BifrostClientService` — no depende de qué sistema de despacho sobreviva; no requiere cambios.

### D. Los dos controllers en colisión — resolución

- `DispatchController` (`src/operations/controllers/dispatch.controller.ts`, base `operations`):
  - `POST :id/dispatch/start` (crea `DispatchStep`s + arranca Temporal, pero con checks pobres:
    lanza `Error` genérico, no `NotFoundException`/`BadRequestException`; solo valida
    `dispatchSteps.length > 0`, sin distinguir "en progreso" de "ya reiniciable tras cancelar").
  - `GET :id/dispatch/status` **duplicado dentro de sí mismo**: `getDispatchStatusNew` (línea 108,
    lee de `DispatchStep`/Temporal — el que realmente consumen `useOperationDispatch.ts` y
    `useDispatchStatus.ts` hoy, porque NestJS resuelve la ruta con el primer método registrado que
    matchea) y `getDispatchStatus` (línea 286, delega a `dispatchService.getDispatchStatus()` —
    lee del JSON blob legacy, **inalcanzable en runtime**, código muerto por sombra de ruta).
  - `PATCH :id/dispatch/modulacion` → `dispatchService.completeModulacion()` (legacy, JSON blob —
    confirmado en B; no mueve ningún `DispatchStep`, por lo que "completar modulación" desde aquí no
    avanza el workflow real ni el `currentStep` que lee el front).
  - `POST :id/dispatch/abort/:stepId` (cancela un `DispatchStep` puntual + señal de cancelación al
    workflow completo — semántica distinta y más basta que "resume").
  - Rutas que sí son necesarias y activamente usadas por el front (`coves`, `validacion/caarem`,
    `pedimento/glosa`, `regenerate-pedimento`, `/pedimento`, `steps/edocuments|cove|manifestacion|
    pedimento|validacion` de ejecución manual) — todas delegan a los métodos "buenos" de
    `OperationDispatchService` descritos en B.
- `OperationsController` (`src/operations/controllers/operations.controller.ts`, misma base
  `operations`):
  - `POST :id/dispatch/start` → `operations.service.ts::startDispatchWorkflow()` — **gana la
    colisión de ruta por orden de registro del módulo** (`controllers: [OperationsController,
    DispatchController]` en `operations.module.ts`). Implementación superior: valida
    `NotFoundException` si la operación no existe, valida `IN_PROGRESS` con `BadRequestException`
    explícito, **limpia `DispatchStep`s previos y permite reiniciar** tras cancelar (el de
    `DispatchController` no soporta reinicio limpio), y usa el mismo `DISPATCH_STEPS_CONFIG`
    compartido.
  - `POST :id/dispatch/cancel` → `cancelDispatchWorkflow()` (cancela el workflow completo de forma
    ordenada).
  - `POST :id/dispatch/resume` → señal `resumeStepSignal` real vía `temporalService.signalResumeStep`
    — **esta es la ruta que consume `useDispatchStatus.ts::resumeStep` en `DispatchMonitor.tsx`,
    hoy activa en producción.** No existe un equivalente "resume" en `DispatchController` (solo
    `abort`, que es una operación distinta).

**Veredicto (pregunta 4, resuelta):** se unifica todo sobre el sistema **Temporal/`DispatchStep`**.
Se conserva el `OperationsController` como dueño de las rutas de control del ciclo de vida del
workflow (`start`/`cancel`/`resume`), por ser la implementación más completa y porque ya gana la
colisión de rutas en runtime. Se conserva `DispatchController` **recortado**: pierde `dispatch/start`
duplicado, pierde el `getDispatchStatus` (legacy, ya inalcanzable — se retira el código muerto, no
solo la ruta), pierde `PATCH :id/dispatch/modulacion` (legacy), pierde `POST :id/dispatch/abort/
:stepId` (semántica solapada con `resume`/`cancel` de `OperationsController`; se evalúa fusionar como
`resume` con `action: 'FORCE_CONTINUE'`/cancelación de step único si el frontend lo necesita — ver
tarea de verificación abajo) y conserva todas las rutas de datos/ejecución de pasos que sí delegan a
`OperationDispatchService` (Bifrost-facing). No hay razón real para mantener ambos sistemas vivos: el
legacy nunca estuvo conectado a Bifrost y su único consumidor front (`completeModulacion` del hook
`useOperationDispatch.ts`) queda reemplazado por la implementación real de MODULACION vía
`enviarModulacionCaarem`/Bifrost dentro del mismo Temporal workflow.

### E. MODULACION — no es "revivir un step muerto", es conectar el front al mecanismo EXTERNAL ya construido

`DISPATCH_STEPS_CONFIG.MODULACION = { order: 8, type: 'EXTERNAL' }`. El workflow ya la trata como
step externo: llama `callBifrostApi('MODULACION')` → `operationDispatchService.enviarModulacionCaarem()`
(payload de aviso de cruce vía Bifrost) → espera el webhook real (`stepCompletedSignal`) para
completar. **Esto ya tiene lógica de negocio real escrita** (`enviarModulacionCaarem`, línea ~3810 de
`operation-dispatch.service.ts`) — lo que falta es una UI/tab en el front que refleje el estado
`AWAITING_BIFROST`/`FAILED`/`COMPLETED` de este step (hoy `renderStepContent()` devuelve `null` para
`modulacion`). No se debe reconectar `StepModulacion.tsx` (archivo muerto, escrito contra el sistema
legacy: PATCH manual VERDE/ROJO) — se debe **construir un componente nuevo** que solo muestre estado
de solo-lectura (progreso/errores del step EXTERNAL) y, si el negocio exige una intervención manual
tipo VERDE/ROJO además del webhook automático de Bifrost, exponerla como una acción de "forzar
resultado" que pase por `resumeStepSignal`/`FORCE_CONTINUE` (mecanismo ya existente en el workflow
para steps fallidos) en vez de reabrir el endpoint legacy `PATCH dispatch/modulacion`.

### F. Shipper condicional (> USD $2,500) — campo confirmado, no existe hoy en ningún sistema

- Campo confirmado: `Pedimento.liquidacionEfectivo` (`prisma/schema.prisma:6198`, `Decimal(14,2)`).
  Ya es consumido en producción por SP-14 (`funds-request-calculator.service.ts` línea ~40: "Categoría
  Impuestos de pedimento: suma `Pedimento.liquidacionEfectivo`"). Es el monto real de liquidación en
  efectivo del pedimento vinculado a la operación — el campo correcto para comparar contra USD
  $2,500 (confirma el diagnóstico del manifiesto: SP-14 ya usa exactamente este campo).
- No existe hoy `StepName.SHIPPER` en el enum Temporal, ni lógica de umbral en ningún repo. Es
  funcionalidad nueva a construir sobre el sistema vivo: no se reconecta `StepShipper.tsx` (archivo
  muerto sin importadores, escrito contra `dispatchStatus?.dispatch?.steps?.shipper`, una forma que
  no existe en el tipo actual `OperationDispatchMetadata.steps` del sistema vivo — confirma el
  diagnóstico).
- **Diseño del step SHIPPER:**
  - Backend: añadir `SHIPPER` al enum Prisma `StepName` (migración nueva), a `DISPATCH_STEPS_CONFIG`
    con `order` entre `PAGO` (order 6) y `DODA` (order 7) — recorrer `order` de DODA→8 y
    MODULACION→9 — y `type: 'INTERNAL'` (no requiere Bifrost; es una asignación operativa
    interna/administrativa, salvo que producto indique lo contrario — ver pregunta abierta al final).
  - El workflow debe **crear el `DispatchStep` de `SHIPPER` dinámicamente, justo antes de ejecutar
    PAGO/DODA** (no al arrancar el despacho): en ese punto del workflow, tras resolver el `Pedimento`
    vinculado a la operación (ya con `liquidacionEfectivo` poblado, calculado en Validación/CAAREM),
    comparar `Number(pedimento.liquidacionEfectivo ?? 0) > 2500` e insertar el `DispatchStep` de
    SHIPPER en ese momento solo si aplica. El stepper pasa de 7 a 8 pasos en caliente — el workflow
    debe leer el conjunto de steps ya creados en DB (no una lista estática fijada al inicio) para
    soportar esta inserción a mitad de flujo.
  - Front: en `DISPATCH_STEPS`/`StepId`/`STEP_LABELS` (`types/operation-dispatch.ts`) añadir
    `shipper` tras `pago` y antes de `doda`. `OperationStepsTabs.tsx` debe **filtrar dinámicamente**
    el step `shipper` de la lista renderizada según si el `dispatchStatus.dispatchSteps` trae un
    registro con `stepName: 'SHIPPER'` (si no viene, el step no aplica a esa operación — no se
    muestra ni cuenta para el total ni el progreso). Construir `StepShipper.tsx` nuevo desde cero
    contra la forma real de `dispatchStatus.dispatchSteps` (no `dispatch.steps.shipper`).

## Reusa / Refactoriza / Crea / Limpia (resumen ejecutable)

- **Reusa:** `dispatch.workflow.ts`, `dispatch.activities.ts`, `dispatch-steps.constant.ts`,
  `webhook.controller.ts`, `BifrostClientService`, `bifrost.health.ts` — no tocar el mecanismo de
  Bifrost. Reusa también los métodos Bifrost-facing de `OperationDispatchService` listados en B/C.
- **Refactoriza:**
  - `prisma/schema.prisma`: añadir `StepName.SHIPPER` (migración nueva, mantener orden lógico
    EDOCUMENTS→COVE→MANIFESTACION→GLOSA→VALIDACION→PAGO→SHIPPER(cond.)→DODA→MODULACION).
  - `dispatch-steps.constant.ts`: añadir `SHIPPER` a `DISPATCH_STEPS_CONFIG` y reordenar `order` de
    DODA/MODULACION.
  - `operations.service.ts::startDispatchWorkflow()` (y el que sobreviva en `DispatchController` si
    aplica): resolver el `Pedimento` de la operación, calcular condicional de Shipper, crear
    `DispatchStep`s condicionalmente (no siempre los 9 valores del enum).
  - `dispatch.controller.ts`: eliminar `startDispatch` (dispatch/start duplicado), eliminar
    `getDispatchStatus` (legacy, código muerto), eliminar `completeModulacion` (PATCH legacy),
    evaluar fusión de `abortStep` con el mecanismo `resume`/`cancel` de `OperationsController`.
  - `operation-dispatch.service.ts`: eliminar `completeModulacion()`, `completeShipper()`,
    `getDispatchStatus()` (legacy) y cualquier helper usado solo por ellos (`createInitialDispatch`
    si queda sin otros consumidores). Conservar el resto intacto (es la capa Bifrost real).
  - `dispatch.dto.ts`: eliminar `DispatchStepId`, `OperationDispatchMetadata` (legacy),
    `CompleteModulacionDto` (si no se reutiliza tal cual — ver E), y los tipos `*Data` que solo sirven
    al JSON blob (`ShipperData` legacy se reemplaza por el nuevo modelo `DispatchStep`, no se
    reutiliza tal cual).
  - `types/operation-dispatch.ts`, `OperationStepsTabs.tsx`, `useOperationDispatch.ts` (front):
    quitar `glosa` de `DISPATCH_STEPS` (7 pasos base + shipper condicional = 7 u 8 según pedimento),
    añadir `shipper` condicional, quitar el llamado a `completeModulacion` legacy y reemplazar por
    lectura del step `MODULACION` (EXTERNAL) vía polling normal + acción de `resume`/`FORCE_CONTINUE`
    si aplica.
  - Resolver duplicación de `DispatchMonitor`: **decisión** — mantener
    `components/dispatch/DispatchMonitor.tsx` (usa `useDispatchStatus.ts`, ya alineado al sistema
    Temporal, único con importador real: `customs-operation/[id]/dispatch/page.tsx`) y **eliminar**
    `app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx` (155 líneas, sin
    importadores — código muerto duplicado, no mencionado en el D1 original). Ambas rutas front
    (`[id]/page.tsx` con tabs y `[id]/dispatch/page.tsx` con monitor) pueden convivir: la primera es
    la vista operativa paso a paso, la segunda un monitor de solo lectura/recuperación de errores;
    no hay conflicto real una vez que ambas leen del mismo `/dispatch/status` (Temporal).
- **Crea:**
  - `StepShipper.tsx` nuevo (no revivir el muerto) contra la forma real de `dispatchStatus`.
  - Componente de estado para `MODULACION` (reemplaza el `null` de `renderStepContent()`), de solo
    lectura + acción de forzar continuar ante fallo, sin el formulario VERDE/ROJO legacy.
  - Panel de error inline para el step interno GLOSA (discrepancias campo a campo) dentro del flujo
    entre Manifestación y Validación, reaprovechando el contenido de `StepPedimento.tsx` pero sin tab
    dedicado.
- **Limpia (archivos muertos, confirmado sin importadores):** `steps/StepShipper.tsx` (el viejo),
  `steps/StepModulacion.tsx`, `steps/StepGlosa.tsx`, `StepGlosa.example.tsx`, `StepVucem.tsx`,
  `StepPrevalidacion.tsx`, `app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx`.

## Fuera de alcance

- El motor Temporal de bajo nivel (retries, timeouts) fuera de lo estrictamente necesario para
  Shipper condicional y limpieza de rutas.
- SP-04/SP-05 (glosa de facturas en Expediente/Zeus) — se asume cerrado y correcto, solo se confía en
  él, no se reabre.
- SP-14 (Solicitud de Fondos) — solo se lee `Pedimento.liquidacionEfectivo`, no se modifica su
  cálculo.

## Pasos

- [x] Migración Prisma: añadir `StepName.SHIPPER`; actualizar `DISPATCH_STEPS_CONFIG` (orden y tipo)
      en `dispatch-steps.constant.ts`. (2026-07-12, vía `npx prisma migrate dev`, migración
      `20260712093255_add_shipper_step_name`.)
- [x] `dispatch.workflow.ts`: justo antes de ejecutar PAGO/DODA, resolver el `Pedimento` vinculado,
      calcular `liquidacionEfectivo > 2500 USD` e insertar dinámicamente el `DispatchStep` de SHIPPER
      (tipo `INTERNAL`) solo si aplica. (2026-07-12.)
- [x] `dispatch.workflow.ts`/`dispatch.activities.ts`: confirmar que `ORDERED_STEPS`/el render del
      stepper se derivan de los `DispatchStep`s ya creados en DB para esa operación (no de una lista
      estática fijada al inicio), de modo que el front pueda ver crecer el stepper de 7 a 8 pasos en
      caliente cuando se inserta SHIPPER. (2026-07-12: confirmado — `getDispatchStatusNew` ya lee
      `operation.dispatchSteps` reales de DB; el front filtra dinámicamente vía
      `dispatchStatus.dispatchSteps` (ver `OperationStepsTabs.tsx::visibleSteps`). El workflow sigue
      iterando `ORDERED_STEPS` como orden de ejecución fijo — eso es correcto y no necesita ser
      dinámico, solo la creación/presencia del `DispatchStep` de SHIPPER lo es.)
- [x] `dispatch.workflow.ts`: implementar SHIPPER como step `INTERNAL` (sin llamada a Bifrost, sin
      espera de webhook) con la lógica real de asignación que corresponda. (2026-07-12:
      `processShipperStep` implementado como checkpoint de auditoría/visibilidad — ver desviación
      documentada en el manifiesto, no había regla de negocio adicional especificada más allá del
      umbral que ya decide su creación.)
- [~] Purga backend: eliminar `DispatchController.startDispatch` (2026-07-12, hecho — duplicado
      inalcanzable confirmado). `getDispatchStatus` (legacy), `completeModulacion` y los tipos legacy
      de `dispatch.dto.ts` **NO se eliminaron** — investigación (2026-07-12) encontró que
      `getDispatchStatus` sostiene la ruta viva `regenerate-pedimento`, que `completeModulacion`/
      `executeStepPublic` son rutas HTTP en vivo también basadas en el blob legacy (no solo las 3
      métodos que el D1 asumía), y que `confirmarValidacionCaarem` podría ser un webhook externo de
      Bifrost configurado fuera de este repo — no se puede confirmar que esté muerto solo con grep.
      Ver manifiesto para el detalle y la recomendación de un sub-plan de purga dedicado.
- [x] Front: quitar `glosa` de `DISPATCH_STEPS`/`StepId`/`STEP_LABELS`; añadir `shipper` condicional
      (filtrado dinámico en `OperationStepsTabs.tsx` según presencia real en `dispatchStatus.dispatchSteps`).
      (2026-07-12.)
- [x] Front: crear `StepShipper.tsx` nuevo contra la forma real de `dispatchStatus.dispatchSteps`.
      (2026-07-12: se reescribió el archivo existente por completo, contenido 100% nuevo.)
- [x] Front: implementar contenido real para `modulacion` en `renderStepContent()` (solo lectura +
      forzar continuar ante fallo, sin formulario legacy VERDE/ROJO). (2026-07-12: nuevo
      `StepModulacionStatus.tsx`, solo lectura; ver desviación documentada en el manifiesto sobre
      FORCE_CONTINUE en steps EXTERNAL.)
- [x] Front: panel de error inline para el step interno GLOSA (discrepancias), sin tab dedicado;
      quitar `StepPedimento`/`StepGlosa` de la navegación de tabs pero conservar su lógica de
      renderizado de discrepancias como panel condicional. (2026-07-12.)
- [x] Front: quitar `completeModulacion` (PATCH legacy) de `useOperationDispatch.ts`; usar
      `resume`/`FORCE_CONTINUE` para intervención manual sobre MODULACION si aplica. (2026-07-12:
      FORCE_CONTINUE no aplica a MODULACION por ser EXTERNAL — ver manifiesto; solo RETRY vía
      `StepErrorAlert`.)
- [x] Eliminar archivos muertos confirmados: `steps/StepShipper.tsx` (viejo), `steps/StepModulacion.tsx`,
      `steps/StepGlosa.tsx`, `StepGlosa.example.tsx`, `StepVucem.tsx`, `StepPrevalidacion.tsx`,
      `app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx`. (2026-07-12.)
- [ ] Verificación end-to-end: iniciar un despacho con pedimento > $2,500 USD (aparece Shipper, 8
      tabs) y uno ≤ $2,500 (7 tabs, sin Shipper); confirmar que GLOSA sigue bloqueando internamente
      ante discrepancias (sin tab, con panel de error); confirmar que el webhook de Bifrost
      (`/api/webhooks/bifrost`) sigue resolviendo steps `AWAITING_BIFROST` sin cambios. **Pendiente**
      — sin `dev_url`/entorno levantado para este cliente; solo se corrieron puertas estáticas
      (tsc/eslint/jest) en ambos repos, ver manifiesto.

## Riesgos y side effects

- Cambiar `DISPATCH_STEPS_CONFIG.order` (insertar SHIPPER) no afecta operaciones ya en curso salvo que
  se reconstruyan sus `DispatchStep`s — validar que el cambio de config no rompe operaciones con
  despacho ya iniciado bajo el enum de 8 valores (migración de datos existentes fuera de alcance si no
  hay operaciones en producción con despacho activo; confirmar antes de desplegar).
- Retirar `DispatchController.abortStep` sin verificar si algún flujo del front lo sigue llamando
  directamente (grep de `dispatch/abort` en el front antes de eliminar la ruta).
- El campo `Pedimento.liquidacionEfectivo` puede no estar poblado aún al momento de
  `startDispatchWorkflow` si el pedimento se genera más adelante en el propio flujo de despacho
  (step PEDIMENTO/MANIFESTACION) — validar el orden real de disponibilidad del dato antes de decidir
  el umbral de Shipper (ver pregunta abierta).

## Decisiones finales sobre Shipper (usuario, 2026-07-12)

1. **Timing:** la condición se evalúa **dinámicamente, justo antes de Pago/DODA** — no al arrancar el
   despacho. El `DispatchStep` de SHIPPER se inserta sobre la marcha si aplica, una vez que
   `Pedimento.liquidacionEfectivo` ya está poblado (tras Validación/CAAREM). El stepper puede crecer
   de 7 a 8 pasos ya iniciado el despacho — el front debe manejar esta inserción dinámica (no asumir
   un conteo fijo de pasos desde el primer render).
2. **Tipo de step:** **INTERNAL, sin Bifrost.** Es una asignación operativa que el ejecutivo resuelve
   dentro del sistema, sin llamada externa ni espera de webhook.

## Criterios de verificación

- Gate estático verde en ambos repos.
- Playwright: iniciar un despacho con un pedimento cuya `liquidacionEfectivo` > $2,500 USD → 8 tabs
  visibles (con Shipper); con ≤ $2,500 → 7 tabs (sin Shipper); sin pestaña "Glosa" en ningún caso;
  Modulación con contenido real (no `null`); sin errores de consola; el webhook de Bifrost sigue
  resolviendo steps en ambiente de prueba (mock/staging).

## Estado

✍️ Redactado — listo para `/implementa` (2026-07-12). D1 corregido con las 4 decisiones de producto
resueltas (GLOSA retirada del stepper, glosa vive en Zeus/Expediente, Shipper condicional con timing
dinámico antes de Pago/DODA y tipo INTERNAL, unificación sobre el sistema Temporal/`DispatchStep`).
Sin preguntas abiertas.
