# Manifiesto — SP-16: Stepper de Despacho (8 pasos → retirar GLOSA, Shipper propio)

**Estado: 🚧 BLOQUEADO (2026-07-12), primer intento de `/implementa`.** Se detuvo en fase
de diseño/verificación, ANTES de tocar código, porque el D1 del sub-plan da por buenas
varias premisas que la auditoría contra el código real (front `carmi-digital` rama
`refactor/customs-operation-sp16`, back `carmi-odin-api-v2` rama
`refactor/customs-operation-sp16`) invalida o matiza de forma material. No se ha escrito
ni modificado ningún archivo de producto; solo se crearon las ramas de trabajo (vacías de
commits, con el diff acumulado previo intacto) y este manifiesto.

## Resumen ejecutivo — por qué bloqueado

El sub-plan asume tres cosas que sostienen su decisión central ("GLOSA desaparece porque
el DGO ya asume esa función", "Shipper y Modulación son solo revivir un step file muerto"):

1. **Que el DGO (SP-05) ya bloquea el inicio de un despacho sin glosar.** Falso: el
   bloqueo (`assertInvoicesGlossedOk`) solo se invoca (a) en el endpoint de solo-lectura
   `GET /reference-documents/reference/:id/can-start-operation`, consumido por el wizard
   de creación (`StepDgoSelection.tsx`), y (b) dentro de `operations.service.ts` `create()`,
   y **solo si la operación se crea con `selectedDgoIds.length > 0`**. `startDispatchWorkflow()`
   —el método que arranca/reinicia un despacho— no llama a esta validación en ningún punto.
   Si SP-16 retira el paso GLOSA del stepper asumiendo que "ya está cubierto", una operación
   creada sin DGOs (o creada antes de SP-05) podría iniciar despacho sin haber pasado nunca
   por un chequeo de glosa.
2. **Que "revivir" `StepShipper.tsx`/`StepModulacion.tsx` es un ejercicio mecánico de
   reconectar un componente huérfano.** Falso: el backend tiene **dos sistemas de
   despacho paralelos y desconectados**:
   - El sistema **vivo** (Temporal): enum `StepName` en `schema.prisma` (8 valores exactos,
     igual a `DISPATCH_STEPS` del front), tabla `DispatchStep`, `dispatch.workflow.ts`
     (`ORDERED_STEPS`). Este es el que alimenta `dispatchStatus.currentStep` que lee
     `OperationStepsTabs.tsx`. Su rama `MODULACION` es un TODO real sin lógica
     (`// TODO: Implementar lógica de modulación`).
   - Un sistema **legacy, paralelo** basado en el enum `DispatchStepId` (`dispatch.dto.ts`,
     11 valores: incluye `PEDIMENTO`, `VUCEM`, `SHIPPER` además de los 8 del sistema vivo),
     persistido como JSON blob en `operation.operationMetadata.dispatch` (no en la tabla
     `DispatchStep`), con su propio `operation-dispatch.service.ts`
     (`completeModulacion()`, `completeShipper()` — ambos con lógica real, pero
     desconectados del workflow de Temporal).
   - El front's `useOperationDispatch.ts` (`completeModulacion`) ya llama a
     `PATCH /operations/:id/dispatch/modulacion`, que enruta al sistema **legacy**, no al
     Temporal. Es decir: "implementar Modulación" reviviendo el step muerto conectaría la UI
     a un backend que no actualiza `dispatchStatus.currentStep` (el que gobierna qué tab está
     activa/habilitada) — el resultado visible sería una tab que "completa" sin que el
     stepper avance.
   - `StepShipper.tsx` (el componente muerto) lee `dispatchStatus?.dispatch?.steps?.shipper`,
     una forma que no existe en el tipo actual de `OperationDispatchMetadata.steps` del
     sistema vivo — reforzando que fue escrito contra el sistema legacy, no el actual.
   - **No existe ninguna lógica de umbral "$2,500 USD"** en ningún repo, ni en el sistema
     vivo ni en el legacy. "Shipper condicional (>2,500 USD)" es una funcionalidad nueva de
     cero, no una reconexión.
3. **Que la duplicación backend es solo `dispatch/start` + `dispatch/status` entre dos
   controllers.** Es peor: `DispatchController` duplica `GET :id/dispatch/status` **dentro
   de sí mismo** (dos métodos — `getDispatchStatusNew` y `getDispatchStatus` — mapeados a
   la misma ruta), además de colisionar con `OperationsController` en `dispatch/start`.
   Decidir "cuál controller vive" no es trivial: `DispatchController` es el que expone las
   rutas realmente usadas por casi todos los steps del front (`coves`, `validacion/caarem`,
   `pedimento/glosa`, `regenerate-pedimento`, `modulacion`, `resume`, `abort`), mientras que
   `OperationsController.startDispatchWorkflow` (el que gana la colisión de `dispatch/start`
   por orden de registro) pasa por lógica distinta (chequeo `IN_PROGRESS`) que
   `DispatchController.dispatch/start` no tiene.

Además, dos correcciones menores de ubicación en el D1: la lógica de arranque de despacho
NO vive en `ui/OperationClient.tsx` (ese archivo es la vista de listado/tabla de
operaciones, sin lógica de despacho) sino en `app/(customerPortal)/customs-operation/[id]/page.tsx`;
y los endpoints `/dispatch/pedimento/glosa` y `/dispatch/regenerate-pedimento` **no están
huérfanos** — los sigue llamando activamente `StepPedimento.tsx` (que exporta el componente
`StepGlosa` que renderiza hoy el tab "glosa").

## Por qué esto bloquea (no es solo "más trabajo del esperado")

El sub-plan pide, en orden: (a) retirar GLOSA confiando en que DGO ya lo cubre — premisa
parcialmente falsa con gap de seguridad real; (b) añadir Shipper y Modulación "reviviendo"
archivos muertos — en realidad requiere decidir primero cuál de los dos sistemas de
despacho backend es el autoritativo, o unificarlos, antes de que "añadir un paso" tenga
sentido; y (c) resolver duplicación de controllers — decisión de arquitectura con impacto
en rutas ya usadas por producción. Implementar las tareas tal como están escritas
significaría: o bien remover GLOSA dejando un hueco de validación real, o bien construir
Shipper/Modulación contra un backend legacy que no mueve el estado que el stepper
efectivamente lee — ambas son "premisas falsas" en el sentido de la regla operativa
(equivalente al caso SP-06 antes de su replanteo), no ajustes menores que se puedan resolver
sobre la marcha sin decisión de producto/arquitectura.

## Decisiones que hacen falta antes de continuar (para el replanteo del sub-plan)

1. **Gate de glosa en dispatch start:** ¿se debe añadir `assertInvoicesGlossedOk` (u
   equivalente) dentro de `startDispatchWorkflow()`/`DispatchController.start` como defensa
   en profundidad, ANTES de poder retirar el tab GLOSA del stepper con seguridad? ¿O el
   producto acepta el gap porque en la práctica todas las operaciones pasan por el wizard
   con DGO?
2. **Unificación de los dos sistemas de despacho:** ¿se migra `Shipper`/`Modulación`
   (y cualquier otro paso legacy) al sistema Temporal/`DispatchStep`, se retira el sistema
   legacy (`DispatchStepId`, JSON blob, `operation-dispatch.service.ts`), o conviven a
   propósito? Esto determina si "Modulación" se implementa escribiendo lógica real dentro
   de `dispatch.workflow.ts` (rama `MODULACION`) + persistiendo en `DispatchStep`, en vez de
   reconectar el endpoint legacy `PATCH .../dispatch/modulacion`.
3. **Umbral Shipper (>2,500 USD):** de dónde sale el monto a comparar (¿valor de factura?
   ¿de la operación?), y contra qué campo — no existe hoy en ningún lado; es una regla de
   negocio nueva a especificar, no a "revivir".
4. **Qué controller de despacho vive:** `DispatchController` parece el candidato natural
   (concentra casi todas las rutas realmente usadas), pero su duplicado interno de
   `dispatch/status` y su colisión con `OperationsController.dispatch/start` necesitan
   decisión explícita de cuál lógica de negocio prevalece (el chequeo `IN_PROGRESS` de
   `OperationsController` no debe perderse silenciosamente).

## Estado de las ramas
- `carmi-digital@refactor/customs-operation-sp16` — creada desde `refactor/customs-operation-sp15`,
  sin commits nuevos, diff acumulado previo intacto.
- `carmi-odin-api-v2@refactor/customs-operation-sp16` — creada desde `refactor/customs-operation-sp13`
  (rama base real del repo backend al momento de iniciar este sub-plan), sin commits nuevos,
  diff acumulado previo intacto.

## Siguiente paso sugerido
Volver a `/plan` (o una sesión de replanteo dirigida, como se hizo con SP-06) para que un
humano resuelva las 4 decisiones de arriba y quede un D1 corregido con tareas ejecutables
sin ambigüedad de arquitectura, antes de reabrir `/implementa` sobre este sub-plan.

---

## 2026-07-12 — Segundo intento de `/implementa`, sobre el D1 rediseñado

**Estado: 🟡 PARCIALMENTE COMPLETADO.** El D1 rediseñado (mismo documento, sección "D1
corregido") resolvió las 4 decisiones pendientes de arriba con buena investigación de
código real. Se implementó todo lo que el D1 rediseñado describía con precisión
verificable. Se encontró **una premisa adicional, no cubierta por el D1 rediseñado**, que
bloquea solo la tarea de "purga backend legacy" (no las demás) — documentada abajo con
recomendación de un sub-plan dedicado en vez de forzarla.

Rama de trabajo reusada en ambos repos: `refactor/customs-operation-sp16` (la del intento
bloqueado, vacía de cambios propios propios, con el diff acumulado de sp13/sp18 intacto —
confirmado antes de tocar código).

### Hallazgo nuevo: la "purga backend" (Pasos, ítem 5) es más entrelazada de lo que el D1
rediseñado asumía

El D1 rediseñado (sección B) afirma que solo 3 métodos son legacy y purgables:
`completeModulacion()`, `completeShipper()`, `getDispatchStatus()`. Una investigación
dedicada (subagente, lectura completa de `operation-dispatch.service.ts` y
`dispatch.dto.ts`) encontró que el blob legacy (`OperationDispatchMetadata`,
`DispatchStepId`, `createInitialDispatch()`) está más entrelazado con rutas HTTP **hoy
vivas** de lo que el D1 asumía:

- `getDispatchStatus()` (el método, no la ruta — la ruta HTTP ya está sombreada por
  `getDispatchStatusNew` dentro del mismo controller) sigue siendo invocado internamente
  por `regeneratePedimento()`, que sí es una ruta viva (`POST :id/dispatch/regenerate-pedimento`,
  usada por `StepPedimento.tsx`/`StepGlosa`). Eliminar `getDispatchStatus()` rompe esa ruta
  sin antes reconstruir su cálculo de progreso sobre la tabla `DispatchStep` real.
- `completeModulacion()` y `executeStepPublic()`/`executeStep()` son rutas HTTP **en vivo**
  (`PATCH :id/dispatch/modulacion`, `POST :id/dispatch/steps/*`) construidas enteramente
  sobre el blob legacy, y **no están mencionadas como tal en el D1 rediseñado** (que solo
  nombra `completeModulacion` entre las tres a eliminar, sin notar que
  `executeStepPublic`/`executeStep` son un cuarto camino vivo con la misma dependencia).
  Llaman internamente a los mismos `completeEDocuments`/`completeCove`/`completeManifestacion`/
  `completeValidacion` que usa Temporal, envueltos en bookkeeping del blob.
- `confirmarValidacionCaarem()` (ruta `POST :id/dispatch/validacion/webhook`) escribe
  **solo** el blob legacy y no toca `DispatchStep` ni señala a Temporal — a diferencia del
  webhook real de Bifrost (`webhook.controller.ts`). Ningún código del repo construye una
  URL hacia esta ruta, lo que sugiere que, si está viva, es porque Bifrost mismo la tiene
  configurada como destino de webhook **fuera de este repo** — algo que un grep no puede
  confirmar ni descartar. Borrarla a ciegas arriesga romper silenciosamente una integración
  externa real, exactamente el riesgo que este sub-plan más quería evitar.
- `generatePedimento()` (usado por `regeneratePedimento`) persiste su único registro de
  "pedimento generado" dentro del blob (`dispatch.steps.pedimento`) — no hay tabla real
  equivalente; purgar el blob sin antes migrar este dato rompe la única fuente de verdad de
  ese estado.

**Decisión tomada en esta sesión:** no forzar la purga de `getDispatchStatus`,
`completeModulacion`, `executeStepPublic`/`executeStep`, `confirmarValidacionCaarem`, ni los
tipos legacy de `dispatch.dto.ts` (`DispatchStepId`, `OperationDispatchMetadata`, `*Data`
legacy). Sí se completó la parte de la purga que **no** tenía esta entrelazación:
`DispatchController.startDispatch` (duplicado confirmado inalcanzable — pierde la colisión
de ruta contra `OperationsController::startDispatchWorkflow`, que además es la
implementación con mejores validaciones). Se recomienda un sub-plan dedicado y acotado
("purga legacy dispatch blob") que primero resuelva: (a) migrar el cálculo de
`regeneratePedimento`/`generatePedimento` a la tabla `DispatchStep` real, (b) confirmar con
quien administra la configuración de Bifrost si `POST :id/dispatch/validacion/webhook`
sigue siendo un destino real, y (c) decidir si `executeStepPublic`/`completeModulacion`
siguen teniendo un consumidor real en el front (grep del front no encontró llamadas activas
a `PATCH .../dispatch/modulacion` tras retirar `completeModulacion` de
`useOperationDispatch.ts` en esta sesión — pero `executeStepPublic`'s rutas
`/dispatch/steps/*` no se tocaron ni se investigó su consumidor front a fondo, al ser
explícitamente fuera del alcance de este intento).

### Qué sí se implementó

**Backend (`carmi-odin-api-v2`, rama `refactor/customs-operation-sp16`):**

- `prisma/schema.prisma`: `StepName.SHIPPER` añadido al enum, entre `PAGO` y `DODA`.
  Migración generada con `npx prisma migrate dev --name add-shipper-step-name`
  (`20260712093255_add_shipper_step_name`) — `prisma migrate status` confirmó estado limpio
  antes de generar.
- `src/operations/constants/dispatch-steps.constant.ts`: `DISPATCH_STEPS_CONFIG.SHIPPER =
  { order: 7, type: 'INTERNAL' }`, reordenados `DODA` (8) y `MODULACION` (9). Nuevas
  constantes exportadas: `SHIPPER_LIQUIDACION_EFECTIVO_THRESHOLD_USD` (2500) y
  `UNCONDITIONAL_ORDERED_STEPS` (todos los steps salvo SHIPPER, para la creación inicial).
- `src/operations/services/operations.service.ts::startDispatchWorkflow()`: ahora crea
  `DispatchStep`s a partir de `UNCONDITIONAL_ORDERED_STEPS` (8, sin SHIPPER) en vez de
  `Object.entries(DISPATCH_STEPS_CONFIG)` completo — SHIPPER ya no se crea al arrancar.
- `src/temporal/activities/dispatch.activities.ts`: tres activities nuevas —
  `evaluateShipperStep` (resuelve el `Pedimento` vinculado vía `OperationPedimento`, compara
  `liquidacionEfectivo > 2500`), `createShipperStep` (crea el `DispatchStep` idempotente),
  `processShipperStep` (INTERNAL, sin Bifrost — ver desviación abajo).
- `src/temporal/workflows/dispatch.workflow.ts`: dentro del loop sobre `ORDERED_STEPS`,
  al llegar a `SHIPPER` (que por su `order` cae exactamente entre PAGO y DODA) evalúa la
  condición y, si no aplica, hace `continue` (no crea el step, no cuenta para el progreso);
  si aplica, crea el `DispatchStep` dinámicamente ahí mismo y lo procesa como INTERNAL vía
  `processShipperStep`. También se limpió una rama muerta (`else if (stepName ===
  'MODULACION')` con un `// TODO` sin resolver dentro del branch INTERNAL — nunca se
  ejecutaba porque MODULACION es EXTERNAL en `DISPATCH_STEPS_CONFIG`; se retiró en vez de
  dejar el TODO, por la regla de no dejar TODOs sin resolver).
- `src/operations/controllers/dispatch.controller.ts`: eliminado `startDispatch`
  (`POST :id/dispatch/start`, duplicado inalcanzable) y sus imports muertos (`StartDispatchDto`,
  `StepName`, `DISPATCH_STEPS_CONFIG`, `DispatchStepConfig`).
- `src/operations/controllers/dispatch.controller.spec.ts`: eliminado el test de
  `startDispatch` y su mock, correspondiente al método borrado.
- Verificación: `npx tsc --noEmit -p .` limpio (mismos errores preexistentes no
  relacionados en specs de otros módulos, confirmados por diff); `npx eslint` limpio (solo
  2 warnings preexistentes de variables no usadas, no tocadas por este cambio);
  `npx jest src/operations/services/operation-dispatch.service.spec.ts
  src/operations/controllers/dispatch.controller.spec.ts
  src/operations/services/operations.service.spec.ts` → 3 suites, 9 tests, todos verdes.

**Desviación documentada — `processShipperStep` sin regla de negocio explícita:** el D1
rediseñado dice "tipo INTERNAL... con la lógica real de asignación que corresponda" pero no
especifica ninguna regla de negocio para SHIPPER más allá del umbral que decide su
creación. No existe en ningún repo (ni legacy ni vivo) una lógica de "asignación de
shipper" preexistente para reutilizar. Se implementó `processShipperStep` como un
checkpoint de auditoría/visibilidad que se completa automáticamente (igual que el `GLOSA`
completa automáticamente si no hay discrepancias) — no lanza error, no requiere
intervención. Si el negocio efectivamente necesita una acción manual (p. ej. capturar un
shipper/transportista asignado), esto requiere una decisión de producto y probablemente un
endpoint nuevo — no estaba en el alcance de tareas de este sub-plan y no se inventó.

**Front (`carmi-digital`, rama `refactor/customs-operation-sp16`):**

- `types/operation-dispatch.ts`: `StepName`/`StepId` incluyen `SHIPPER`/`shipper`. `glosa`
  se conserva en `StepId`/`STEP_LABELS` (no se elimina el tipo) porque sigue siendo un gate
  real cuyo estado hay que poder leer de `dispatchSteps` para el panel inline — solo se
  retiró de `DISPATCH_STEPS` (el array que alimenta los tabs navegables). `shipper` se
  añadió a `DISPATCH_STEPS` entre `pago` y `doda`.
- `app/(customerPortal)/customs-operation/components/OperationStepsTabs.tsx`: nuevo cálculo
  `visibleSteps` (filtra `shipper` de `DISPATCH_STEPS` salvo que `dispatchStatus.dispatchSteps`
  tenga un registro real con `stepName === 'SHIPPER'`) usado en todos los índices/conteos/
  navegación en vez de la constante estática. Nuevo panel `showGlosaGate`: si el
  `DispatchStep` de GLOSA está `IN_PROGRESS` (spinner "Validando pedimento…") o `FAILED`
  (reusa el componente completo `StepGlosa` de `StepPedimento.tsx`, con su
  `StepErrorAlert` de retry/force-continue ya existente), se muestra entre los tabs
  Manifestación/Validación sin ser un tab seleccionable. `renderStepContent()` ahora tiene
  casos reales para `shipper` (nuevo `StepShipper.tsx`) y `modulacion` (nuevo
  `StepModulacionStatus.tsx`) en vez de `null`. El gate de "Siguiente" que antes miraba
  `activeStep === 'glosa'` ahora mira `activeStep === 'manifestacion' && (isGlosaFailed ||
  !pedimentoGlosaCanProceed)`, ya que `glosa` nunca vuelve a ser `activeStep`.
- `app/(customerPortal)/customs-operation/[id]/page.tsx`: nuevo helper `toVisibleStep()` que
  mapea `'glosa' → 'manifestacion'` en los tres puntos donde el `currentStep` del backend se
  usaba para fijar `activeStep` (incluida la inicialización) — evita que el front intente
  activar un tab que ya no existe cuando el workflow está corriendo GLOSA internamente.
- `app/(customerPortal)/customs-operation/components/steps/StepShipper.tsx`: reescrito
  desde cero (mismo archivo, contenido 100% nuevo) contra `dispatchStatus.dispatchSteps`
  real — ya no lee `dispatchStatus?.dispatch?.steps?.shipper` (forma legacy inexistente en
  producción). Solo lectura; usa `StepErrorAlert` con `stepType="INTERNAL"` para el caso
  `FAILED`.
- `app/(customerPortal)/customs-operation/components/steps/StepModulacionStatus.tsx`
  (nuevo archivo): solo lectura del estado real del `DispatchStep` MODULACION
  (`AWAITING_BIFROST`/`FAILED`/`COMPLETED`), usando `StepErrorAlert` con
  `stepType="EXTERNAL"`.
- `hooks/useOperationDispatch.ts`: eliminado `completeModulacion` (llamaba al `PATCH`
  legacy) y su import ahora-innecesario de `DispatchResult`. `VALID_STEP_IDS`/`STEP_LABELS`/
  `stepOrder` incluyen `shipper`. `totalSteps` dejó de estar hardcodeado en `8` en las dos
  funciones que lo usaban (`calculateProgress` y el mapeo de `fetchDispatchStatus`) — ahora
  se calcula como `dispatchSteps.filter(s => s.stepName !== 'GLOSA').length`, reflejando 7 u
  8 según si SHIPPER aplica para esa operación.
- Archivos eliminados (`git rm`, confirmados sin importadores antes de borrar):
  `steps/StepModulacion.tsx`, `steps/StepGlosa.tsx`, `steps/StepGlosa.example.tsx`,
  `steps/StepVucem.tsx`, `steps/StepPrevalidacion.tsx`,
  `app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx` (el duplicado sin
  importadores — se confirmó que `[id]/dispatch/page.tsx` importa el otro,
  `components/dispatch/DispatchMonitor.tsx`, que se conserva intacto).
- Verificación: `npx tsc --noEmit -p tsconfig.json` sobre el proyecto completo → 0 errores.
  `npx eslint` sobre los 6 archivos tocados/nuevos → 0 errores, solo warnings preexistentes
  de estilo (`no-explicit-any`) más un warning nuevo de `FileCheck` sin usar, corregido en
  el mismo cambio. No se encontraron tests (`*.spec.ts`/`*.test.ts`) que referencien
  `useOperationDispatch`, `OperationStepsTabs` ni `DispatchMonitor` — no había specs de
  front que actualizar para este alcance.

**Desviación documentada — FORCE_CONTINUE en MODULACION:** el D1 rediseñado (sección E)
sugiere exponer una acción de "forzar resultado" vía `resumeStepSignal`/`FORCE_CONTINUE`
para MODULACION "si el negocio lo exige". Se verificó en `dispatch.workflow.ts` que
`FORCE_CONTINUE` está **explícitamente prohibido para steps EXTERNAL** (el branch
`else` del workflow lo marca `FAILED` y termina el workflow si se intenta) — y
`operations.service.ts::resumeDispatchStep()` además restringe `FORCE_CONTINUE` a
`stepName === 'GLOSA'` únicamente, a nivel de validación HTTP. El componente ya existente
`StepErrorAlert.tsx` (reusado tal cual, sin tocar) ya sabía esto de antes: solo muestra el
botón de "Forzar Continuación" cuando `stepType === 'INTERNAL'`, y para `EXTERNAL` muestra
una nota explicando por qué no está disponible. `StepModulacionStatus.tsx` usa
`stepType="EXTERNAL"`, por lo que MODULACION queda correctamente limitado a "Reintentar"
(RETRY) — no se intentó forzar una funcionalidad que el propio backend ya impide.

### Verificación end-to-end (Playwright)

**No ejecutada.** Este cliente no tiene `dev_url` configurado en
`config/clientes.json` y no se levantó un entorno para esta sesión (ver nota operativa del
cliente). Solo se corrieron las puertas estáticas (typecheck/lint/tests unitarios) en
ambos repos, documentadas arriba. Queda pendiente validar visualmente: (a) 8 tabs con
Shipper visible para un pedimento con `liquidacionEfectivo > 2500`, 7 tabs sin él para uno
menor; (b) el panel de GLOSA aparece/desaparece correctamente sin romper la navegación;
(c) el webhook de Bifrost sigue resolviendo steps `AWAITING_BIFROST` — no se tocó
`webhook.controller.ts`, `BifrostClientService` ni `bifrost.health.ts` en ningún momento de
esta sesión, por lo que no se espera regresión, pero no se verificó en runtime.

### Archivos tocados en esta sesión

Backend: `prisma/schema.prisma`, `prisma/migrations/20260712093255_add_shipper_step_name/`,
`src/operations/constants/dispatch-steps.constant.ts`,
`src/operations/services/operations.service.ts`,
`src/temporal/activities/dispatch.activities.ts`,
`src/temporal/workflows/dispatch.workflow.ts`,
`src/operations/controllers/dispatch.controller.ts`,
`src/operations/controllers/dispatch.controller.spec.ts`.

Front: `types/operation-dispatch.ts`, `hooks/useOperationDispatch.ts`,
`app/(customerPortal)/customs-operation/components/OperationStepsTabs.tsx`,
`app/(customerPortal)/customs-operation/[id]/page.tsx`,
`app/(customerPortal)/customs-operation/components/steps/StepShipper.tsx` (reescrito),
`app/(customerPortal)/customs-operation/components/steps/StepModulacionStatus.tsx` (nuevo);
eliminados: `steps/StepModulacion.tsx`, `steps/StepGlosa.tsx`, `steps/StepGlosa.example.tsx`,
`steps/StepVucem.tsx`, `steps/StepPrevalidacion.tsx`,
`app/(customerPortal)/customs-operation/components/DispatchMonitor.tsx`.

### Siguiente paso sugerido

1. Un sub-plan corto y acotado ("purga legacy dispatch blob") que resuelva las 3 preguntas
   de la sección "Hallazgo nuevo" arriba antes de borrar `getDispatchStatus`,
   `completeModulacion`, `executeStepPublic/executeStep`, `confirmarValidacionCaarem` y los
   tipos legacy de `dispatch.dto.ts`.
2. Verificación end-to-end con Playwright cuando haya `dev_url`/entorno disponible para
   este cliente, cubriendo los 3 puntos de la sección "Verificación end-to-end" arriba.
