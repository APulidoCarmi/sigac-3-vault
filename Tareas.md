# Tareas — sigac-3

Nota central de tareas del cliente (modo sin Jira). Gestionada por las
skills /hoy y /reunion: las tareas nuevas entran como `- [ ]` con
wikilink a la reunión de origen; al completarse se marcan `- [x]`.

Organizada por **sección temática** (no cronológica) desde 2026-07-22. Al
agregar una tarea nueva, ubícala bajo la sección que le corresponda; si no
calza en ninguna, crea una nueva sección o añádela al final.

## Bandeja de entrada y dashboard

- [x] Implementar límite de visualización en bandeja de entrada (máx 3-5 referencias/movimientos top) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: medida temporal hasta validar Excel con Enrique)
- [x] Validar bandeja de entrada con Enrique en pantalla ([[2026-07-14 - Prueba operación real desde ticket a facturación]])
- [x] Definir búsquedas y visualización del módulo de Pedimentos en dashboard ([[2026-07-14 - Prueba operación real desde ticket a facturación]])
- [ ] Agregar acción para marcar una referencia como "urgente" y que aparezca en la bandeja de entrada ([[2026-07-31 - Revisión de Movimientos - Guía]]; nota: pedido original de Enrique; v1 sin restricción de quién puede marcarla, se ajustan criterios después)

## Referencia, DGO y Operaciones (core)

- [ ] Implementar movimientos dinámicos por tipo de tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: entrada, subdivisión, salida, previo vía MovementTypeByTraffic, sigue vigente; aéreo: cerrado — no requiere catálogo de movimientos ni Subdivisión, el régimen mixto se resuelve vía Dgo/Invoice, ver [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; marítimo: sigue sin definir)
- [ ] Implementar consolidación de DGO (unificar Factura + Packing List + Mercancía + Config. Pedimento) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: 1 DGO = 1 Pedimento, actúa como source of truth)
- [ ] Implementar generación de operaciones desde DGOs (no desde movimientos) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: cambio fundamental en lógica)
- [ ] Implementar nomenclatura dinámica (Recinto/Previo) por tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: "Verificación", marítimo/aéreo: "Previo")
- [ ] Implementar validación mejorada ETA + fecha de arribo ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; no solo ETA, sino validar arribo real y proceso posterior)
- [ ] Mostrar rastreo de guías↔factura↔número de parte en el DGO y como pestaña en el detalle de referencia ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: Carlos Leobardo (Taico) necesita ver qué guías corresponden a qué factura y qué número de parte sin tener que entrar factura por factura; hoy en SIGAC 2 no existe esa vista y es lento con hasta 20 facturas por operación; aún no se inicia)
- [ ] Definir y agregar nivel de glosado "perfil de operación / tipo de operación" ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: distintos tipos de operación usan distintas fechas ancla para medir el tiempo de facturación (métrica de turnaround); **pendiente de definir con Germán** qué tipos de operación existen y qué fecha usa cada uno antes de poder implementarlo)

## Detalle de Referencia y UI

- [ ] Implementar pestaña Tickets en detalle de referencia (reemplazar Instrucciones) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; permitir crear o vincular ticket existente)
- [ ] Reordenar pestañas en detalle de referencia ([[2026-07-13 - Revisión de pantallas (bandeja entrada, detalle referencia)]])

## Expediente Aduanero

- [ ] Implementar checklist automático en Expediente Aduanero, conectado a reglas reales de documentos obligatorios por perfil de cliente desde MOMP ([[2026-07-14 - Prueba operación real desde ticket a facturación]], [[2026-07-10-refactor-flujo-ejecutivo-sp04-expediente-aduanero]]; basado en perfiles: cliente, número de parte, aduana, agente; nota: pendiente de análisis, no prioridad actual — hoy MOMP solo aporta un flag booleano de perfil completo, hace falta que exponga las reglas documentales reales por cliente)
- [ ] Implementar comparación trilateral en Expediente Aduanero ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; PDF original vs datos extraídos vs datos glosados, Anexo 22)
- [ ] Permitir subir varios documentos del mismo tipo en una sola carga de "document" ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: ej. varios certificados TLC o varios packing list juntos, declarando el tipo correctamente; hoy la pantalla truena al intentarlo)
- [ ] Ampliar el catálogo de tipos de documento según anexo 22 (carta 318, factura de fletes, póliza de seguros, etc.) en vez de forzar la categoría genérica "anexo" ([[2026-07-24 - Prueba operación real desde ticket a facturación]])
- [ ] Crear tipo de documento "Previo" (reporte de verificación previa), distinto de factura comercial ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: este tipo debe reflejarse en Expediente Aduanero para poder contrastar el previo contra la factura comercial durante la glosa digital)
- [ ] Mejorar UX de carga de documentos en Expediente Aduanero ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: subida múltiple sin reabrir el modal, thumbnail/preview de la primera página, mostrar el tipo de documento asignado directamente en el listado sin abrir sub-pantalla, buscador/filtro de documentos, y edición manual de la fecha de recepción)
- [ ] Permitir marcar cuál versión de un documento (ej. factura corregida vs. original con error) es la vigente que alimenta el DGO ([[2026-07-24 - Prueba operación real desde ticket a facturación]])
- [ ] **PRIORIDAD.** Al subir una factura en el DGO, integrar la factura al mismo objeto de expediente asociado al movimiento/referencia (mismo expediente donde se suben los documentos de la referencia) ([[2026-07-31 - Revisión de Movimientos - Guía]])
- [ ] Permitir cargar el documento de una factura en el DGO de forma que quede vinculado a esa factura (documento + datos manuales) y aparezca también en Expediente Aduanero, clasificado con tipo de documento 94 "Factura o lista de empaque comercial"; si no se sube el documento en ese momento y ya existe en Expediente Aduanero un documento no ligado a ninguna factura manual, poder ligar ese documento existente a la factura ([[2026-07-31 - Revisión de Movimientos - Guía]]; causa raíz analizada en [[2026-08-04 - Análisis causa raíz documento de factura no aparece en Expediente Aduanero]]: son tablas separadas —`invoice_images` vs `reference_documents`— sin ninguna FK/llamado que las cruce; falta decidir mapeo a enum `FACTURA` vs `LISTA_EMPAQUE`)

## Recinto y Citas

- [ ] Mover botón "Crear operación" fuera de DGO individual ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]])
- [ ] Migrar locations (recintos) a interfaz dinámicos por tráfico ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]; nota: migrar desde SIGAC 2 — analizar de dónde sacar los datos para la migración)
- [ ] Definir flujo de citas con ejecutivos y almacén ([[2026-07-13 - Revisión de pantallas (operaciones, recinto, citas)]]; nota: es NECESARIO tener esta reunión antes de implementar)

## Flujo Querétaro (aéreo)

Iniciativa derivada de la visita a Querétaro (16 y 20 de julio) y de la sesión
de análisis del 21-jul que reconcilió esos hallazgos contra el as-is del
código. Ver [[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]],
[[2026-07-20 - Recap visita Querétaro]], [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]].

### Tablero de Guías — ingesta y matching

- [x] Construir tablero de control de "guías" pre-referencia ([[2026-07-20 - Recap visita Querétaro]]; nota: ingesta del manifiesto diario de Terminal, alta de guías con estatus de seguimiento ya que no todas se procesan el mismo día, comparación contra prealerta cuando exista, y creación de referencia(s) desde selección de guías; pantalla NUEVA, distinta del air-manifest ya existente en código que rastrea etapas post-referencia; nombre "guías" acordado con Germán y Carlos, no "manifiesto")
- [ ] Ingerir manifiesto de guías línea por línea (por Guía House) con sus campos reales ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: capturar Descripción, Peso, Piezas, Bulto, Valor, Destinatario, Domicilio Destinatario, Remitente, Domicilio Remitente y Vuelo como campos propios —no enterrados en extractedJson—, agrupados por Guía Master/Guía House; hoy ni AirManifest ni UnidentifiedWaybill los tienen)
- [ ] Reconciliar Destinatario contra el catálogo de Company al momento de la ingesta ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: no existe hoy ningún matcher automático reusable —bol-reconciliation solo matchea número de guía exacto, company.service solo tiene búsqueda manual ILIKE—; el ejecutivo confirma/corrige el cliente candidato desde el momento en que se ingiere la guía, para que no nazca mostrándose como "sin identificar" en cuanto al cliente)
- [ ] Crear entidad Team (grupo de ejecutivos) y vincularla con Company ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: no existe hoy ningún concepto de equipo de trabajo —lo que hay, UserCompany/UserCompanyRole, es asignación individual 1 a 1—; se necesita para filtrar el tablero de Guías por "las guías de mi equipo" y para que un cliente tenga varios ejecutivos asignados de golpe)
- [ ] Extender el modelo de guía sin identificar existente para que la guía tenga ciclo de vida propio ([[2026-07-20 - Recap visita Querétaro]]; nota: reusar y extender la entidad ya existente en vez de crear una nueva)
- [ ] Vincular UnidentifiedWaybill con AirManifest vía linkedAirManifestId ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: FK en UnidentifiedWaybill —mismo patrón que linkedShipmentId/linkedReferenceId—, no al revés, ya que AirManifest.referenceId es obligatorio hoy; se reusa UnidentifiedWaybill —genérica BOL+AWB+OTHER— en vez de fusionar tablas o crear una nueva)
- [ ] Mostrar solo la guía house en la tabla de guías, guardando también la guía master en BD ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: confirmado por Daniel para Taico como la guía que da la directriz operativa real)
- [ ] Dividir la tabla de guías en tabs por tipo de tráfico (aéreo/marítimo/terrestre), visibles según el perfil/tráfico asignado al usuario ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: campos necesarios son sustancialmente distintos entre aéreo y marítimo, ver tarea de campos marítimos en sección Marítimo)
- [ ] Definir nuevos estatus de guía aérea: Revalidado, Peso certificado, Digitalizada ([[2026-07-24 - Prueba operación real desde ticket a facturación]])

### Movimientos y facturas por guía (Querétaro)

- [x] Agregar en el módulo de Guías el formulario de facturas para subir factura (documento + datos) vinculada directamente a la guía House ([[2026-07-31 - Revisión de Movimientos - Guía]]; implementado: `GuiaInvoicesModal.tsx` reutiliza `InvoiceFormModal`/`InvoiceMerchandiseAccordion` del DGO con `guiaId`; backend con campo `guiaId` en `Invoice` y auto-asignación de `referenceId`/`dgoId` en `invoices.service.ts`; nota: gap conocido documentado en el propio modal — si la guía aún no tiene Referencia/movimiento asociado, la factura no se propaga a Expediente Aduanero)
- [x] **PRIORIDAD.** Generar automáticamente un movimiento de entrada por cada guía House al crearse (backend + frontend visible en detalle de referencia; si referencia es aérea, formulario de movimiento de entrada muestra campos de guía, no de movimiento terrestre) ([[2026-07-31 - Revisión de Movimientos - Guía]])
- [ ] **PRIORIDAD.** Cuando se suba una factura desde DGO/Referencia sin guía asociada, preguntar a qué guía House pertenece para vincularla ([[2026-07-31 - Revisión de Movimientos - Guía]])

### Régimen y DGO

- [ ] Asignar régimen por guía desde el tablero de Guías, heredable a la Referencia al crearla ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: el ejecutivo lo asigna guía por guía antes de crear la Referencia; si las guías seleccionadas traen regímenes distintos entre sí, se genera 1 DGO por cada régimen diferente presente en la selección)
- [x] Agregar clave de pedimento como campo dependiente del régimen en el módulo de guías (select en cascada) ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; implementado 2026-07-26: `pedimentoCode` agregado a `CreateGuiaDto`/`UpdateGuiaDto` y a `GuiasService` en backend; en frontend, `GuiaFormModal` y `BulkAssignGuiasModal` usan `SearchableSelect` filtrado por `declarationCodes.filter(c => c.regimeCode === regimen)` vía `useCustomsDeclarationCodes`, reseteando la clave si cambia el régimen)
- [ ] Validar guía máster y régimen homogéneo al crear/editar una Referencia ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: hoy esa validación solo existe al crear la Operación, no al armar la Referencia)
- [ ] Generar/vincular AirManifest al crear Referencia desde el tablero de Guías ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: crear la Referencia es el único gesto necesario, no hace falta pantalla de asignación de facturas en la guía; nace con 1 DGO por default —o N si hubo régimen mixto— y el ejecutivo puede subdividir DGO/factura/partida después con el flujo normal ya existente)
- [ ] Vincular AirManifest con Invoice vía tabla puente InvoiceAirManifestLink ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: junction M:N nueva espejeando InvoiceShipmentLink ya existente en terrestre; no vincular a Dgo directo, sería redundante con Invoice.dgoId)
- [ ] Agregar proforma de pedimento dentro del tab DGO de la Referencia ([[2026-07-20 - Recap visita Querétaro]]; nota: visible vacía cuando no hay datos, se llena conforme se captura el DGO; distinta de la proforma de Operación ya existente; tarea asignada a Ángel en el reparto de trabajo)
- [ ] Construir UI de Expediente Aduanero para datos extraídos por Zeus, con generación y sincronización del DGO ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]], [[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: al subir un documento —ej. factura comercial— si Zeus extrae datos que corresponden a una entidad del DGO, esa entidad se genera/llena automáticamente; en Expediente Aduanero se muestran esos datos extraídos y son editables; si el ejecutivo corrige un dato ahí, la corrección se propaga al DGO; esta tarea es solo la UI y la lógica de generación/sincronización, la integración real —llamada HTTP a Zeus— es tarea aparte, no cableada hoy; nota 2026-07-24: debe mostrarse marcado con el ícono de Zeus para diferenciarlo del documento fuente)

### Previo

- [ ] Remodelar Previo de 1:1 con Referencia a 1:1 con guía ([[2026-07-20 - Recap visita Querétaro]]; nota: una referencia puede tener N guías y el previo se hace por guía)
- [ ] Mostrar comparación vacía por guía en tab Previo mientras no llega resultado ([[2026-07-20 - Recap visita Querétaro]]; nota: compara peso, bultos, piezas, marca/submodelo/modelo/serie y origen; se llena conforme llega el previo capturado en la app móvil; la creación del previo no compete a SIG3, el ejecutivo solo tiene acceso de lectura)
- [ ] Mantener "Solicitar Previo" como acción opcional/condicional ([[2026-07-20 - Recap visita Querétaro]]; nota: no siempre es necesaria, dado que la app móvil puede iniciar el previo de forma autónoma)

### Prealerta y menú

- [ ] Generalizar ingesta de prealerta a todos los clientes ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: hoy es exclusiva de Tyco, se generaliza por si algún otro cliente lo requiere a futuro)
- [ ] Quitar "Citas" del menú de detalle de referencia solo para tráfico aéreo y reordenar el menú ([[2026-07-20 - Recap visita Querétaro]]; nota: orden definido por Germán)

### Diferidas / pendientes de análisis (no prioridad actual)

- [ ] Definir equivalente marítimo del tablero de guías ([[2026-07-20 - Recap visita Querétaro]], [[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: la ronda del 20-jul solo cubrió tráfico aéreo Querétaro; la sesión del 24-jul con Rocío (Manzanillo/GKN) aportó el detalle de campos pendiente: referencia cliente, referencia SIGAC, proveedor, bultos o contenedores, tipo de contenedor —dry cargo/high cube/etc., catálogo abierto—, buque, recinto —catálogo de Manzanillo ya existe—, transportista, ETA, estatus de captura del documento tipo "A24", observaciones libres, "envío de POD" y fecha de cita)
- [ ] Analizar candado de catálogo de número de parte por autorización de cliente ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: comparar cómo lo maneja SIGAC 2 vs cómo abordarlo en SIGAC 3, confirmar cifra ~5,000/1,000,000 con el cliente, decidir si se automatiza o se mantiene aprobación humana, y si se configura por cliente)
- [ ] Definir catálogo de tipos de movimiento por tráfico para aéreo y encaje de la subdivisión de guía incompleta ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: **posible inconsistencia señalada 2026-07-22** — esta tarea sigue preguntando si se reusa la Subdivisión de Shipment; ya se resolvió en la sesión del 21-jul que aéreo NO la necesita (régimen mixto vía Dgo/Invoice, llegada parcial vía DGO/factura/partida); revisar si cerrar o reescribir)
- [ ] Definir dato ancla del previo en tráfico marítimo (guía vs. número de contenedor) ([[2026-07-20 - Recap visita Querétaro]])
- [ ] Dejar nota de escalabilidad del patrón guía↔DGO/régimen para tráfico marítimo ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: pendiente de la reunión de marítimo del 21-jul que no se documentó)

## Marítimo (Manzanillo/Veracruz)

- [ ] Definir sección de citas propia para marítimo (previo, despacho, retorno de vacío, PIT) ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: previo/despacho/retorno de vacío son de Veracruz (Rocío); PIT es exclusivo de Manzanillo (Lilia) y no es una cita en sí, sino la vinculación del transporte a la terminal vía AIPA trámite y despacho)
- [ ] Evaluar con Rocío, Mariana y Lilia la API de tracking de buques (no contenedores) ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: limitación detectada en la demo — la API da el ETA al siguiente puerto de tránsito, no al puerto final si hay transbordos; falta validar si existe un "port log" que resuelva esto; reunión aparte a organizar por Germán)

## Seguimientos puntuales

- [ ] Revisar con Fer (lunes 2026-07-27) qué métodos de valoración e incoterms de proveedores están o no configurados/sincronizados para Taico ([[2026-07-24 - Prueba operación real desde ticket a facturación]]; nota: detectado en la demo que el sistema no jaló automáticamente el método de valoración/incoterm de un proveedor)

## Manifestación de Valor / COVE

- [ ] Implementar en factura los conceptos de incrementables desglosados según la manifestación de valor (COVE) ([[2026-07-31 - Daily Scrum - incrementables de factura para manifestación de valor y subdivisión por guía]]; nota: dividir el campo genérico de incrementable en los 6-7 conceptos que exige la manifestación —vs. los 3-4 actuales de factura (flete, embalaje, seguro, otros)—, y agregar a factura fechas, precio pagado/por pagar/compenso pago (aclarar con Enrique el término exacto de esta tercera opción), forma de pago, dos campos libres y manejo de moneda —factura en su moneda original, manifestación en pesos—; urgente: el proceso de manifestación de valor arranca 2026-08-01; guiarse del desarrollo que ya hacen Carlos/Brenda/Miguel en Sigac 2.0. **Validar además** (no implementar de nuevo) que los campos de vinculación y método de valoración por factura —que Angel confirmó en el meet que ya existen ("sí, lo tengo")— efectivamente cubran lo que pide la manifestación de valor para esos dos conceptos)
