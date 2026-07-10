# Sub-plan SP-16: Stepper de Despacho — validar 8 pasos, retirar GLOSA, Shipper propio

Parte de [[2026-07-10-refactor-flujo-ejecutivo]].

## Contexto

Pantalla #25 del [[Inventario_Pantallas_v3]] (🟡 modificar). El despacho es un stepper
orquestado por Temporal. Origen: glosario "Despacho" y Brechas #1/#2/#3 del
[[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]],
[[2026-06-25 - Daily Scrum - flujo de importación y checklist dinámico]] (orden SIGAC 2)
y [[2026-06-30 - Prueba operación real - glosa, pago y DODA]]. **Decisiones de la
entrevista:** validación **completa** de los 8 pasos contra los meets; **GLOSA
desaparece** (→ 7 pasos; la asume el DGO a nivel referencia); **Shipper (>2,500 USD)
= paso propio condicional**.

## D1 — punto de partida
- **Reusa:** ruta `customs-operation/[id]/page.tsx` → `ui/OperationClient.tsx`
  (dispatch start :158) → `components/OperationStepsTabs.tsx` (renderiza `DISPATCH_STEPS`
  de `types/operation-dispatch.ts`, 8 pasos: edocuments → cove → manifestacion → glosa
  → validacion → pago → doda → modulacion). Steps en `customs-operation/components/steps/`:
  `StepEDocuments.tsx` (`POST /operations/:id/edocuments/send`), `StepCove.tsx`
  (`/dispatch/coves`), `StepManifestacion.tsx`, **`StepPedimento.tsx` = GLOSA**
  (`/dispatch/pedimento/glosa`, `/dispatch/regenerate-pedimento`), `StepValidacion.tsx`
  (`/dispatch/validacion/caarem`), `StepPago.tsx`, `StepDoda.tsx`. Hooks
  `useStartDispatch.ts`, `useOperationDispatch.ts`, `useDispatchStatus.ts`. `modulacion`
  hoy devuelve `null` (no implementado).
- **Refactoriza:**
  - **Retirar el paso GLOSA** (`StepPedimento`) — la glosa la asume el DGO (SP-05) a
    nivel referencia → stepper baja a **7 pasos**. Resolver los endpoints que quedan
    huérfanos (`/dispatch/pedimento/glosa`, `/dispatch/regenerate-pedimento`).
  - **Validar los 7/8 pasos restantes** contra los meets: orden, fusión, altas/bajas;
    confirmar si **MODULACION** (hoy null) equivale a "liberación" de SIGAC 2.
  - Resolver la **duplicación de DispatchMonitor**: `components/DispatchMonitor.tsx`
    solo lo usa la segunda ruta `customs-operation/[id]/dispatch/page.tsx` — decidir
    cuál ruta vive. Y la **duplicación backend**: `operations.controller.ts` y
    `dispatch.controller.ts` ambos con base `operations` y `dispatch/start`+`dispatch/status`
    duplicados.
- **Crea / revive:**
  - **Shipper (>2,500 USD) como paso propio condicional** — **revivir** el archivo
    muerto `steps/StepShipper.tsx` (hoy sin importador) en vez de crear de cero.
  - Evaluar **revivir `steps/StepModulacion.tsx`** (muerto) para implementar MODULACION.
- **Limpia (muertos no usados):** `steps/StepGlosa.tsx`, `StepGlosa.example.tsx`,
  `StepVucem.tsx`, `StepPrevalidacion.tsx` (tras decidir cuáles no se reviven).

## Fuera de alcance
- El motor Temporal de backend (solo la orquestación de pasos vista desde el front).

## Pasos
- [ ] Ejercicio de validación de los 8 pasos contra los meets (documentar decisiones).
- [ ] Retirar GLOSA (`StepPedimento`) y limpiar sus endpoints huérfanos → 7 pasos.
- [ ] Añadir Shipper condicional (revivir `StepShipper.tsx`).
- [ ] Implementar MODULACION (¿revivir `StepModulacion.tsx`? confirmar vs "liberación").
- [ ] Resolver duplicación de DispatchMonitor y de rutas backend.
- [ ] Eliminar los step files muertos no revividos.

## Riesgos y side effects
- **Depende de SP-05 (DGO)**: no retirar GLOSA hasta que el DGO asuma esa función.
- Tocar Temporal/orquestación: vigilar cancelar/reintentar/forzar por paso.

## Criterios de verificación
- Gate estático verde. Playwright: iniciar un despacho, ver 7 pasos (sin GLOSA), que
  Shipper aparezca solo cuando valor > 2,500 USD, y MODULACION renderice; sin errores
  de consola.

## Estado
📋 Por implementar.
