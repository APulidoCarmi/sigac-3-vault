# Reunión 2026-08-11 — Revisión de pendientes para hacer una operación completa

## Asistentes

- Jesus Vega Portillo
- German Castro
- Angel Huberto Pulido Burgos
- Miguel Gomez
- Brenda Rentería
- Carlos Alexis Galaviz Rosas ("Charlie")
- Elian Shair Armendariz Puch
- Fernando Angel Lopez Soto ("Fer")

## Decisiones (con su porqué)

- Se necesita una operación real (con sellos digitales de un cliente) para poder correr todo el flujo de punta a punta hasta la validación del pedimento — es lo que detiene a Jesús en sus pruebas. Ángel ya le confirmó que tiene una para conectarla.
- El COV se genera hoy directamente contra la API de Bifrost, no desde la aplicación, porque hay candados: para hacer el COV se necesita primero la factura.
- Jesús ya modificó el front del stepper, pero solo lo mínimo viable para que funcionara con lo cambiado en backend — sin cambios estéticos. Falta reconciliar/mergear esos cambios con lo que tiene Ángel para que el flujo completo funcione junto.
- **Manifestación de valor — 4 casos posibles según el esquema del cliente** (orden oficial confirmado por Brenda):
  1. Carmi hace todo desde SIGAC (automatizado); el cliente solo recibe un link para revisar y aceptar.
  2. Carmi captura toda la información y el cliente entra al portal solo para firmar con su propia firma/sello.
  3. El cliente entra al portal de Carmi y hace él mismo todo el proceso (llena datos, sube firma, genera la manifestación).
  4. El cliente usa su propia herramienta externa y solo nos sube el acuse/folio — es el único caso completamente externo al sistema.
- Se decidió que Jesús se enfoque primero en automatizar el caso 1 (caso base) y que, una vez validada esa lógica, se ramifiquen los otros 3 casos — pero la tubería (pipeline) debe diseñarse desde ahora pensando en los 4 casos, no solo en el 1.
- El link que se envía al cliente para manifestación de valor debe reutilizar un mecanismo que ya se trabajó antes (Jesús no recuerda si fue él o "Fero"); no se debe rehacer desde cero. German pidió confirmar dónde está ese trabajo previo — se aclaró que no es el que hizo "Marcos" (que replicaba lo de Abraham, la versión de SIGAC 2, que German considera con muchas observaciones y no apta para reusar en SIGAC 3).
- En el caso 4 (cliente externo), no hace falta que suban un documento: el acuse se puede consultar directamente contra el Buzón del SAT usando el folio — Jesús confirma que ese flujo de consulta por folio ya existe. Actualmente lo único que sube el cliente es el folio, no un documento.
- La consulta contra Buzón requiere que el cliente haya dado de alta el RFC de consulta correspondiente (ej. Lic. Carmona u otra patente). Si no lo agregaron, no se puede validar — hay que detectar ese error y notificar automáticamente al cliente de que falta ese alta.
- El link de manifestación de valor (para los 4 casos) debe llevar seguridad vía código OTP enviado al correo de la persona a la que se compartió — sin pedir usuario/contraseña ni login. Brenda confirma que dar seguridad también da más confianza al cliente.
- Además del link, se debe enviar una notificación por correo cuando se timbra la manifestación de valor (con el acuse/documento para descargar) — mejora la experiencia del cliente, que de otro modo se queda esperando sin saber que ya se completó.
- Jesús debe loguear en su stepper de operación automática (incluyendo manifestación de valor) no solo que se hizo la manifestación, sino **por cuál de los 4 casos se hizo**, porque el tipo determina el cobro/precio que se le hace al cliente.
- **Trámite y despacho — 3 formas de generar el previo:**
  1. A partir de los datos de la factura (ya implementado; pendiente que Querétaro lo apruebe en la reunión de mañana 2026-08-12).
  2. Cuando llega un paquete con documentos físicos (factura en papel): se le toma foto/se escanea y se extraen los datos automáticamente, y de ahí sigue el flujo igual que el caso 1.
  3. Cuando no llega ningún documento: hay que armar un packing list a mano a partir de las etiquetas de la mercancía, ya sea tomando foto de las etiquetas y haciendo OCR, o capturando manualmente.
- El sistema debe decidir automáticamente cuál de los 3 casos aplica: si ya hay datos de factura, es el caso 1 sin preguntar nada; si no, sí hay que preguntar si llegaron documentos físicos con el paquete.
- El packing list es "como una factura sin precios": solo descripciones, cantidades y partidas.
- Carlos confirma que la verificación/previo ya está preparada para generar tanto el caso con factura como el caso sin factura — falta solo agregar un botón que detone el flujo cuando no hay factura. También hay una IA de Ángel que ya genera un packing list, aunque con pocos campos (solo número de parte, prácticamente) — se puede tomar como base.
- Reunión de trámite y despacho programada para mañana (2026-08-12) con Charlie y Elian, para mostrarles de nuevo el previo y recoger feedback.
- **Tarifario:** se decide tratarlo en una reunión separada y dedicada, estableciéndolo como prioridad, porque ya van tres reuniones previas sobre tarifario que no llegaron a nada — cada vez alguien (Miguel, Ángel o Fer) se distraía con otra tarea. Ángel señala que la lógica definida originalmente (basada en SIGAC 2) cambió y su backend de tarifario debe rehacerse/ajustarse a como quedó definido para SIGAC 3.
- Fer va a mover su reunión de tarifas de las 2pm a la 1pm y agregar a Miguel, Dunia y Yolanda, para definir las tarifas especiales de almacén (EE.UU.) que se manejan de forma manual.
- **Limpieza de la interfaz de Referencia antes de la demo** (objetivo: presentarlo en 2 semanas, fin de sprint — Querétaro, versión básica de aéreo): la instrucción de German es quitar todo lo que no funcione o no se vaya a usar en esta etapa, para no generar ruido ni mostrarle al cliente botones inservibles.
  - Se elimina el bloque "área de trabajo" del detalle de referencia, dejando solo la línea de tiempo — en la visita a Querétaro dijeron que se les hacía innecesario y no se ha vuelto a mencionar como necesidad en ninguna reunión posterior.
  - Se unifican en una sola línea del header: importación, valor (USD), antigüedad, actualizar y acciones (hoy están repartidos en dos niveles junto con borrador).
  - Se elimina la sección "resumen" del detalle de referencia, dejando solo la línea de tiempo.
  - La pestaña "instrucciones" del DGO se quita temporalmente de la demo: viene de un pedido de Alejandro Canales (mucho de lo que dispara la glosa son las instrucciones del cliente), pero no está definido cómo debe verse ni ser accionable todavía — se retoma como prioridad en el siguiente sprint, no se descarta.
  - La pestaña "citas" **no** se quita: Jesús confirma que ya se terminó (con PRs abiertos pendientes de aceptar); solo faltaba historial de que el ticket lo traía Jesús.
  - Carlos sugiere, en vez de ocultar el panel derecho completo, reducir márgenes (menú izquierdo, área de trabajo) y hacer el menú izquierdo colapsable — Ángel mencionó la idea de un menú de dos niveles como el de la consola de Google Cloud, pero no se profundizó ("no hay que perder tiempo en eso").
  - Los PRs de front de Jesús, Elian y Fernando quedan pendientes de aceptar hasta mañana: Carlos pidió no aceptarlos todavía porque metió cambios en el backend que está corrigiendo.
- Prioridad del sprint: hacer funcionar la versión básica de aéreo para Querétaro. La pantalla de contenedores + asociar guías con contenedores (marítimo) queda en el backlog de Ángel, sin prioridad inmediata, pero puede adelantarse si cabe en el sprint.
- Movimientos: la pestaña ya funciona para aéreo y terrestre; falta probar marítimo.
- DGO: los 189 campos del pedimento no se muestran todos porque el formato varía según el tipo de operación — hoy solo contempla una variante, falta cubrir las demás.
- Pedimento: el campo IGI está mal ubicado (en un lugar que no le corresponde). En agente aduanal/apoderado aduanal se debe mostrar la razón social de la patente (el licenciado, ej. Lic. Carmona) además de la empresa que factura los servicios (ej. el forwarder) — hoy solo se muestra esta última. Los cambios exactos se van a terminar de definir en el daily de mañana (2026-08-12) con Fer, quien está invitado.
- Falta un botón de información en el pedimento que abra la pantalla de detalle de todas las partes/bloques del pedimento (a vincular por Fer con el enlace de bloques que ya tiene Ángel).
- Falta un código QR en la sección de certificaciones del pedimento, que al escanearse lleve al link público de la referencia (para que el cliente consulte su referencia).
- Falta el campo de "cuenta aduanera" (cuenta de garantía del cliente, usada cuando hay que dejar dinero en prenda antes de importar/exportar). Brenda aclara que **no es el mismo campo replicado**: hay datos de cuenta y monto a nivel encabezado del pedimento, y otros campos —como el precio estimado— a nivel detalle de partida. Ángel va a generar el ticket para analizarlo, revisando primero cómo está resuelto en SIGAC 2 (cree que puede terminar quedando solo a nivel partida, pero falta confirmar).
- DGO: falta poder vincular, desde la parte de factura, los documentos ya subidos en el expediente electrónico con los datos capturados manualmente (para saber que esa factura corresponde a ese documento); y que al subir la factura desde el DGO, también aparezca reflejada en el expediente electrónico.
- Falta conectar la factura del DGO con Zeus.
- Las facturas de Querétaro vienen en formato "multifactura" (formato "Taiko"): una sola factura visual contiene varias partidas, pero cada partida se trata como una factura individual — hay que usar el endpoint de multifactura del API, no el de factura normal.
- Se debe agregar iconografía visual por tipo de tráfico junto al número de referencia (camión para terrestre, avión para aéreo, algo simple para marítimo) — que sea muy visible, no un detalle sutil.
- Ángel va a generar un ticket rápido para reordenar el header del detalle de referencia (ver arriba, limpieza de interfaz).
- Ángel debe reordenar el menú lateral en orden lógico — ya había un ticket de esto del sprint anterior.
- Recinto: todavía no está implementado. Jesús traía tareas de recinto pero no alcanzó a trabajarlas — se identificará el ticket y se asignará dentro de este sprint.
- Verificación (previo) ya está lista, salvo cambios menores que le pidan a Elian.
- Falta generar el formato de verificación como PDF descargable para enviarlo al cliente — no es prioritario ahora mismo, pero no se debe olvidar.
- Verificación/previo: agregar un checkbox para indicar si el producto/número de parte recibido es diferente al que indicaba la factura (ej. "esperaba pernos pero llegaron tornillos"), y un campo de observaciones generales sobre la descripción — pedido por Salvador/Carlos en el demo anterior.
- Verificación/previo: revisar que jale bien marca y modelo, y permitir capturar el número de serie a nivel pieza dentro de un bulto (un bulto puede contener N piezas y cada una puede tener su propia serie, que a veces sí viene y a veces no).
- Verificación/previo: agregar la modal de información de fracción arancelaria (la misma que ya existe en el front web) — reusar el componente existente, no duplicarlo.
- Verificación/previo: mostrar ahí mismo los documentos (factura) del expediente electrónico, reusando el componente de documentos ya construido, con previsualización (muchas veces no se van a poder imprimir, así que hay que poder verlos ahí).
- Ángel: pendiente preguntar al equipo de operación terrestre sobre el Excel de columnas de la bandeja de entrada (se le olvidó preguntar hoy y ayer; lo hará mañana).
- Usuarios/Login: el login ya funciona vía Identity/Odin. Falta que la sincronización de contraseña hacia SIGAC 2 funcione — hoy SIGAC 2 no responde al guardarla, así que se decidió permitir guardar solo en Odin aunque SIGAC 2 falle, para no bloquear el acceso del usuario.
- Equipos: solo ciertos usuarios seleccionados pueden crear equipos (no todos, para evitar caos). Si un usuario ya pertenece a un equipo y se le agrega a otro, se muestra una advertencia y se le pide justificar por qué entra a un segundo equipo.
- Carlos señala que el módulo de Equipos quedó ligado a usuario y sucursal, y que debería ser reutilizable, permitiendo que un usuario esté en más de un equipo — pendiente de revisión conjunta con Elian.
- El login ya toma las compañías dadas de alta en la base de datos de Odin (ya no apunta fijo a "la 20" de SIGAC 2).
- Pendiente: que Elian consuma directamente el servicio de Identity en Odin para el cambio de contraseña, en lugar de reimplementarlo — da retrocompatibilidad automática y evita duplicar lógica que ya existe, de cara a cuando se apague el "Identity" viejo.
- **Decisión de arquitectura — Contactos vs. Usuarios:** "Contactos" es un concepto heredado de SIGAC 2 para nombrar a los usuarios externos de la aplicación (clientes y proveedores). En SIGAC 3 todos son "usuarios", diferenciados por una bandera interno/externo en la tabla de usuarios de Identity. German decide que **contactos y usuarios deben ser el mismo objeto/servicio por detrás**, aunque la UI se vea distinta según quién lo usa: calidad da de alta usuarios desde el perfil de la empresa (una empresa, varios usuarios), mientras que un gerente de cuenta asigna empresas desde el perfil del usuario (un usuario, varias empresas) — misma relación usuario↔empresa (con rol/permisos), vista desde dos lados distintos. Se requiere una reunión dedicada con Fer, German, Elian y Carlos para cerrar esta unificación, porque German no tiene visibilidad de cómo quedó finalmente lo que hicieron Fer y Elian del lado de contactos/perfil de cliente.
- Al dar de alta un contacto del lado del cliente, se debe generar automáticamente el usuario correspondiente (mismo dato, un solo paso) — Carlos confirma que así estaba funcionando originalmente, hay que revisar si se rompió.
- Se confirmó (Ángel se acordó a mitad de la reunión) que el webhook de aceptación de solicitud de fondos ya existe: cuando Tony aprueba la solicitud en SIGAC 2, el webhook avisa a SIGAC 3 para continuar el flujo del stepper en Temporal.
- Cambio de cadencia de reuniones: las reuniones de los martes serán internas del equipo (para alinearse, evitar retrabajo), y las de los viernes quedan como demos, coincidiendo con el lanzamiento de cada sprint.
- Carlos señala como riesgo general que se está retrabajando código porque falta más coordinación al lanzar cada sprint en conjunto — motivo por el cual se instauran estas reuniones de martes.

## Lógica de negocio

- **Manifestación de valor — 4 esquemas de cliente** (nombres oficiales de cara al cliente, dados por Brenda):
  1. Carmi hace todo desde SIGAC.
  2. Carmi captura toda la información y el cliente entra al portal solo para firmar.
  3. El cliente entra al portal de Carmi y hace él mismo todo el proceso.
  4. El cliente usa su propia herramienta y solo entrega el acuse/folio.
  - El tipo de esquema (1-4) determina el cobro/precio que se le hace al cliente por el servicio — debe quedar logueado en el sistema.
  - Solo el esquema 4 es completamente externo al sistema de Carmi.
  - La validación del acuse contra el Buzón del SAT se hace por folio (no requiere que suban un documento), y solo es posible si el cliente dio de alta el RFC de consulta correspondiente (la patente que hace la consulta, ej. Lic. Carmona u otra distinta).
  - Cuando Carmi hace la manifestación de valor por el cliente (esquemas 1-3), el sistema pone automáticamente como RFC de consulta el de la patente que factura, y si es una patente distinta a la del Lic. Carmona, se agregan ambos RFCs para que no haya problema al consultar.
  - Todos los esquemas (1-4) envían un link al cliente, protegido con OTP por correo (sin usuario/contraseña).
  - Para clientes con esquema de "previa autorización" (ej. APL, Nalco, Ecolab), además del link se debe notificar por correo cuando se autoriza y cuando se timbra — el link no necesariamente se cierra tras la aprobación, el cliente puede seguir viendo/descargando la manifestación desde ahí.
- **Trámite y despacho — 3 formas de generar el previo**, con selección automática: si ya hay datos de factura capturados, es el caso base (sin preguntar); si no hay datos, se pregunta si llegaron documentos físicos (factura en papel → extracción por foto/OCR) o si no llegó ningún documento (packing list armado a mano a partir de etiquetas, con opción de foto+OCR o captura manual). El packing list equivale a una factura sin precios: solo descripciones, cantidades y partidas.
- **Multifactura (formato "Taiko", usado por proveedores de Querétaro):** una factura con múltiples partidas se presenta como un solo documento, pero cada partida corresponde a una factura individual dentro del sistema — requiere el endpoint específico de multifactura en la API, distinto al de factura normal.
- **Cuenta aduanera / cuenta de garantía:** cuenta bancaria del cliente usada para dejar dinero en prenda antes de una importación/exportación. Tiene campos distintos a nivel encabezado del pedimento (datos de la cuenta, monto) y a nivel detalle de partida (p. ej. precio estimado) — no son el mismo dato replicado en dos niveles.
- **Agente aduanal / apoderado aduanal en el pedimento:** debe mostrarse la razón social de la patente (el licenciado que ampara la operación, ej. Lic. Carmona), no solo la empresa que factura los servicios al cliente (ej. el forwarder/carga que factura). Ambos datos son relevantes y van juntos.
- **Contactos = usuarios externos:** en SIGAC 2, "contactos" era la tabla usada para dar de alta a clientes y proveedores como usuarios externos de la aplicación, distinta de "personas" (usuarios internos/de Carmi). En SIGAC 3 todos son "usuarios", diferenciados por una bandera interno/externo en la tabla de usuarios de Identity — por detrás debe ser un único servicio/objeto, aunque la UI cambie según el flujo de alta (desde perfil de empresa o desde perfil de usuario). La relación usuario↔empresa se modela como una tabla puente usuario-empresa con rol/permisos por empresa; un usuario puede estar asociado a varias empresas.
- **Instrucciones del cliente y su relación con la glosa:** según Alejandro Canales, mucho de lo que dispara o guía la glosa son las instrucciones explícitas que da el cliente al ejecutivo (p. ej. si la operación debe ser importación o exportación). El objetivo a futuro es tener esas instrucciones capturadas de forma estructurada (dropdowns configurables), no solo como texto libre, para poder auditar automáticamente si la operación ejecutada corresponde con la instrucción dada (p. ej. detectar que se hizo una exportación cuando se instruyó una importación).

## Tareas — detectadas, no creadas

Por la [[reunion-sin-crear-tareas|preferencia registrada]] para este cliente, estas tareas quedan listadas aquí para revisión, sin darlas de alta en Jira ni en `Tareas.md`. Posible relación con backlog abierto marcada donde aplica.

**Manifestación de valor**
- Incluir en manifestación de valor los campos faltantes a nivel factura (definidos por Enrique) y a nivel movimiento (mapeo que Miguel le va a pasar a Ángel a partir de lo hecho en SIGAC 2) — responsable: Ángel. *Posible fusión* con SCRUM-542 (incrementables de factura para manifestación de valor), mismo tema general; a confirmar alcance exacto.
- Construir el pipeline de manifestación de valor para el caso 1 (automatizado), diseñado desde el inicio para soportar los 4 casos, con link reutilizable/mutable — responsable: Jesús.
- Agregar validación de RFC de consulta faltante al validar el acuse contra Buzón, con notificación automática al cliente del error — responsable: Jesús/Carlos.
- Loguear en el stepper de operación automática por cuál de los 4 casos se hizo la manifestación de valor (para efectos de cobro).
- Enviar notificación por correo al cliente cuando se timbra la manifestación de valor (con el acuse para descargar).
- Implementar seguridad OTP por correo en el link de manifestación de valor, reusando el servicio de correos/links ya existente (el mismo que usa Jesús para el reporte de gente).

**Trámite y despacho / previo**
- Implementar caso 2 del previo: extracción de datos por foto/OCR de factura física — responsable: Elian (revisando código existente).
- Implementar caso 3 del previo: packing list manual/OCR de etiquetas cuando no hay ningún documento — responsable: Elian, pendiente de validar alcance con German/PO.
- Agregar checkbox de "producto/número de parte diferente al de la factura" y campo de observaciones generales en el previo.
- Revisar captura de marca/modelo/número de serie a nivel pieza dentro de un bulto en el previo.
- Agregar modal de información de fracción arancelaria en verificación/previo, reusando el componente ya existente en el front web.
- Mostrar y previsualizar en el previo los documentos del expediente electrónico (factura), reusando el componente de documentos ya construido. *Posible fusión* con SCRUM-524/SCRUM-525 (integración de factura al expediente aduanero).
- Generar PDF descargable del formato de verificación para enviar al cliente (baja prioridad, no olvidar).

**Pedimento / DGO**
- Hacer que el DGO contemple todas las variantes de campos del pedimento (189 campos), no solo la variante actual — responsable: Ángel.
- Corregir ubicación del campo IGI y mostrar datos del propietario/licenciado (no solo quien factura) en agente aduanal/apoderado — a definir en detalle en el daily de mañana (2026-08-12) con Fer.
- Agregar botón de información en el pedimento que abra el detalle de bloques/partes del pedimento — responsable: Fer, vinculando con el enlace de bloques de Ángel.
- Agregar código QR en la sección de certificaciones del pedimento, enlazando al link público de la referencia.
- Agregar campo de "cuenta aduanera"/garantía a nivel encabezado y a nivel detalle de partida del pedimento — responsable: Ángel, revisar primero cómo está en SIGAC 2.
- Permitir vincular, en el DGO, documentos ya subidos en expediente electrónico con los datos manuales capturados en factura (y viceversa: que subir factura desde DGO también la refleje en expediente electrónico). *Posible fusión* con SCRUM-525 (ya abierto, describe justo este cruce `invoice_images` vs `reference_documents`).
- Conectar la factura del DGO con Zeus.
- Implementar manejo del formato "multifactura" (Taiko) usando el endpoint correcto de la API.

**Interfaz de Referencia (limpieza para demo)**
- Reordenar el header del detalle de referencia: unificar en una sola línea importación/valor/antigüedad/actualizar/acciones, quitar "área de trabajo" (dejar solo línea de tiempo) y quitar la sección "resumen" — responsable: Ángel (dijo que generaría el ticket).
- Reordenar el menú lateral en orden lógico — responsable: Ángel; nota: puede ya existir un ticket de esto del sprint anterior, a confirmar ID.
- Agregar iconografía visual (camión/avión/barco) junto al número de referencia según tipo de tráfico del movimiento — responsable: Ángel.
- Definir la pestaña "instrucciones" del DGO (dropdowns configurables en vez de solo texto libre) para poder auditar instrucción vs. operación ejecutada — pospuesto al siguiente sprint, no se pierde el hilo.

**Recinto y contenedores**
- Implementar pantalla/funcionalidad de Recinto — Jesús traía tareas de esto sin alcanzar a trabajarlas; falta identificar el ticket y reasignar en este sprint.
- Pantalla de contenedores + asociación de guías con contenedores para marítimo — backlog de Ángel, sin prioridad inmediata este sprint.

**Usuarios y Contactos**
- Consumir directamente el servicio de Identity de Odin para el cambio de contraseña (en vez de reimplementarlo) — responsable: Elian.
- Revisar y corregir el módulo de Equipos para que sea reutilizable y permita que un usuario pertenezca a más de un equipo — responsable: Elian/Carlos.
- Corregir que al dar de alta un contacto del lado del cliente se genere automáticamente el usuario correspondiente (un solo paso) — responsable: Elian, dentro del trabajo de unificación contactos/usuarios.

**Otros / bugs puntuales**
- Definir con Miguel qué cambio se necesita para arreglar la solicitud de fondos a nivel operación, que quedó rota tras un refactor — responsable: Ángel.
- Confirmar que se aceptaron los PRs abiertos de "citas" (Jesús) una vez Carlos corrija lo que rompió en backend.
- Preguntar al equipo de operación terrestre por el Excel de columnas pendiente de la bandeja de entrada — responsable: Ángel.

## Reuniones a agendar (mencionadas, no son tareas de desarrollo)

- Reunión dedicada de Tarifario (Ángel, Miguel, Fer) — prioridad, no soltar el hilo.
- Reunión de tarifas de almacén movida de las 2pm a la 1pm, con Miguel, Dunia y Yolanda — responsable: Fer.
- Reunión de trámite y despacho, mañana 2026-08-12, con Charlie y Elian invitados.
- Daily de mañana 2026-08-12 sobre cambios de pedimento (IGI, agente aduanal), con Fer invitado.
- Reunión dedicada para unificar Contactos y Usuarios — Fer, German, Elian, Carlos.

## Tickets

(Ninguno mencionado explícitamente por ID en esta reunión.)

## Temas descartados por irrelevantes

- Saludos, despedidas y comentarios de sobremesa al cierre de la reunión.
- Problemas técnicos del meet (micrófonos, silencios, "no te escucho").
- Comentario de Ángel sobre el diseño de menú de dos niveles "como la consola de Google Cloud" — se mencionó como idea de diseño pero se descartó explícitamente profundizar ("no hay que perder tiempo en eso").
