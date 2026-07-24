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

## Referencia, DGO y Operaciones (core)

- [ ] Implementar movimientos dinámicos por tipo de tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: entrada, subdivisión, salida, previo vía MovementTypeByTraffic, sigue vigente; aéreo: cerrado — no requiere catálogo de movimientos ni Subdivisión, el régimen mixto se resuelve vía Dgo/Invoice, ver [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; marítimo: sigue sin definir)
- [ ] Implementar consolidación de DGO (unificar Factura + Packing List + Mercancía + Config. Pedimento) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: 1 DGO = 1 Pedimento, actúa como source of truth)
- [ ] Implementar generación de operaciones desde DGOs (no desde movimientos) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; nota: cambio fundamental en lógica)
- [ ] Implementar nomenclatura dinámica (Recinto/Previo) por tráfico ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; terrestre: "Verificación", marítimo/aéreo: "Previo")
- [ ] Implementar validación mejorada ETA + fecha de arribo ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; no solo ETA, sino validar arribo real y proceso posterior)

## Detalle de Referencia y UI

- [ ] Implementar pestaña Tickets en detalle de referencia (reemplazar Instrucciones) ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; permitir crear o vincular ticket existente)
- [ ] Reordenar pestañas en detalle de referencia ([[2026-07-13 - Revisión de pantallas (bandeja entrada, detalle referencia)]])

## Expediente Aduanero

- [ ] Implementar checklist automático en Expediente Aduanero, conectado a reglas reales de documentos obligatorios por perfil de cliente desde MOMP ([[2026-07-14 - Prueba operación real desde ticket a facturación]], [[2026-07-10-refactor-flujo-ejecutivo-sp04-expediente-aduanero]]; basado en perfiles: cliente, número de parte, aduana, agente; nota: pendiente de análisis, no prioridad actual — hoy MOMP solo aporta un flag booleano de perfil completo, hace falta que exponga las reglas documentales reales por cliente)
- [ ] Implementar comparación trilateral en Expediente Aduanero ([[2026-07-14 - Prueba operación real desde ticket a facturación]]; PDF original vs datos extraídos vs datos glosados, Anexo 22)

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

### Régimen y DGO

- [ ] Asignar régimen por guía desde el tablero de Guías, heredable a la Referencia al crearla ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: el ejecutivo lo asigna guía por guía antes de crear la Referencia; si las guías seleccionadas traen regímenes distintos entre sí, se genera 1 DGO por cada régimen diferente presente en la selección)
- [ ] Validar guía máster y régimen homogéneo al crear/editar una Referencia ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: hoy esa validación solo existe al crear la Operación, no al armar la Referencia)
- [ ] Generar/vincular AirManifest al crear Referencia desde el tablero de Guías ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: crear la Referencia es el único gesto necesario, no hace falta pantalla de asignación de facturas en la guía; nace con 1 DGO por default —o N si hubo régimen mixto— y el ejecutivo puede subdividir DGO/factura/partida después con el flujo normal ya existente)
- [ ] Vincular AirManifest con Invoice vía tabla puente InvoiceAirManifestLink ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: junction M:N nueva espejeando InvoiceShipmentLink ya existente en terrestre; no vincular a Dgo directo, sería redundante con Invoice.dgoId)
- [ ] Agregar proforma de pedimento dentro del tab DGO de la Referencia ([[2026-07-20 - Recap visita Querétaro]]; nota: visible vacía cuando no hay datos, se llena conforme se captura el DGO; distinta de la proforma de Operación ya existente; tarea asignada a Ángel en el reparto de trabajo)
- [ ] Construir UI de Expediente Aduanero para datos extraídos por Zeus, con generación y sincronización del DGO ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: al subir un documento —ej. factura comercial— si Zeus extrae datos que corresponden a una entidad del DGO, esa entidad se genera/llena automáticamente; en Expediente Aduanero se muestran esos datos extraídos y son editables; si el ejecutivo corrige un dato ahí, la corrección se propaga al DGO; esta tarea es solo la UI y la lógica de generación/sincronización, la integración real —llamada HTTP a Zeus— es tarea aparte, no cableada hoy)

### Previo

- [ ] Remodelar Previo de 1:1 con Referencia a 1:1 con guía ([[2026-07-20 - Recap visita Querétaro]]; nota: una referencia puede tener N guías y el previo se hace por guía)
- [ ] Mostrar comparación vacía por guía en tab Previo mientras no llega resultado ([[2026-07-20 - Recap visita Querétaro]]; nota: compara peso, bultos, piezas, marca/submodelo/modelo/serie y origen; se llena conforme llega el previo capturado en la app móvil; la creación del previo no compete a SIG3, el ejecutivo solo tiene acceso de lectura)
- [ ] Mantener "Solicitar Previo" como acción opcional/condicional ([[2026-07-20 - Recap visita Querétaro]]; nota: no siempre es necesaria, dado que la app móvil puede iniciar el previo de forma autónoma)

### Prealerta y menú

- [ ] Generalizar ingesta de prealerta a todos los clientes ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: hoy es exclusiva de Tyco, se generaliza por si algún otro cliente lo requiere a futuro)
- [ ] Quitar "Citas" del menú de detalle de referencia solo para tráfico aéreo y reordenar el menú ([[2026-07-20 - Recap visita Querétaro]]; nota: orden definido por Germán)

### Diferidas / pendientes de análisis (no prioridad actual)

- [ ] Definir equivalente marítimo del tablero de guías ([[2026-07-20 - Recap visita Querétaro]]; nota: esta ronda solo cubre tráfico aéreo Querétaro; depende de la reunión de marítimo del 21-jul que no se documentó)
- [ ] Analizar candado de catálogo de número de parte por autorización de cliente ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: comparar cómo lo maneja SIGAC 2 vs cómo abordarlo en SIGAC 3, confirmar cifra ~5,000/1,000,000 con el cliente, decidir si se automatiza o se mantiene aprobación humana, y si se configura por cliente)
- [ ] Definir catálogo de tipos de movimiento por tráfico para aéreo y encaje de la subdivisión de guía incompleta ([[2026-07-16 - Flujo diario de manifiestos, régimen y candados (Querétaro)]]; nota: **posible inconsistencia señalada 2026-07-22** — esta tarea sigue preguntando si se reusa la Subdivisión de Shipment; ya se resolvió en la sesión del 21-jul que aéreo NO la necesita (régimen mixto vía Dgo/Invoice, llegada parcial vía DGO/factura/partida); revisar si cerrar o reescribir)
- [ ] Definir dato ancla del previo en tráfico marítimo (guía vs. número de contenedor) ([[2026-07-20 - Recap visita Querétaro]])
- [ ] Dejar nota de escalabilidad del patrón guía↔DGO/régimen para tráfico marítimo ([[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]; nota: pendiente de la reunión de marítimo del 21-jul que no se documentó)
