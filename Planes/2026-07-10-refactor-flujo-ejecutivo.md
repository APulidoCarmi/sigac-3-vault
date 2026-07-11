# Plan paraguas: Refactor del flujo del Ejecutivo de Cliente (importación/exportación) — SIGAC 3

> Plan paraguas según [[Planes paraguas y replaneo]]: este archivo lleva el
> contexto y las decisiones de toda la iniciativa y el índice de sub-planes
> hijos (cada uno es un plan normal, apto para una sesión limpia de
> `/implementa`, con sus propios criterios de verificación). El paraguas **no
> se implementa**: se actualiza al cerrar cada hijo.

## Contexto

Refactor del flujo del **ejecutivo de cliente** (importación/exportación) en
SIGAC 3. Encuadre de las 3 fuentes según [[_Como leer este contexto]]:

- **Reuniones/** = cómo trabaja HOY el ejecutivo en SIGAC 2 (requisitos / dolores).
- **Código de `sigac-3` (D1)** = estado ACTUAL, punto de partida (as-is).
- **Arquitectura/** ([[Documento_Entendimiento_SIGAC3_Ejecutivo_v2]],
  [[Inventario_Pantallas_v3]]) = destino (to-be). Lo marcado NO decidido
  (M8/M9/M10) se resolvió en la entrevista de este plan (ver abajo).

**El refactor = llevar D1 (as-is) al objetivo de Arquitectura (to-be),
resolviendo los dolores de las reuniones.**

Área de código (dentro de alcance): front
`carmi-digital/app/(customerPortal)/references/**` (dominio Referencia) +
`carmi-digital/app/(customerPortal)/customs-operation/**` (despacho) +
`carmi-digital/components/{shipment-creation,movimientos}/**`, respaldado por
`carmi-odin-api-v2` (controllers `references`, `operations`, `dispatch`,
`shipments`, `osd`, `reference-documents`, `pedimentos`). Orientación: graphify
sobre el repo + inventario de código D1 levantado para este plan (rutas citadas
en cada sub-plan).

### Inventario de código D1 — hallazgos que corrigen al Documento de Entendimiento

Levantados con graphify + verificación en fuente; condicionan varios sub-planes:

1. **Wizard de referencia** vive en `references/createReference/page.tsx` (4
   pasos). La extracción es `POST /api/workflow/process` (**v1**), NO
   `/api/workflow/v3/process` como decía el Documento.
2. **Detalle de referencia** (`references/[id]/page.tsx` →
   `references/components/ReferenceTabs.tsx`) usa **Tabs de shadcn**, no menú
   lateral. Tabs reales: `overview`(Resumen), `inventory`(Mercancía),
   `shipments`(Movimientos), `globalExpenses`, `legalConfiguration`,
   `instructions`, `documents`, `appointments`, `warehouse`, `operations`,
   `timeline`. (El Documento no mencionaba instructions/appointments/warehouse.)
3. **Dos wizards de operación distintos.** El Documento describía el de 6 pasos
   `components/operations/CreateOperationModal.tsx`, que está en
   `components/operations` = **FUERA de alcance**. El wizard IN-SCOPE es el de
   **4 pasos** `customs-operation/createOperation/page.tsx` (su cálculo de
   impuestos está comentado). **Decisión tomada:** el refactor de #21 opera
   sobre el de 4 pasos (ver Decisiones).
4. **Despacho** (`customs-operation/[id]/page.tsx` → `ui/OperationClient.tsx` →
   `components/OperationStepsTabs.tsx`) es un stepper de **8 pasos**
   (`DISPATCH_STEPS` en `types/operation-dispatch.ts`). El paso GLOSA es
   `steps/StepPedimento.tsx` (importado como `StepGlosa`). Existen **archivos
   muertos reutilizables**: `steps/StepShipper.tsx`, `steps/StepModulacion.tsx`,
   `steps/StepVucem.tsx`, `steps/StepPrevalidacion.tsx`, `steps/StepGlosa.tsx`.
   Hay **duplicación de DispatchMonitor** (segunda ruta `[id]/dispatch/page.tsx`)
   y de rutas backend (`operations.controller` y `dispatch.controller` ambos con
   base `operations`).
5. **B1 / virtual NO existe** en los DTOs de odin: el tipo de tráfico es
   `trafficTypeId → TransportMode`, no un enum con B1. Confirma que la Brecha #5
   necesita un **spike de esquema** que abarque `carmi-db-api` (prisma/entidades),
   no solo odin.
6. **Código muerto confirmado** (a limpiar en los sub-planes que lo tocan):
   `references/reference-stepper/index.tsx`, `tabs/ReferencePedimento.tsx`,
   `drawers/pedimento/PedimentoProformaDocument.tsx`, `PedimentoHbsDocument.tsx`.

### Origen (trazabilidad)

Reuniones de discovery SIGAC 2: [[2026-06-25 - Daily Scrum - flujo de importación y checklist dinámico]],
[[2026-06-26 - Prueba operación real - flujo ejecutivo y consolidación de documentos]],
[[2026-06-30 - Daily Scrum - e-documents y COVE]],
[[2026-06-30 - Prueba operación real - glosa, pago y DODA]],
[[2026-07-01 - Marítimo Sigac 3.0]], [[2026-07-01 - Aéreo Sigac 3.0 - operaciones virtuales (B1)]],
[[2026-07-02 - Aéreo Sigac 3.0]], [[2026-07-08 - Aéreo SIGAC 3.0]].
Diseño SIGAC 3: [[2026-03-19 - Análisis de Referencia y Movimientos]] (M0),
[[2026-04-28 - Prueba operación real desde ticket a facturación]] (M0b),
[[2026-07-06 - Seguimiento de refactor]] (M8),
[[2026-07-07 - Revision de pantallas flujo operación]] (M9),
[[2026-07-07 - Revision solicitud de fondos]] (M10).

## Decisiones tomadas (entrevista de este plan)

1. **Columna vertebral = plano documental primero.** Orden: Referencia (wizard
   3 pasos + Listado + Detalle) → Expediente aduanero → DGO → wizard de
   Operación alineado al DGO. Porqué: el DGO es el cambio más profundo (modelo
   de 3 planos) y todo lo demás cuelga de él; se estabiliza el modelo antes de
   tocar movimientos especializados.
2. **Navegación del detalle: migrar a menú lateral (M9).** D1 usa Tabs. Es un
   cambio de *shell* transversal a todas las pantallas de detalle → se planea en
   su propio sub-plan y los demás cuelgan de él.
3. **Wizard de Operación (#21): refactorizar el de 4 pasos IN-SCOPE**
   (`customs-operation/createOperation`), con **Step 0 = "Seleccionar DGO(s)"**
   y **reactivar el cálculo de impuestos** (hoy comentado). El de 6 pasos
   (`components/operations`) queda fuera. Porqué: coherente con el framing
   (`operations` = fuera de alcance).
4. **Solicitud de Fondos (#23): 3 categorías con cálculo automático** (gastos
   comprobados, impuestos de pedimento, honorarios), **SIN** validación de
   tesorería contra banco ni URL pública al cliente (esas → fase posterior).
5. **Despacho (#25): validación completa de los 8 pasos** contra los meets;
   **el paso GLOSA desaparece** (stepper baja a 7, la glosa la asume el DGO a
   nivel referencia); **Shipper (>2,500 USD) = paso propio condicional** del
   stepper.
6. **B1 / virtual: spike de esquema de BD primero** (bloquea las pantallas B1);
   de ahí sale el diseño de soporte B1 + identificadores MS/IM/B1/IC + alertas
   de folios.
7. **Portal de Documentos del Cliente (#26): dentro del refactor**, después de
   Expediente + glosado (depende de ellos).
8. **Incrementables aéreo (punto abierto f#6): por guía + auto-suma** a nivel
   referencia/DGO — resuelve el dolor del Excel; impacta el modelo del DGO.
9. **Tab #12: nombre unificado "Recinto"**, contenido adaptado por tráfico
   (almacén / almacén fiscalizado / terminal).
10. **Escala: cientos/miles/día** → paginación/virtualización desde el diseño en
    listados, DGO por partida, agrupaciones y polling.

### Discrepancias entre fuentes a vigilar (no resueltas en silencio)

- **Orden del wizard de referencia:** M9 dice *datos básicos → subir a CEUS →
  auditoría*; [[Inventario_Pantallas_v3]] dice *Documentos → Doc. Aceptados →
  Datos Básicos*. → se decide en el sub-plan SP-02.
- **Línea de tiempo:** M9 la mueve *dentro de Resumen*; el Inventario #13 la
  deja como tab propio. → se decide en SP-03.
- **Campo "orden de compra"** y **dos sabores de creación (cliente/ejecutivo)**:
  solo aparecen en M9. → se incorporan en SP-02.
- **Zeus == CEUS:** mismo motor de extracción/glosa (variación de dictado). Se
  usa "Zeus/CEUS" indistintamente; unificar nomenclatura al implementar.
- **Modelo del DGO:** M9 dice "un DGO por factura"; el Documento (con tu
  aclaración posterior) fija "1 DGO = 1 pedimento, 1 por default, se separa". Se
  sigue el Documento como autoritativo. → detalle en SP-05.

## Fuera de alcance

- **Exportación Terrestre y Exportación Aéreo** (bloqueadas — falta sesión de
  discovery; pregunta de alcance #3 del Inventario). Exportación **Marítimo** sí
  entra (definida en el Documento).
- Todo lo llamado **`operations`** (`customerPortal/operations`,
  `dashboard/operations` = Control Tower de SIGAC 2, `components/operations`,
  `actions/operations`, módulos `operations` de las APIs).
- **Solicitud de Fondos:** validación de tesorería contra banco, notificación al
  cliente por URL pública para comprobante, y prevención de cobro duplicado
  cross-pedimento (fase posterior — ver SP-14).
- **Configuración de MOMP** (onboarding/perfil de cliente); sus reglas sí se
  consumen (solo lectura) en Expediente/DGO.
- **Ejecución física del Previo** (la hace almacén / trámite y despacho); el
  ejecutivo solicita y consulta resultado.
- Roles ajenos: almacén, clasificación, tesorería, trámite y despacho (solo
  lectura / consulta para el ejecutivo).

## Sub-planes hijos (índice por fases, en orden de dependencia)

Estado: 📋 por redactar · ✍️ redactado (listo para `/implementa`) · 🔨 en curso · ✅ cerrado.
Nota de agrupación (justificada, ver estándares de CLAUDE.local.md): las
pantallas de movimientos especializados por tráfico se agrupan en un sub-plan
por tráfico porque el propio [[Inventario_Pantallas_v3]] las trata como una sola
superficie de diseño (Fase 3 = "material de referencia... no 26 pantallas
nuevas"); cada sub-plan lista los # del inventario que cubre. Las pantallas
✅ absorbidas (#7 Mercancía, #9 Incrementables, #10 Identificadores) no tienen
sub-plan propio: viven dentro de SP-05 (DGO).

### Fase 0 — Spike (desbloquea B1)
- [[2026-07-10-refactor-flujo-ejecutivo-sp00-spike-b1-virtual]] — Spike esquema B1/virtual (Brecha #5). ✅ Ver [[SP-00 - Spike esquema B1-virtual - conclusiones]].

### Fase 1 — Plano documental (columna vertebral)
- [[2026-07-10-refactor-flujo-ejecutivo-sp03-detalle-shell-menu-lateral]] — Shell del detalle + menú lateral + Resumen (#4, #13). Transversal, va primero. ✅ Cerrado (2026-07-10).
- [[2026-07-10-refactor-flujo-ejecutivo-sp01-listado-referencias]] — Listado de Referencias + tabla clásica (#2). ✅ Cerrado (2026-07-10).
- [[2026-07-10-refactor-flujo-ejecutivo-sp02-wizard-crear-referencia]] — Wizard 3 pasos (#3). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp04-expediente-aduanero]] — Expediente + glosado Zeus/CEUS (#5). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp05-dgo-datos-glosados]] — DGO, fuente única de verdad (#6; absorbe #7/#9/#10). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]] — Wizard Operación 4 pasos → Step 0 DGO (#21). 📋

### Fase 2 — Movimientos y logística
- [[2026-07-10-refactor-flujo-ejecutivo-sp07-tab-movimientos-por-trafico]] — Tab Movimientos rediseño por tráfico + vínculo flexible DGO (#8; reusa #14/#15/#17). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp08-recinto]] — Recinto (solo lectura) por tráfico (#12). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp09-previo-osd]] — Previo tab + Solicitar Previo + OS&D consulta (#12b, #20b, #16). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp10-movimientos-aereo]] — Manifiestos + Revalidación + Asignación Transporte aéreo (#20, #18, #19). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp11-movimientos-maritimo]] — Revalidación/Retorno de Vacío/Recuperación/Toma/Carga/Ingreso + Generar Cita (#18, #19, #20c + export marítimo). 📋

### Fase 3 — Operación y despacho
- [[2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]] — Tab Operaciones como agrupador vía DGO (#11). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp13-solicitar-salida]] — Solicitar Salida mini-modal (#22). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp14-solicitud-fondos]] — Solicitud de Fondos 3 categorías (#23). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp15-proforma-pedimento]] — Ver Proforma/Pedimento + limpiar código muerto (#24). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp16-despacho-stepper]] — Stepper de despacho: validar 8 pasos, retirar GLOSA, Shipper propio (#25). 📋

### Fase 4 — Entrada y cliente
- [[2026-07-10-refactor-flujo-ejecutivo-sp17-bandeja-entrada]] — Bandeja de Entrada / Inbox (#1). 📋
- [[2026-07-10-refactor-flujo-ejecutivo-sp18-portal-documentos-cliente]] — Portal de Documentos del Cliente (#26). 📋

## Riesgos y side effects transversales

- **Cambio de shell (menú lateral)** toca todas las pantallas de detalle: hacerlo
  primero (SP-03) y que los demás cuelguen, o se re-trabaja UI varias veces.
- **DGO es el nodo crítico:** absorbe 3 tabs (Mercancía/Costos/Config Legal) y
  cambia el modelo de datos (partidas, incrementable por guía). Un error aquí se
  propaga a Operación (#21), Despacho (#25) y Solicitud de Fondos (#23).
- **Retirar el paso GLOSA** del stepper interactúa con `StepPedimento.tsx` y sus
  endpoints (`/dispatch/pedimento/glosa`, `/dispatch/regenerate-pedimento`) y con
  la deuda de duplicación (DispatchMonitor / rutas backend duplicadas).
- **Escala cientos/miles/día:** listados y DGO por partida deben paginar/virtualizar;
  vigilar los tres mecanismos de polling que pueden solaparse (1s+2s+3s).
- **Código de compañeros:** el listado de operaciones lo desarrolla Carlos
  (M8); coordinar SP-01/SP-12 con él. `components/operations` (fuera de alcance)
  comparte nombres con el flujo in-scope: no tocarlo por error.
- **Zeus/CEUS (extracción v1):** el wizard usa `/api/workflow/process` v1;
  confirmar qué versión debe usar el Expediente re-glosado antes de cablear SP-04.

## Criterios de verificación (globales)

- Puertas estáticas del proyecto (lint/typecheck/tests) verdes tras cada hijo
  (skill `/verify`; comandos a detectar en el repo).
- Cada sub-plan que toca front se valida con Playwright MCP revisando la consola
  del navegador en el flujo afectado (`/verify`).
- El paraguas se actualiza al cerrar cada hijo: marcar estado, anotar decisiones
  que afecten a los siguientes (replaneo sobre este archivo si algo se invalida,
  según [[Planes paraguas y replaneo]]).

## Convención de ramas de implementación (ejecución desatendida)

Decisión operativa tomada al arrancar la ejecución desatendida del plan completo
(2026-07-10), no especificada por el usuario — documentada aquí para que
cualquier agente que retome la ejecución en una sesión nueva la respete:

- **Rama por sub-plan, encadenada por repo**: cada sub-plan de código usa
  `refactor/customs-operation-spNN` (dos dígitos). Dentro de un mismo repo, la
  rama de un sub-plan **parte de la rama del sub-plan anterior que tocó ese
  mismo repo** (no de la rama base original), porque los sub-planes de Fase 1
  en adelante dependen unos de otros en el mismo código (p. ej. SP-01 vive
  dentro del shell que crea SP-03). Si un sub-plan es el primero en tocar un
  repo, su rama parte de la rama en la que estaba el repo al arrancar esta
  ejecución.
- **Ramas base por repo al arrancar (2026-07-10):** `carmi-digital` estaba en
  `test` (con `public/firebase-messaging-sw.js` modificado sin commitear,
  ajeno a este plan — no tocar); `carmi-odin-api-v2` estaba en `staging` (con
  `src/operations/services/operations.service.ts` modificado sin commitear,
  ajeno y además parte de `operations`, fuera de alcance — no tocar).
- **Nunca push.** Alcance solo `customs-operation` — nunca `operations`.
- Un sub-plan que NO toca un repo dado no genera rama en ese repo.

## Estado del paraguas

📋 Redactado — pendiente de detallar los sub-planes hijos y validar en sesión
limpia antes de `/implementa`.
