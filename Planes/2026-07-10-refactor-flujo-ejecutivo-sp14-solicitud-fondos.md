# Sub-plan SP-14: Solicitud de Fondos (3 categorías)

Parte de [[2026-07-10-refactor-flujo-ejecutivo]]. Brecha grande (D1 vs propuesta).

## Contexto

Pantalla #23 del [[Inventario_Pantallas_v3]] (🔵 nueva). Origen:
[[2026-07-07 - Revision solicitud de fondos]] (M10, con Miguel) y
[[2026-06-30 - Prueba operación real - glosa, pago y DODA]] (esquemas de pago).
**Decisión de la entrevista:** 3 categorías con cálculo automático, **SIN** validación
de tesorería contra banco ni URL pública al cliente (esas → fase posterior).

## D1 — punto de partida
- **Reusa / refactoriza:** `references/components/tabs/ReferenceOperations.tsx:198-215`
  (`handleSolicitarFondos`: **POST vacío** a `/operations/:id/solicitar-fondos` +
  `alert()` nativo; botón en :477). Backend `operations.controller.ts:202`
  `@Post(':id/solicitar-fondos')`. Reemplazar el `alert()` por toast.
- **Referencia (otro flujo, no reusar tal cual):** `app/(customerPortal)/finance/components/Funds-stepper.tsx`
  + `app/actions/finance/get-SolicitudFondos.ts` (lado finanzas/tesorería) — sirve de
  referencia de datos, es un flujo distinto.
- **Crea:** el módulo de solicitud de fondos, **centralizado dentro del módulo de
  operación/referencia**, con 3 categorías integradas y recalculadas/totalizadas
  automáticamente:
  1. **Gastos comprobados** (ej. lavado de contenedores, maniobras — engancha con
     Retorno de Vacío, SP-11).
  2. **Impuestos de pedimento** — desde la sección **"efectivo"** del cuadro de
     liquidación del pedimento.
  3. **Honorarios** — generados automáticamente por la **configuración de tarifas**
     (por peso, porcentaje o criterio).
  - **Solicitud complementaria / pagos parciales** (ajuste de montos).

## Fuera de alcance (fase posterior)
- **Validación de tesorería contra banco** antes de autorizar el pago.
- **Notificación al cliente por URL pública** para adjuntar comprobante de depósito.
- **Prevención de cobro duplicado** de servicios por-operación en operaciones
  multi-pedimento (cruce de referencias entre procesos).

## Pasos
- [ ] Reemplazar el POST vacío + alert por el módulo real (toast en vez de alert).
- [ ] Categoría impuestos: extraer del cuadro de liquidación ("efectivo").
- [ ] Categoría honorarios: cálculo automático por configuración de tarifas.
- [ ] Categoría gastos comprobados: captura + integración.
- [ ] Recalcular y totalizar automáticamente las 3; solicitud complementaria/parcial.

## Riesgos y side effects
- Depende de SP-05 (DGO/pedimento) y SP-06 (impuestos calculados) para los montos.
- Su tamaño real puede afectar cuántas pantallas necesita el flujo (el Documento lo
  señaló) — reevaluar el inventario si crece.

## Criterios de verificación
- Gate estático verde. Playwright: generar una solicitud con las 3 categorías
  recalculadas automáticamente y una complementaria; feedback por toast; sin errores
  de consola.

## Estado
📋 Por implementar.
