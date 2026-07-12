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
