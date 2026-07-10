# Reunión 2026-04-28 — Prueba operación real desde ticket a facturación

Revisión de principio a fin (interoperabilidad entre equipos): se empieza una
operación y se avanza hasta el final recogiendo feedback. Dos grandes bloques:
**(A)** módulo de Clasificación / Regla Octava (con clasificadores externos) y
**(B)** revisión campo a campo del formulario de **Movimiento de salida**, que
desemboca en **(C)** un rediseño del orden del flujo operación ↔ salida.

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo)
- German Castro (producto/dirección)
- Carlos Alexis Galaviz Rosas (implementación / almacén-WMS)
- Elian Shair Armendariz Puch (desarrollo — clasificación / regla octava)
- Enrique Lopez (ejecutivo de cliente / operación)
- Perla Lopez (operación)
- Salvador Cervantes Franco (clasificador)
- Geovanny Manuel Martinez (clasificador)
- Jose Antonio Limon (ejecutivo — manejaba reglas octavas de GKN)
- Daniel Peña (almacén)
- Fernando Angel Lopez Soto (ingeniería), Miguel Gomez, Cesar Aguirre

## Decisiones (con su porqué)

### A. Módulo Clasificación / Regla Octava → "Permisos"
- **Renombrar el módulo "Regla octava" a "Permisos"** y usar **un solo listado
  para todos los tipos de permiso** (regla octava, cartas cupo, registros
  sanitarios, permisos previos, NOMs, avisos automáticos…), con **filtro por
  tipo de permiso**. Porqué: Salvador y Geovanny lo prefieren a tener un listado
  por tipo — es más sencillo y todos los permisos comparten naturaleza (vigencia
  y/o cantidad de mercancía con "descargos" por operación).
- **Agregar columna "clave" del permiso** (C1, N6, NM para NOMs, etc.). Se saca
  del **Anexo 22, páginas 115–117** (viene clave y descripción, separado por
  Secretaría: Salud, Economía…). Formato clave/descripción como el catálogo.
- **La regla octava se liga a nivel fracción**, con relación **N:N**: un número
  de parte puede tener varias reglas octavas y una regla octava varios números
  de parte (confirmado por Geovanny). El **clasificador** la da de alta y aparece
  como **sugerencia** al ejecutivo, que puede seleccionar otra de la misma
  fracción o "sin regla octava".
- **Mostrar al ejecutivo solo los permisos vigentes** para seleccionar (ocultar
  los vencidos en la selección) — para que solo elija los listos para usarse.
- **Mostrar saldos del permiso** (cantidad y valor) con un indicador discreto
  (p. ej. tooltip al pasar el mouse) de vigencia/saldo; el sistema debe **leer
  contra el registro al capturar** y **alertar o bloquear validación** si el
  permiso está vencido o por vencerse (aplica a *cualquier* permiso, no solo
  regla octava). Matiz (Perla): el folio no se renueva —es otro trámite— y el
  cliente lleva su propio conteo de saldos; aun así la alerta de vencimiento se
  considera útil para avisar al cliente ~1 mes antes.
- **Digitalizar (subir) el PDF del permiso** y guardarlo en el sistema como
  historial, en la pantalla donde se configura el vencimiento. Hoy los ejecutivos
  lo tienen en su computadora. Por ahora permitir subir cualquier PDF (validar
  después si pasa por algún cifrado). Elian pidió un archivo de muestra.
- **Dar de alta permisos a nivel entidad/compañía** (módulo Entidades →
  Permisos) con historial de vigentes/vencidos, para que cualquier ejecutivo que
  maneje ese cliente los consulte sin pedírselos a otro compañero.
- **Estado en color:** verde = permiso activo/vigente, rojo = vencido.
- **Descripción de mercancía vs de fracción:** se muestran dos descripciones —la
  de la tarifa/fracción y la de la mercancía. La que debe mostrarse es la **que
  coloca el clasificador** (la de la mercancía), no la de la fracción. Se guardan
  ambas. El **Nico** también debe aparecer.
- Tipo de permiso: no hay listado oficial descargable, pero son limitados
  (registro sanitario, permiso previo, aviso automático…) → armar el catálogo.
  Régimen: normalmente **definitivo**, salvo que se requiera la clave de
  pedimento. Poner **placeholders** en todos los inputs.
- **Proceso de pruebas:** el módulo aún no está en staging; Elian manda el link
  al día siguiente por la tarde. Salvador y Geovanny **revisan estos días y
  levantan tickets**; se retoma el **viernes 4–6 pm**. Salvador necesita reset de
  password (mismo usuario/contraseña que SIGAC 2; Miguel apoya).

### B. Formulario de Movimiento de salida (revisión campo a campo)
- **ETD:** se toma por default del ETD capturado en la referencia; editable.
- **Hora de salida real:** no la modifica el ejecutivo; se llena
  **automáticamente** cuando almacén digitaliza la salida.
- **Peso del embarque y unidad de peso:** se traen del movimiento de entrada;
  editables (bodega los coloca en subdivisión; el ejecutivo puede recalcularlos
  contra documentos —BL/packing, peso por tarima—).
- **Tipo de material:** hoy solo Nestlé (sus categorías: materia prima, sólidos,
  lácteos…). Se hace **configurable a nivel cliente** (en Configuraciones,
  gestionado por **compliance**), y se **mueve al movimiento de entrada** (no va
  en salida).
- **Verificación:** se **elimina** del movimiento de salida — no se hace
  verificación en la salida (Yolanda lo confirmó); el campo de "verificación
  extra" que pidió almacén es solo para la **entrada**.
- **Tipo de flete:** se **elimina** del movimiento de salida (no manejan
  pagado/por pagar en salidas).
- **Transportista (Carrier):** se **mueve** para quedar antes de "datos de
  cruce". Se agregan **CAT y SCAC a nivel entidad transportista** (config de
  compañía, rol transportista) junto con nombre, RFC, contactos, teléfono; al
  seleccionar el transportista se jalan sus datos, editables. (Hoy CAT/SCAC no
  se guardan a nivel compañía — confirmado por Fernando.)
- **Documentos y ubicaciones** (DBL, tracking, país origen/destino, ubicación
  origen/destino): se **elimina** el apartado en el tab de transporte del
  movimiento de salida (no se usan).
- **Guías:** solo aplican a ferrocarril → se **quita el tab de guías** en el
  movimiento de salida **terrestre** (se conserva en entrada).
- **Bultos:** se **elimina el tab de bultos** (no se añaden bultos a mano). En su
  lugar, junto al peso del embarque (tab transporte) se muestra la **cantidad
  total de bultos** derivada de los movimientos/subdivisiones seleccionados, con
  **drill-down** (clic) para ver el detalle hasta donde exista (nivel bulto →
  embalaje exterior → interior). Es **informativo**; almacén registra el detalle.
- **Contenedores:** se **elimina el tab de contenedores** en salida terrestre
  (solo hay **un contenedor por movimiento de salida terrestre**); se mueven los
  tres campos —tipo de contenedor, número de contenedor, número de sello— junto a
  "identificación del transporte". Estos datos los pone el **ejecutivo** (se los
  da el transportista/cliente); para cajas directas (que no se descargan) se
  pueden jalar desde la entrada. Carlos revisará si se cruzan con alguna cita para
  jalarlos.

### C. Reordenar el flujo operación ↔ movimiento de salida (decisión mayor)
- **Problema:** hoy el movimiento de salida es prerrequisito para generar la
  operación, pero el ejecutivo **no quiere que la mercancía "salga"** al crear el
  movimiento de salida: necesita generar operación, pedimento, COVE, document,
  validaciones y **solicitud de fondos** —incluso pagar el pedimento— **antes**
  de mandar la instrucción real de salida a almacén. Obligar a crear el
  movimiento de salida primero **agrega un paso** en vez de quitarlo.
- **Decisión:** la **operación se genera a partir del movimiento de entrada o de
  subdivisión** (de ahí se jalan los datos), no del movimiento de salida. El
  **movimiento de salida se vuelve un paso dentro del flujo de operación**,
  disparado por un botón **"Solicitar salida"** (análogo al de "solicitud de
  fondos") que abre un modal con los pocos campos que quedan (identificación del
  transporte, tipo/número de contenedor, número de sello). La **orden de carga**
  a almacén se envía **solo cuando el ejecutivo lo indica** (evento orden de
  carga); bodega no hace nada hasta esa señal. Ángel diseñará la nueva pantalla
  de operación y la mostrará; se retoma el viernes.

## Lógica de negocio
- **Regla octava (contexto, Geovanny/Limón):** la Secretaría de Economía autoriza
  importar cierta mercancía con determinada(s) fracción(es) bajo un aviso/permiso
  con **vigencia** y **cantidad**; se declara a nivel partida (donde va el N3 se
  declara C1 + regla octava + firma de descargo + cantidad) para que a nivel
  central se vaya **descargando** el saldo. Un mismo permiso puede listar varias
  fracciones. Cuando se agota (cantidad o tiempo) hay que renovar con un **nuevo
  folio** (no se renueva el mismo). Caso GKN: el importador manda el formato, se
  sube en Entidades del cliente y aplica a todas las sucursales/aduanas por donde
  entre esa fracción (Laredo, México, Veracruz, Manzanillo…). GKN se maneja por
  **pedimento consolidado**, por lo que el saldo real no se conoce hasta el cierre
  semanal; el cliente puede agotarlo en días.
- **Subdivisión / bultos:** cuando llegan varias facturas en una entrada y solo se
  saca una, la operación en Nuevo Laredo no sabe físicamente en qué bulto está
  cada pieza; por eso hace falta un **paso intermedio de subdivisión** antes de la
  salida. El **movimiento de subdivisión** se asocia a un movimiento de entrada,
  lo parte en dos (o más) y de esas partes se jala al movimiento de salida. Un
  movimiento de entrada que ya tiene subdivisiones **no puede escalarse directo**:
  se escalan sus subdivisiones. Regla de contabilidad: **no se puede subdividir
  más de lo que entró**. En almacén, al recibir el formato de subdivisión,
  **regresan a verificación** con referencias guion-1/-2/-3 y **re-verifican
  bultos** (proceso actual, confirmado por Daniel Peña).
- **Cola de trabajo de almacén:** una sola cola para todos los warehouse (todos
  pueden hacer todas las actividades), divisible por tipo/perfil si se necesita.
  Pendiente validar con Yoli/Limón cómo ve almacén los movimientos de
  entrada/salida/subdivisión/reexpedición (app web vs móvil).

## Tareas
> Detectadas, **no creadas** (a petición del usuario). Agrupadas por bloque; las
> sub-viñetas son candidatas a ticket individual.

**Módulo Permisos (Clasificación) — Elian:**
- Renombrar "Regla octava" → "Permisos"; unificar tipos de permiso en un listado
  con filtro por tipo.
- Agregar columna "clave" del permiso (del Anexo 22, págs. 115–117).
- Completar migración desde SIGAC 2 de régimen y cuotas/saldos (editables).
- Mostrar saldos (cantidad/valor/vigencia) con indicador y alerta; validar contra
  registro al capturar; bloquear/alertar si vencido o por vencer.
- Mostrar al ejecutivo solo permisos vigentes en la selección.
- Permitir subir/adjuntar el PDF del permiso (digitalización) en la config de
  vencimiento.
- Dar de alta permisos a nivel entidad/compañía con historial vigentes/vencidos.
- En tab Mercancías: mostrar la descripción del clasificador (no la de fracción),
  guardar ambas y mostrar el Nico.
- Placeholders en todos los inputs; estado verde para permiso vigente.
- Subir el módulo a staging y notificar a Salvador/Geovanny; reset de password de
  Salvador (Miguel).

**Formulario Movimiento de salida — Ángel:**
- ETD default de la referencia (editable); hora de salida real automática al
  digitalizar salida en almacén; peso/unidad desde la entrada (editable).
- Tipo de material configurable por cliente (compliance) y movido a movimiento de
  entrada.
- Eliminar campo de verificación en salida.
- Eliminar tipo de flete en salida.
- Agregar CAT y SCAC a nivel entidad transportista y jalarlos al seleccionar;
  mover el campo transportista (Carrier) antes de datos de cruce.
- Eliminar apartado de documentos y ubicaciones (DBL, tracking, país/ubicación
  origen-destino) del tab transporte.
- Quitar tab de guías en salida terrestre.
- Eliminar tab de bultos; mostrar cantidad total de bultos junto al peso con
  drill-down informativo.
- Eliminar tab de contenedores en salida terrestre; mover tipo/número de
  contenedor y número de sello junto a identificación del transporte.

**Flujo y movimientos — Ángel:**
- Implementar el movimiento de subdivisión (asociado a entrada; escala
  subdivisiones a salida; no exceder lo que entró).
- Reordenar el flujo: generar la operación desde el movimiento de entrada/
  subdivisión; movimiento de salida como paso "Solicitar salida" (botón/modal)
  dentro de la operación; enviar orden de carga a almacén solo a indicación del
  ejecutivo. Rediseñar la pantalla de operación.

**Validaciones pendientes (compromisos, no dev):**
- Viernes: Salvador/Geovanny prueban el módulo de permisos y levantan tickets.
- Viernes/semana: validar con Yoli/Limón cómo ve almacén los movimientos
  (entrada/salida/subdivisión/reexpedición) y la cola de trabajo (web vs móvil).

## Tickets
- (ninguno con ID; se levantarán durante las pruebas de estos días)

## Temas descartados por irrelevantes
- Saludos, despedidas y agradecimientos.
- Problemas técnicos del meet (audio cortado, "no te escuché", "estás tirando",
  gente entrando/saliendo de llamada, reset de sesión).
- Muletillas y repeticiones de la transcripción automática.
