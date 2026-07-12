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
- [[2026-07-10-refactor-flujo-ejecutivo-sp03-detalle-shell-menu-lateral]] — Shell del detalle + menú lateral + Resumen (#4, #13). Transversal, va primero. ✅ Cerrado (2026-07-11, rehecho desde cero): nueva rama `refactor/customs-operation-sp03` en `carmi-digital` partiendo de `test`, diff sin commitear para revisión humana.
- [[2026-07-10-refactor-flujo-ejecutivo-sp01-listado-referencias]] — Listado de Referencias + tabla clásica (#2). ✅ Cerrado (2026-07-11), re-implementado desde cero tras el descarte de la versión anterior. Ambos repos, gate estático verde, Playwright pendiente de validación humana.
- [[2026-07-10-refactor-flujo-ejecutivo-sp02-wizard-crear-referencia]] — Wizard 3 pasos (#3). 🟡 Cierre parcial (2026-07-12): las dos decisiones de producto pendientes quedaron resueltas y validadas por el usuario (distinción cliente/ejecutivo confirma `IsInternalUser` sin cambios de código; `stepChecklistGlosa` y `stepInvoicesReference` eliminados por código muerto confirmado). Gate estático verde en ambos repos. Sigue 🟡 solo por el bloqueo de infraestructura en la migración Prisma del campo OC (historial de migraciones inconsistente en el entorno) — ver manifiesto.
- [[2026-07-10-refactor-flujo-ejecutivo-sp04-expediente-aduanero]] — Expediente + glosado Zeus/CEUS (#5). ✅ Cerrado (2026-07-12, implementado junto con SP-05 en el mismo subagente por su dependencia circular — ver manifiestos de ambos). Ramas `refactor/customs-operation-sp05` (digital y odin, encadenada desde `sp04`), diff sin commitear.
- [[2026-07-10-refactor-flujo-ejecutivo-sp05-dgo-datos-glosados]] — DGO, fuente única de verdad (#6; absorbe #7/#9/#10). ✅ Cerrado (2026-07-12). Reducciones de alcance documentadas en su manifiesto: filtrado por DGO en endpoints heredados de Mercancía/Gastos/Identificadores pendiente para N>1 DGO, indicador de consistencia vs Previo y MV electrónica fuera (dependen de SP-09), happy path Ángel/German sin correr (falta entorno con Playwright).
- [[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]] — Wizard Operación 4 pasos → Step 0 DGO (#21). ✅ Cerrado (2026-07-12), tercer paso tras el bloqueo inicial y el replanteo: Step 0 real (`StepDgoSelection.tsx`) construido desde cero con soporte multi-DGO/multi-referencia (régimen aduanero homogéneo validado en back y front), migración `dgoId` en `OperationPedimento` (CLI), `POST /operations` conecta cada pedimento a la referencia real de su DGO, bloqueo de edición de DGO ya vinculado, cálculo de impuestos (TaxEngine) cableado en el Step de Despacho, bloqueo de glosa (`can-start-operation`) antes de crear la operación, y limpieza del código huérfano (`PedimentoWizardContext`, `StepInventorySelection`, `reference-selection/`, `hooks/`). Ver manifiesto para limitaciones documentadas (prellenado por-grupo parcial, `next build` bloqueado por dependencias preexistentes no relacionadas).

### Fase 2 — Movimientos y logística
- [[2026-07-10-refactor-flujo-ejecutivo-sp07-tab-movimientos-por-trafico]] — Tab Movimientos rediseño por tráfico + vínculo flexible DGO (#8; reusa #14/#15/#17). ✅ Cerrado (2026-07-11): `ReferenceShipments.tsx` pasó a ser un router delgado por `reference.trafficType.code` (nuevo campo expuesto en `ReferenceDetail`) que reusa sin cambios el flujo terrestre ya existente (renombrado a `ReferenceShipmentsTerrestre.tsx`) y deja un placeholder explícito para aéreo/marítimo (SP-10/SP-11). Trazabilidad DGO↔movimiento nueva de punta a punta (no existía): backend resuelve `Shipment → InvoiceShipmentLink → Invoice.dgoId → Dgo` deduplicado en `GET /shipments/reference/:referenceId`, front pinta badges "DGO-N" por movimiento. Ambos repos, gate estático verde, Playwright pendiente de sesión humana.
- [[2026-07-10-refactor-flujo-ejecutivo-sp08-recinto]] — Recinto (solo lectura) por tráfico (#12). ✅ Cerrado (2026-07-11): `ReferenceWarehouse.tsx` adaptado in-place (sin router por archivos, a diferencia de SP-07, porque no hay sub-plan futuro que rehaga Recinto por tráfico) — nombre/copy por `reference.trafficType.code` (Almacén/Almacén Fiscalizado/Terminal) y remoción de toda acción de escritura (confirmar arribo/inspeccionar/despachar, crear solicitud, subir evidencia, crear registro), dejando el tab 100% consulta. Solo `carmi-digital` tocado (backend no diferencia por tráfico hoy — ver manifiesto). Gate estático verde, Playwright pendiente de sesión humana.
- [[2026-07-10-refactor-flujo-ejecutivo-sp09-previo-osd]] — Previo tab + Solicitar Previo + OS&D consulta (#12b, #20b, #16). ✅ Cerrado (2026-07-12): modelo `Previo` nuevo en odin (1 por Reference, destino calculado por tráfico) que cierra el stub `PREVIO_CONSISTENCY_VALIDATOR` que SP-04 había dejado listo para SP-09 en `GlosaEngineService`; `OSDReportForm.tsx` reescrito de formulario de captura a vista de consulta (`OsdReportView`), reusando el cliente/UI de `features/osd/**` ya existente en vez de duplicarlo. Nuevo tab "Previo" y modal "Solicitar Previo" en el detalle de referencia. Ambos repos, gate estático verde (505/505 suites odin, 88/88 digital), Playwright pendiente de sesión humana. Deuda documentada en su manifiesto: copy del botón OSD en `ReferenceShipmentsTerrestre.tsx` (SP-07) sin actualizar, tipo de cita "PREVIO" legacy del front sin reconciliar.
- [[2026-07-10-refactor-flujo-ejecutivo-sp10-movimientos-aereo]] — Manifiestos + Revalidación + Asignación Transporte aéreo (#20, #18, #19). ✅ Cerrado (2026-07-12): 3 dominios backend greenfield (`transport-assignment`, `air-revalidation`, `air-manifest`) siguiendo el patrón `previo`/`dgo`, con máquina de etapas estricta para el tablero de manifiestos (llega→ETA→confirma llegada→descarga→desconsolida→mesa de previo→listo para previo) y semáforo de transporte de solo lectura para el ejecutivo (lo actualiza "trámite y despacho", rol/sub-plan futuro). `ReferenceShipmentsAereo.tsx` reemplaza el placeholder de SP-07 para AEREO. Componente `TransportAssignmentPanel` construido path-neutral (`components/shipments/transport-assignment/**`) y documentado literal en su manifiesto para reuso directo de SP-11. Ambos repos, gate estático verde (odin 30/30 tests nuevos, digital tsc+eslint 0 errores), Playwright pendiente de sesión humana. Desviación documentada: selector de `shipmentId` manual en el tab (sin lista real de shipments aéreos cableada aún, fuera del alcance declarado).
- [[2026-07-10-refactor-flujo-ejecutivo-sp11-movimientos-maritimo]] — Revalidación/Retorno de Vacío/Recuperación/Toma/Carga/Ingreso + Generar Cita (#18, #19, #20c + export marítimo). ✅ Cerrado (2026-07-12): 5 dominios backend greenfield en `carmi-odin-api-v2` (`maritime-revalidation`, `maritime-guarantee-recovery` — 1:1 por Shipment, mismo shape que `air-revalidation`; `maritime-empty-return` — 1:1 por Shipment con máquina de etapas TRANSPORT_REQUESTED→FUMIGATION_WASH_IN_PROGRESS→FUMIGATION_WASH_DONE→FUNDS_REQUESTED→EMPTY_ACKNOWLEDGED, enganchando SP-14 vía un `fundsRequestId` de referencia libre sin FK formal; `maritime-loading` — N por Reference con etapas SCHEDULED→LOADING_IN_PROGRESS→LOADED→SEALED, mismo shape que `air-manifest`; `maritime-appointment` — #20c Generar Cita, N por Shipment, modelo nuevo deliberadamente NO acoplado al `Appointment` de warehouse). `TransportAssignmentPanel` (SP-10) reusado tal cual para Retorno de Vacío y Toma de Vacío, sin tocar backend. Front: `ReferenceShipmentsMaritimo.tsx` reemplaza el placeholder de SP-07 para MARITIMO, ramificado por `reference.serviceType` (IMPORT: Revalidación/Retorno de Vacío/Recuperación de Garantía; EXPORT: tablero de Carga de Mercancía a nivel Reference + Toma de Vacío/Ingreso de Mercancía por shipmentId). `ReferenceShipmentsTrafficPending.tsx` eliminado (quedó sin uso tras este cierre, ya no lo usaba ni AEREO desde SP-10). Ambos repos, gate estático verde (odin: 48/48 tests nuevos en 10 suites, `prisma migrate dev` limpio — migración `20260712062422_add_maritime_movements_domain`; digital: tsc 0 errores, eslint 0 errores/0 warnings en los 25 archivos nuevos/tocados). Playwright pendiente de sesión humana (mismo bloqueo transversal de toda la iniciativa). Desviaciones documentadas en el manifiesto: selector manual de `shipmentId` (mismo patrón que SP-10 aéreo) y de `operationId` para abrir `SolicitudFondosDrawer` (SP-14 no expone lista de operaciones por movimiento); el folio de la Solicitud de Fondos resultante se captura manualmente para vincularlo al Retorno de Vacío.

### Fase 3 — Operación y despacho
- [[2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]] — Tab Operaciones como agrupador vía DGO (#11). ✅ Cerrado (2026-07-12): solo `carmi-odin-api-v2` tocado — `ReferencesService.getOperations` (`src/references/services/references.service.ts`) suma una tercera rama al `OR` del `where` (`pedimentos.some.dgo.referenceId`) para traer operaciones cuyo vínculo real con la referencia vive en el DGO de alguno de sus pedimentos, no solo en `referenceId`/`operationMetadata.additionalReferenceIds` (legacy, snapshot estático al crear la operación que no se actualiza si se agregan pedimentos vía DGO después). `linkedReferences`/`allLinkedIds` también incorporan esas referencias derivadas de DGO. Front sin cambios: `ReferenceOperations.tsx` ya soportaba multi-referencia desde SPEC-007 (SP-07), D1 solo pedía ajuste de fuente de datos. Gate estático verde (7/7 tests en `references.service.spec.ts`, 2 nuevos cubriendo el `where` y el caso de operación vinculada solo vía DGO de otra referencia; tsc/eslint sin errores nuevos). Playwright pendiente de sesión humana (no toca front). Coordinar con Carlos (M8) sigue siendo nota externa, sin acción tomada aquí.
- [[2026-07-10-refactor-flujo-ejecutivo-sp13-solicitar-salida]] — Solicitar Salida mini-modal (#22). ✅ Cerrado (2026-07-11): `OutboundForm.tsx`/`PedimentoSelectorForExit.tsx` (D1 literal) resultaron huérfanos, no conectados a ningún flujo vigente — en su lugar, endpoint nuevo `POST /operations/:id/request-exit` (`carmi-odin-api-v2`, `OperationsService.requestExit`) que hereda del INBOUND asociado (`referenceId`, `clientCompanyId`, `transportModeId`, `warehouseLocationId`, `carrierId`, `customsBrokerId`) y delega en `ShipmentsService.create` (mismo método del flujo OUTBOUND normal, sin duplicar `validateInventoryAvailability`); lanza 400 si no hay INBOUND ("salida requiere entrada"). Front: mini-modal nuevo `SolicitarSalidaModal.tsx` (patrón `RequestPrevioModal`) con solo sello/contenedor/tipo (`useContainerTypes` + `SearchableSelect` reusado), botón "Solicitar Salida" en `OperationHeader.tsx` (solo visible si la operación tiene INBOUND). Ambos repos, gate estático verde (odin: tsc/eslint sin errores nuevos, 523 test suites/3447 tests OK incluyendo 3 nuevos de `requestExit`; digital: tsc 0 errores en todo el repo, eslint 0 errores en archivos tocados — `next lint` está roto repo-wide, hallazgo aparte no relacionado). Playwright pendiente de sesión humana (servidores dev ocupados por otra sesión en curso, no se reiniciaron para no interferir). Ver manifiesto para detalle de la brecha D1 vs. estado real.
- [[2026-07-10-refactor-flujo-ejecutivo-sp14-solicitud-fondos]] — Solicitud de Fondos 3 categorías (#23). ✅ Cerrado (2026-07-12): módulo nuevo `FundsRequest`/`FundsRequestItem` en `carmi-odin-api-v2` (`src/funds-requests/**`), independiente del puente legacy `Operation.solicitarFondos` (Carmi DB, solo operaciones migradas — se dejó intacto). Impuestos = `Pedimento.liquidacionEfectivo` sumado por operación; honorarios = motor nuevo (`FundsRequestCalculatorService`) que evalúa porcentaje/mínimo del tarifario del cliente, rango por peso (terrestre, vía `OperationShipment→Shipment.grossWeight`) y servicios de tarifario ligados a `ServiceUnitCriterion` código 22 ("Honorarios"), tomando el mayor candidato; gastos comprobados = captura manual dentro del módulo (decisión: no se reutilizó `GlobalExpense`, que es para incrementables/decrementables de valoración aduanera, dominio distinto). Front: `ReferenceOperations.tsx` (líneas 197-221 de la versión pre-SP-14) reemplaza el POST directo + `alert()` por un nuevo drawer `SolicitudFondosDrawer.tsx` con toast (`useToast`). Ambos repos, gate estático verde (odin: 19/19 tests nuevos, tsc/eslint sin errores nuevos; digital: tsc/eslint 0 errores). Playwright pendiente de sesión humana. Ver manifiesto para detalle de la brecha entre lo que decía el D1 del sub-plan (backend "vacío") y lo que había realmente en el repo.
- [[2026-07-10-refactor-flujo-ejecutivo-sp15-proforma-pedimento]] — Ver Proforma/Pedimento + limpiar código muerto (#24). ✅ Cerrado (2026-07-12): sub-plan de puro reuso/limpieza, sin cambios funcionales. El viewer real (`OperationProformaDrawer.tsx`, montado en `ReferenceOperations.tsx`) ya reflejaba correctamente identificadores (SPEC-009), multi-pedimento y TC efectivo (SPEC-TC-001) desde antes de este paraguas — no requirió refactor. Los 3 archivos muertos que el D1 señalaba (`PedimentoProformaDocument.tsx`, `PedimentoHbsDocument.tsx`, `ReferencePedimento.tsx`) ya habían sido eliminados por SP-04. Hallazgo propio no listado en el D1: `drawers/pedimento/ProformaHeader.tsx`, `ProformaPartidas.tsx`, `ProformaLiquidation.tsx`, `ProformaAnnexes.tsx` (implementación SPEC-008 anterior del Anexo 22, huérfana desde que `PedimentoHtmlViewer.tsx` la reemplazó) también sin importadores — se eliminaron junto con la limpieza declarada. Solo `carmi-digital` tocado. Gate estático verde (tsc limpio, eslint sin errores nuevos en archivos tocados). Playwright pendiente de sesión humana.
- [[2026-07-10-refactor-flujo-ejecutivo-sp16-despacho-stepper]] — Stepper de despacho: 7 pasos, retirar GLOSA (queda como gate interno), Shipper condicional (#25). 🟡 Parcialmente completado (2026-07-12, segundo intento sobre D1 rediseñado) — SHIPPER condicional implementado end-to-end (Prisma + workflow + front), GLOSA retirada de tabs con panel inline, MODULACION con contenido real de solo lectura, archivos muertos limpiados; la purga del sistema legacy de despacho (JSON blob) quedó diferida a un sub-plan propio por entrelazamiento con rutas hoy vivas (`regenerate-pedimento`, `completeModulacion`, posible webhook externo de Bifrost). Puertas estáticas verdes en ambos repos; verificación Playwright pendiente (sin `dev_url`). Ver manifiesto.

### Fase 4 — Entrada y cliente
- [[2026-07-10-refactor-flujo-ejecutivo-sp17-bandeja-entrada]] — Bandeja de Entrada / Inbox (#1). 🚧 Bloqueado (2026-07-12): D1 desactualizado/inválido en 3 de 4 piezas (tabs automático/temporal/avanzado, cartas sin firmar, guías sin identificar — ninguna existe como dato en ningún repo, requieren decisión de producto). Sin código. Ver manifiesto.
- [[2026-07-10-refactor-flujo-ejecutivo-sp18-portal-documentos-cliente]] — Portal de Documentos del Cliente (#26). ✅ Cerrado (2026-07-12): módulo nuevo `carmi-odin-api-v2/src/reference-portal/**` — token público (`ReferencePublicToken`) atado a `clientCompanyId` (no a una sola Reference, a diferencia del precedente 1:1 de OSD) para soportar la búsqueda por OC entre varios expedientes del mismo cliente; cada acción sobre una Reference puntual valida `clientCompanyId` y responde 404 si no coincide (mitigación IDOR). Documentos del cliente entran por `ReferenceDocumentsService.create` (SP-04) con el nuevo flag `uploadedByClient`, y disparan `reglosaExpediente` al subir. Preview/descarga individual usa signed URLs de GCP de ~10 min generados al vuelo (no el `publicUrl` de la carga); el ZIP del expediente completo se arma en streaming con `archiver` (dependencia nueva, fijada a v7 por incompatibilidad ESM de v8 con este proyecto CommonJS). Front: página nueva `app/(public)/document-portal/[token]/page.tsx` + proxies `app/api/public/document-portal/**` (mismo patrón que OSD), sin tocar `components/operations`/`customerPortal/operations` (fuera de alcance). Ambos repos, gate estático verde (odin: 527 test suites/3472 tests OK, tsc/eslint sin errores nuevos; digital: tsc 0 errores en todo el repo, eslint 0 errores en archivos nuevos, `next build` exitoso). Playwright pendiente de sesión humana con datos sembrados (compañía + referencia + OC de prueba). Ver manifiesto para detalle de diseño.

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
- **Reinicio de Fase 1 (2026-07-11):** las ramas `refactor/customs-operation-sp01`
  y `refactor/customs-operation-sp03` (digital) y `refactor/customs-operation-sp01`
  / `sp02` (odin, esta última era idéntica a sp01 — solo el punto de partida
  encadenado para SP-02, sin trabajo propio) se descartaron y eliminaron a
  petición del usuario para rehacer SP-03 y SP-01 desde cero. La cadena vuelve
  a partir de las ramas base: `carmi-digital` en `test`, `carmi-odin-api-v2` en
  `staging`. SP-00 (spike, sin código) se deja cerrado sin cambios.

## Estado del paraguas

✅ **Fase 1 completa cerrada (2026-07-12)** — sus 6 sub-planes están ✅:
SP-00 (spike, sin código), SP-03 (shell menú lateral), SP-01 (Listado de
Referencias), SP-02 (Wizard 3 pasos crear referencia — cierre parcial
documentado en su propio manifiesto: bloqueo de infraestructura en la
migración Prisma del campo OC, sin impacto en el resto de la cadena), SP-04
(Expediente + glosado), SP-05 (DGO) y SP-06 (Wizard Operación → Step 0 DGO,
multi-referencia/multi-DGO). Cadena de ramas encadenada sin cortes:
SP-03 → SP-01 → SP-02 → SP-04 → SP-05 → SP-06, diff acumulado sin commitear en
ambos repos (`carmi-digital`, `carmi-odin-api-v2`), rama compartida
`refactor/customs-operation-sp06`.

**Fase 2 iniciada:** SP-07 (tab Movimientos por tráfico) ✅ cerrado
(2026-07-11), primer hijo transversal de Fase 2. SP-08 (Recinto por tráfico)
✅ cerrado (2026-07-11) — solo tocó `carmi-digital`. Cadena de ramas
extendida: ...→ SP-06 → SP-07 → SP-08 (`refactor/customs-operation-sp08` en
`carmi-digital`; `carmi-odin-api-v2` se quedó en la rama `sp08` creada por
convención pero sin diff propio, sigue cargando el diff acumulado de SP-07),
diff acumulado sin commitear en ambos repos.

✅ **Fase 2 completa cerrada (2026-07-12)** — sus 5 sub-planes están ✅: SP-07
(tab Movimientos por tráfico), SP-08 (Recinto por tráfico), SP-09 (Previo +
OS&D), SP-10 (Movimientos aéreo) y SP-11 (Movimientos marítimo). Cadena de
ramas extendida: ...→ SP-08 → SP-09 → SP-10 → SP-14 → SP-11
(`refactor/customs-operation-sp11` en ambos repos), diff acumulado sin
commitear en ambos repos. `TransportAssignmentPanel` (SP-10, path-neutral)
terminó reusado por SP-11 sin cambios, validando el contrato documentado en
su manifiesto.

Pendiente transversal (todas las fases): **validación Playwright end-to-end de
principio a fin sigue pendiente de sesión humana** — ningún sub-plan de Fase 1
levantó el entorno con Playwright (falta `dev_url` configurado para este
cliente; SP-06 además documentó en su manifiesto un bloqueo preexistente y no
relacionado de `next build`/Turbopack por dependencias `three`/`mammoth` no
resueltas en `node_modules`, a resolver con un `pnpm install` limpio antes de
la sesión de Playwright).

## Cierre del paraguas (2026-07-12)

**SP-18 fue el último sub-plan del paraguas.** Con su cierre, la Fase 4 y la
lista completa de sub-planes quedan resueltos, con dos excepciones explícitas
que requieren decisión humana antes de poder retomarse:

- **SP-16 (Stepper de despacho)**: 🟡 Parcialmente completado (2026-07-12,
  segundo intento sobre el D1 rediseñado) — implementado SHIPPER condicional
  (Prisma + Temporal workflow + front), retiro de GLOSA como tab con panel
  inline, contenido real de MODULACION, limpieza de archivos muertos. Diferida
  a un sub-plan propio la purga del sistema legacy de despacho (JSON blob),
  por depender de rutas hoy en vivo no cubiertas por el D1 (`regenerate-pedimento`,
  `completeModulacion`/`executeStepPublic`, posible webhook externo de
  Bifrost) — ver su manifiesto.
- **SP-17 (Bandeja de Entrada / Inbox)**: 🚧 Bloqueado — D1 desactualizado/
  inválido en 3 de 4 piezas (tabs automático/temporal/avanzado, cartas sin
  firmar, guías sin identificar), sin dato real en ningún repo; requiere
  decisión de producto antes de poder diseñarse. Sin código.

Todo lo demás (Fases 1-2 completas, SP-12/13/14/15 de Fase 3, y SP-18 de Fase
4) está ✅ cerrado, con la validación Playwright end-to-end pendiente de
sesión humana en todos los casos (transversal, no es un bloqueo de ningún
sub-plan en particular — falta `dev_url`/entorno levantado + datos de prueba
sembrados). El diff acumulado de toda la ejecución sigue sin commitear en
ambos repos, en la rama compartida `refactor/customs-operation-sp18`.
