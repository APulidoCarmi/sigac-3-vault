# Tareas — sigac-3

Nota central de tareas del cliente (modo sin Jira). Gestionada por las
skills /hoy y /reunion: las tareas nuevas entran como `- [ ]` con
wikilink a la reunión de origen; al completarse se marcan `- [x]`.

Organizada por **sección temática** (no cronológica) desde 2026-07-22. Al
agregar una tarea nueva, ubícala bajo la sección que le corresponda; si no
calza en ninguna, crea una nueva sección o añádela al final.

## Bandeja de entrada y dashboard

- [x] Implementar límite de visualización en bandeja de entrada (máx 3-5 referencias/movimientos top) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: medida temporal hasta validar Excel con Enrique) — SCRUM-496
- [x] Validar bandeja de entrada con Enrique en pantalla ([[2026-07-14 - Prueba operación real desde ticket a facturación]]) — SCRUM-497
- [x] Definir búsquedas y visualización del módulo de Pedimentos en dashboard ([[2026-07-14 - Prueba operación real desde ticket a facturación]]) — SCRUM-498

## Referencia, DGO y Operaciones (core)

- [x] Implementar movimientos dinámicos por tipo de tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: entrada, subdivisión, salida, previo vía MovementTypeByTraffic, sigue vigente; aéreo: cerrado — no requiere catálogo de movimientos ni Subdivisión, el régimen mixto se resuelve vía Dgo/Invoice, ver [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; marítimo: sigue sin definir) — SCRUM-499
- [x] Implementar consolidación de DGO (unificar Factura + Packing List + Mercancía + Config. Pedimento) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: 1 DGO = 1 Pedimento, actúa como source of truth) — SCRUM-519
- [x] Implementar generación de operaciones desde DGOs (no desde movimientos) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: cambio fundamental en lógica) — SCRUM-520
- [x] Implementar nomenclatura dinámica (Recinto/Previo) por tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: "Verificación", marítimo/aéreo: "Previo") — SCRUM-521
- [x] Implementar validación mejorada ETA + fecha de arribo ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; no solo ETA, sino validar arribo real y proceso posterior) — SCRUM-522
- [x] Mostrar rastreo de guías↔factura↔número de parte en el DGO y como pestaña en el detalle de referencia ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: Carlos Leobardo (Taico) necesita ver qué guías corresponden a qué factura y qué número de parte sin tener que entrar factura por factura; hoy en SIGAC 2 no existe esa vista y es lento con hasta 20 facturas por operación; aún no se inicia) — SCRUM-523

## Detalle de Referencia y UI

- [x] Implementar pestaña Tickets en detalle de referencia (reemplazar Instrucciones) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; permitir crear o vincular ticket existente) — SCRUM-526

## Expediente Aduanero

- [x] Implementar checklist automático en Expediente Aduanero, conectado a reglas reales de documentos obligatorios por perfil de cliente desde MOMP ([[2026-07-14 - Prueba operación real desde ticket a facturación]], [[2026-07-10-refactor-flujo-ejecutivo-sp04-expediente-aduanero]]; basado en perfiles: cliente, número de parte, aduana, agente; nota: pendiente de análisis, no prioridad actual — hoy MOMP solo aporta un flag booleano de perfil completo, hace falta que exponga las reglas documentales reales por cliente) — SCRUM-495
- [x] **PRIORIDAD.** Al subir una factura en el DGO, integrar la factura al mismo objeto de expediente asociado al movimiento/referencia (mismo expediente donde se suben los documentos de la referencia) ([[2026-07-31 - Revisión de Movimientos - Guía]]) — SCRUM-524
- [x] Permitir cargar el documento de una factura en el DGO de forma que quede vinculado a esa factura (documento + datos manuales) y aparezca también en Expediente Aduanero, clasificado con tipo de documento 94 "Factura o lista de empaque comercial"; si no se sube el documento en ese momento y ya existe en Expediente Aduanero un documento no ligado a ninguna factura manual, poder ligar ese documento existente a la factura ([[2026-07-31 - Revisión de Movimientos - Guía]]; causa raíz analizada en [[2026-08-04 - Análisis causa raíz documento de factura no aparece en Expediente Aduanero]]: son tablas separadas —`invoice_images` vs `reference_documents`— sin ninguna FK/llamado que las cruce; falta decidir mapeo a enum `FACTURA` vs `LISTA_EMPAQUE`) — SCRUM-525

## Recinto y Citas

- [x] Mover botón "Crear operación" fuera de DGO individual ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]) — SCRUM-527

## Flujo Querétaro (aéreo)

Iniciativa derivada de la visita a Querétaro (16 y 20 de julio) y de la sesión
de análisis del 21-jul que reconcilió esos hallazgos contra el as-is del
código. Ver [[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]],
[[2026-07-20 - Recap visita Querétaro]], [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]].

### Tablero de Guías — ingesta y matching

- [x] Construir tablero de control de "guías" pre-referencia ([[2026-07-20 - Recap visita Querétaro]]; nota: ingesta del manifiesto diario de Terminal, alta de guías con estatus de seguimiento ya que no todas se procesan el mismo día, comparación contra prealerta cuando exista, y creación de referencia(s) desde selección de guías; pantalla NUEVA, distinta del air-manifest ya existente en código que rastrea etapas post-referencia; nombre "guías" acordado con Germán y Carlos, no "manifiesto") — SCRUM-500
- [x] Extender el modelo de guía sin identificar existente para que la guía tenga ciclo de vida propio ([[2026-07-20 - Recap visita Querétaro]]; nota: reusar y extender la entidad ya existente en vez de crear una nueva) — SCRUM-528
- [x] Vincular UnidentifiedWaybill con AirManifest vía linkedAirManifestId ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: FK en UnidentifiedWaybill —mismo patrón que linkedShipmentId/linkedReferenceId—, no al revés, ya que AirManifest.referenceId es obligatorio hoy; se reusa UnidentifiedWaybill —genérica BOL+AWB+OTHER— en vez de fusionar tablas o crear una nueva) — SCRUM-531
- [x] Mostrar solo la guía house en la tabla de guías, guardando también la guía master en BD ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: confirmado por Daniel para Taico como la guía que da la directriz operativa real) — SCRUM-529
- [x] Definir nuevos estatus de guía aérea: Revalidado, Peso certificado, Digitalizada ([[2026-07-24 - Prueba operación real desde ticket a facturación]]) ([[2026-08-11-nuevos-estatus-guia-aerea]]) — SCRUM-504

### Movimientos y facturas por guía (Querétaro)

- [x] Agregar en el módulo de Guías el formulario de facturas para subir factura (documento + datos) vinculada directamente a la guía House ([[2026-07-31 - Revisión de Movimientos - Guía]]; implementado: `GuiaInvoicesModal.tsx` reutiliza `InvoiceFormModal`/`InvoiceMerchandiseAccordion` del DGO con `guiaId`; backend con campo `guiaId` en `Invoice` y auto-asignación de `referenceId`/`dgoId` en `invoices.service.ts`; nota: gap conocido documentado en el propio modal — si la guía aún no tiene Referencia/movimiento asociado, la factura no se propaga a Expediente Aduanero) — SCRUM-501
- [x] **PRIORIDAD.** Generar automáticamente un movimiento de entrada por cada guía House al crearse (backend + frontend visible en detalle de referencia; si referencia es aérea, formulario de movimiento de entrada muestra campos de guía, no de movimiento terrestre) ([[2026-07-31 - Revisión de Movimientos - Guía]]) — SCRUM-502
- [ ] **PRIORIDAD.** Cuando se suba una factura desde DGO/Referencia sin guía asociada, preguntar a qué guía House pertenece para vincularla ([[2026-07-31 - Revisión de Movimientos - Guía]]) — SCRUM-505

### Régimen y DGO

- [x] Asignar régimen por guía desde el tablero de Guías, heredable a la Referencia al crearla ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: el ejecutivo lo asigna guía por guía antes de crear la Referencia; si las guías seleccionadas traen regímenes distintos entre sí, se genera 1 DGO por cada régimen diferente presente en la selección) — SCRUM-530
- [x] Agregar clave de pedimento como campo dependiente del régimen en el módulo de guías (select en cascada) ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; implementado 2026-07-26: `pedimentoCode` agregado a `CreateGuiaDto`/`UpdateGuiaDto` y a `GuiasService` en backend; en frontend, `GuiaFormModal` y `BulkAssignGuiasModal` usan `SearchableSelect` filtrado por `declarationCodes.filter(c => c.regimeCode === regimen)` vía `useCustomsDeclarationCodes`, reseteando la clave si cambia el régimen) — SCRUM-503
- [x] Validar guía máster y régimen homogéneo al crear/editar una Referencia ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: hoy esa validación solo existe al crear la Operación, no al armar la Referencia) — SCRUM-532
- [x] Generar/vincular AirManifest al crear Referencia desde el tablero de Guías ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: crear la Referencia es el único gesto necesario, no hace falta pantalla de asignación de facturas en la guía; nace con 1 DGO por default —o N si hubo régimen mixto— y el ejecutivo puede subdividir DGO/factura/partida después con el flujo normal ya existente) — SCRUM-533
- [x] Vincular AirManifest con Invoice vía tabla puente InvoiceAirManifestLink ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: junction M:N nueva espejeando InvoiceShipmentLink ya existente en terrestre; no vincular a Dgo directo, sería redundante con Invoice.dgoId) — SCRUM-534
- [x] Agregar proforma de pedimento dentro del tab DGO de la Referencia ([[2026-07-20 - Recap visita Querétaro]]; nota: visible vacía cuando no hay datos, se llena conforme se captura el DGO; distinta de la proforma de Operación ya existente; tarea asignada a Ángel en el reparto de trabajo) — SCRUM-535

### Previo

- [x] Remodelar Previo de 1:1 con Referencia a 1:1 con guía ([[2026-07-20 - Recap visita Querétaro]]; nota: una referencia puede tener N guías y el previo se hace por guía) — SCRUM-536
- [x] Mantener "Solicitar Previo" como acción opcional/condicional ([[2026-07-20 - Recap visita Querétaro]]; nota: no siempre es necesaria, dado que la app móvil puede iniciar el previo de forma autónoma) — SCRUM-538

### Prealerta y menú

- [x] Quitar "Citas" del menú de detalle de referencia solo para tráfico aéreo y reordenar el menú ([[2026-07-20 - Recap visita Querétaro]]; nota: orden definido por Germán) — SCRUM-537

### Diferidas / pendientes de análisis (no prioridad actual)

- [ ] Definir equivalente marítimo del tablero de guías ([[2026-07-20 - Recap visita Querétaro]], [[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: la ronda del 20-jul solo cubrió tráfico aéreo Querétaro; la sesión del 24-jul con Rocío (Manzanillo/GKN) aportó el detalle de campos pendiente: referencia cliente, referencia SIGAC, proveedor, bultos o contenedores, tipo de contenedor —dry cargo/high cube/etc., catálogo abierto—, buque, recinto —catálogo de Manzanillo ya existe—, transportista, ETA, estatus de captura del documento tipo "A24", observaciones libres, "envío de POD" y fecha de cita) — SCRUM-539
- [ ] Dejar nota de escalabilidad del patrón guía↔DGO/régimen para tráfico marítimo ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: pendiente de la reunión de marítimo del 21-jul que no se documentó) — SCRUM-540

## Marítimo (Manzanillo/Veracruz)

- [ ] Definir sección de citas propia para marítimo (previo, despacho, retorno de vacío, PIT) ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: previo/despacho/retorno de vacío son de Veracruz (Rocío); PIT es exclusivo de Manzanillo (Lilia) y no es una cita en sí, sino la vinculación del transporte a la terminal vía AIPA trámite y despacho) — SCRUM-541

## Seguimientos puntuales


## Manifestación de Valor / COVE

- [ ] Implementar en factura los conceptos de incrementables desglosados según la manifestación de valor (COVE) ([[2026-07-31 - Daily Scrum - incrementables de factura para manifestación de valor y subdivisión por guía]]; nota: dividir el campo genérico de incrementable en los 6-7 conceptos que exige la manifestación —vs. los 3-4 actuales de factura (flete, embalaje, seguro, otros)—, y agregar a factura fechas, precio pagado/por pagar/compenso pago (aclarar con Enrique el término exacto de esta tercera opción), forma de pago, dos campos libres y manejo de moneda —factura en su moneda original, manifestación en pesos—; urgente: el proceso de manifestación de valor arranca 2026-08-01; guiarse del desarrollo que ya hacen Carlos/Brenda/Miguel en Sigac 2.0. **Validar además** (no implementar de nuevo) que los campos de vinculación y método de valoración por factura —que Angel confirmó en el meet que ya existen ("sí, lo tengo")— efectivamente cubran lo que pide la manifestación de valor para esos dos conceptos) — SCRUM-542
