# Reunión 2026-08-11 — Revisión de pendientes para hacer una operación completa

## Asistentes
- Jesus Vega Portillo
- German Castro
- Angel Huberto Pulido Burgos
- Miguel Gomez
- Brenda Rentería
- Carlos Alexis Galaviz Rosas

## Decisiones (con su porqué)
- Jesús se enfoca primero en automatizar el **caso 1** (Carmi hace toda la manifestación de valor desde Sigac) y, con esa lógica base, se agregan después los otros tres casos. — Porqué: partir del caso base permite ramificar las demás situaciones (Ángel y Jesús coinciden en el enfoque).
- Aun implementando por fases, la "tubería" (pipeline de manifestación de valor) se debe diseñar pensando en los 4 casos desde el inicio, no solo en el caso 1. — Petición explícita de German para que Jesús no diseñe algo que luego no escale a los otros casos.
- El link que se envía al cliente para la manifestación de valor debe ser **mutable** (reutilizable/editable), no una implementación de un solo uso. — German lo marca como importante.
- No confiar en el flujo de manifestación de valor heredado de Sigac 2 (el que "replicaba lo que hizo Abraham", no el de Marcos) para Sigac 3: hay que rehacerlo. — Llevan 6 meses con esa versión en Sigac 2 y ha tenido muchas observaciones de clientes que piden algo "más rápido y más sencillo".
- Miguel le pasará a Ángel (documento o extracción con IA) todos los cambios de campos que agregó en Sigac 2 y a qué nivel los agregó, para que Ángel extraiga la lógica y la traslade a Sigac 3.
- Se va a levantar un ticket para el ajuste de backend del **caso 4**: recibir la manifestación de valor que sube el cliente externo, validarla, y al aprobarse continuar con el stepper.

## Lógica de negocio

**COV (Comprobante/Certificado, sobre API de Bfrost):** Jesús lo genera directamente contra la API de Bfrost; todavía no sobre la aplicación porque hay "candados" — para el COV se requiere la factura. Ángel es quien más conoce este tema.

**Campos faltantes de manifestación de valor:** Ángel aún no los ha incluido en Sigac 3 (están en su backlog). Según la reunión que tuvo con Enrique, esos campos van a **nivel factura**. Miguel confirma que en Sigac 2 los agregó tanto a nivel factura como a nivel "tráfico/referencia"; en el CIGAC 3, el equivalente de "nivel tráfico" es **nivel movimiento**. Conclusión: hay que modificar/mapear esos campos a nivel factura y a nivel movimiento.

**Los 4 casos de manifestación de valor según el tipo de cliente** (orden oficial confirmado por Brenda, el orden en que German los explicó inicialmente fue distinto):
1. **Carmi hace todo desde Sigac** — se envía un link al cliente con toda la información de la manifestación de valor, solo para que revise y acepte.
2. **Carmi captura toda la información y el cliente entra al portal para firmar** — el link que le llega al cliente incluye toda la manifestación de valor más un campo para subir su firma.
3. **El cliente entra al portal de Carmi y hace todo él mismo** — llena los datos, sube su firma, genera la manifestación de valor y continúa el flujo.
4. **El cliente usa su propia herramienta externa** y solo entrega el acuse a Carmi — es el único caso externo por completo al sistema; el flujo se retoma cuando el cliente sube el link/acuse.

**Validación del acuse (caso 4 y similares):**
- Lo que sube el cliente es solo el **folio** (confirmado por Brenda: "el puro folio"), no un documento completo — el documento de la manifestación de valor lo genera Carmi.
- El folio se valida contra el **Buzón** (SAT). Jesús ya tiene trabajado un flujo para consultar por folio.
- Requisito indispensable para poder consultar: el cliente debe haber agregado el **RFC de consulta** (p. ej. el del "licenciado Carmona") en sus sellos digitales ante el SAT. Si no lo agregan, Carmi no puede hacer la consulta.
- Caso de error: si el cliente no agregó el RFC de consulta, el sistema debe **detectarlo y notificar automáticamente** que no fueron agregados como RFC autorizado — para evitar que el cliente asuma que la manifestación quedó validada cuando en realidad la consulta fallará (pedido explícito de Carlos Alexis, confirmado por Brenda).
- El caso 2 (cliente sube acuse) se valida con Carlos ("Zeus" en la transcripción, probablemente error de transcripción); si se aprueba, continúa el flujo del stepper.

**Backend de Jesús — estado actual:**
- Ya soporta usar los sellos digitales ya subidos, o los que vengan en el payload, para BFR/Bfrost.
- Ya está listo para el caso en que el cliente sube directamente el acuse (sin que Carmi genere la manifestación), incluyendo guardar el documento donde corresponde y correr el proceso de validación.

**Frontend (stepper de manifestación de valor):** Jesús hizo cambios mínimos viables sobre el stepper existente para que funcione con lo modificado en backend — sin cambios estéticos, el flujo visual quedó igual. Duda abierta: si estos cambios de Jesús necesitan mergearse con los que tiene Ángel en esa misma parte.

**Cierre de manifestación de valor (validación contra Buzón):** el flujo de validar el folio contra Buzón ya existe (backend de Jesús). Falta el último paso: cuando no se puede validar porque el cliente no agregó el RFC de consulta, decírselo al cliente y detener/redirigir el stepper en consecuencia (ver bloque anterior). Con eso, backend y frontend de manifestación de valor están alineados.

**Bloqueante para las pruebas de Jesús:** para llegar a la validación del pedimento, Jesús necesita una operación real de un cliente con sellos disponibles, para poder generar facturas, pasar e-documents, pasar COVE y llegar a manifestación de valor. Es lo que le pidió a Ángel al inicio de la reunión.

**Coordinación stepper front/back (Ángel y Jesús):** Jesús trae el stepper y se está asegurando de conectarlo con backend; quedó pactado que Jesús lo probaría manual (avanzando paso a paso) mientras Ángel se encarga de la automatización — pero Jesús ya lo dejó automatizado hasta el proceso de pago (ahí quedó manual a propósito, con un botón para enviar a pagar). German pide que Ángel y Jesús se reúnan, sinceren el listado de pendientes y decidan explícitamente quién hace qué, para que no haya piezas que ambos asuman que le tocaban al otro.

**Servicio de envío de correos/links:** ya existe un servicio para lanzar correos con links (trabajado por Carlos), y ya lo usa Jesús (lo utilizó para el reporte de la gente). German pide que todos usen ese mismo servicio, incluido para el envío de la manifestación de valor.

**Seguridad del link de manifestación de valor:** debe llevar seguridad (Brenda: da más confianza al cliente). Mecanismo acordado: no crear usuario/contraseña, solo pedir el correo al que se compartió el link y enviar un **código OTP** para confirmar que quien entra es la persona correcta.

**Notificaciones al cliente:** los 4 casos (no solo 2 y 4) reciben link. Para clientes con esquema de autorización previa (ejemplos mencionados: APL, Nalco, Ecolab), además del link hay que notificarles cuando ya se autorizó y cuando ya se firmó/timbró (con el acuse para descargar) — mejora la experiencia del cliente. El link no necesita cerrarse tras la aprobación: el cliente puede seguir viendo/descargando la manifestación desde ahí, y adicionalmente se le manda el acuse por correo.

**Sellos para firmar cuando el cliente no tiene los suyos (caso de firma por Carmi):** cuando el cliente firma pero Carmi no tiene sus sellos, no se guardan esos sellos; se consulta usando el número de operación con la firma del licenciado Carmona o de la patente que se haya agregado como RFC de consulta. Brenda: como Carmi prellena la información, en automático se pone el RFC de consulta de la patente, y si es una patente distinta a la de Carmona se agregan ambos RFC — así no hay problema para la consulta.

**Logs del stepper de manifestación de valor:** Jesús debe loguear no solo que se hizo la manifestación de valor, sino **por cuál de los 4 casos/tipos se hizo** — porque el cobro cambia según el tipo (1, 2, 3 o 4).

**Trámite y despacho — 3 casos para armar el "previo" (Elian):**
1. **A partir de los datos de la factura** (caso ya construido, visto en demo). Pendientes detectados en el demo:
   - Campo de **observaciones a la descripción** por partida (pedido por Salvador).
   - **Checkbox para indicar discrepancia de producto**: si el número de parte que llegó es diferente al que indicaba la factura (ej. "esperaba pernos pero llegaron tornillos") — pedido por Carlos Alexis.
2. **Llega la caja con documentos físicos (factura) pero sin datos capturados**: hoy el personal toma foto/escanea la factura y escribe/calca el previo a mano sobre ella. Se quiere tomar foto de esa factura, extraer los datos automáticamente, y continuar el flujo igual que en el caso 1.
3. **No hay ningún documento**: hay que armar un **packing list** desde cero. Se leen las etiquetas de la mercancía, con dos opciones: (a) tomar foto de las etiquetas y hacer OCR para extraer datos, o (b) llenarlo a mano.

**Detección automática de caso (pregunta de Elian, resuelta por German):** no se le pregunta al operador qué caso es — se infiere: si ya hay datos de factura, es el caso 1 directo; si vienen documentos en el paquete pero sin datos, se le pregunta/confirma y se va por el caso 2 (foto + extracción, luego sigue como el caso 1).

**Definición de packing list (para Elian):** es como una factura sin precios — solo descripciones, cantidades, y detalla todas las partidas.

**Estado de avance ya existente (aportado por Carlos Alexis y Ángel):**
- Ya existe un formato de packing list en la IA de Ángel.
- La verificación/previo ya está preparada para generar una verificación tanto si hay factura como si no la hay — Carlos ya lo había mencionado el viernes anterior; solo faltaría agregar un botón que detone ese tipo de previo cuando no hay documento.
- En la parte donde se genera la factura (sección de e-documents/DODA — "DGOS" en la transcripción), ya existe un apartado para crear packing list, aunque con pocos campos.

**Fuentes de referencia para el formato de packing list:** Elian pidió ver un ejemplo real para saber exactamente qué se extrae y no dejar campos fuera. Charlie tiene formatos que trajeron de Querétaro de cómo trámite y despacho llena esto hoy; Brenda tiene expedientes con ejemplos de packing list en Drive y se los compartirá a Elian.

**Formato de packing list existente en DODA/e-documents:** trae pocos campos (según Ángel, básicamente número de parte); Carlos Alexis confirma que le falta información y que hay que rediseñarlo para que sea usable en más flujos, no solo el actual — dice que buscará el formato y se lo pasará a Elian. Plan de Elian: revisar el código, implementar primero el caso 2 (foto+extracción de factura) que pidió German, y sobre el caso 3 (packing list desde cero) preguntar a German o al PO una vez revisado el código, para validar qué se está extrayendo. Mañana hay una reunión de trámite y despacho (invitados Charlie y Elian) para mostrarles el previo otra vez y recoger feedback.

**Pendientes adicionales de verificación/previo (Carlos Alexis):**
- Revisar que jale bien la captura de **marca, modelo y número de serie**.
- Cuando el previo se inicia por **bulto** y el bulto contiene **piezas en serie**, verificar que se pueda capturar la serie a nivel pieza en el detalle (Ángel ya la capturaba en su momento; puede haber n cantidad de series o ninguna). Agregar un check de "sí venía serie" cuando aplique.
- Hay un componente/modal (probablemente en Recepción) que muestra toda la información de la **fracción arancelaria**; hay que agregarlo también en verificación y previo, reutilizando el componente existente (no duplicarlo) — importante mostrarlo igual que en el front web.
- En el previo debe poderse ver fácilmente la **factura y los documentos** asociados, reutilizando el componente de documentos ya construido (parte del mismo "expediente" en el que trabaja Carlos Alexis), con previsualización (no siempre se pueden imprimir).

**Facturación — estado (Miguel):** aún no ha revisado cómo armar el flujo de facturación "hacia atrás"; tiene los servicios de "pindrado" (timbrado) y facturación como servicios puros, pero falta generar los XML de las facturas y de los pagos. Necesita conocer mejor la estructura completa antes de avanzar.

**Tarifas — estado (Fernando/"Fer"):** tiene nuevos elementos pero le faltan los elementos necesarios para generar las facturas y para consultar tarifas a futuro; se necesita una reunión dedicada para continuar.

**Tarifario — decisión de retomarlo como prioridad:** el backend que Ángel había hecho para que los "logs" (¿flujos?) buscaran tarifas se tiene que **rehacer**, porque se construyó siguiendo la lógica de Sigac 2 y esa lógica cambió para Sigac 3. Ya hubo ~3 reuniones previas sobre tarifario que no se cerraron porque cada quien (Miguel, Ángel, Fer) se distrajo con otras tareas. Decisión: hacer una reunión separada dedicada a tarifario y no soltar el hilo hasta cerrarlo, estableciéndolo como prioridad.

**Tarifas especiales de almacén (aportado por Fernando, vía Adriana):** hay un stopper porque, además de los tarifarios normales que maneja Adriana, existen tarifas manuales de bodega para almacén (EE.UU., especiales). Adriana preguntó si Miguel ya había trabajado antes una sección de tarifas para almacén. Fernando va a mover la reunión de las 2pm a la 1pm y agregar a Miguel, Dunia y Yolanda para que indiquen qué servicios usan más y definan las tarifas especiales. (Aclaración de German: la reunión ya sostenida con Dunia fue para arreglar algo de Sigac 2 a petición de Armando, no esto — son reuniones distintas).

**Revisión en vivo de la interfaz de Referencia (Ángel comparte pantalla), pantalla de listado:**
- Botones y filtros de la vista de listado funcionan correctamente; sin pendientes de mejora ahí.
- Bandeja de entrada: sigue pendiente la validación de columnas Excel con el equipo de operación terrestre — Ángel no ha podido preguntarles (se le pasó hoy y ayer), lo hará mañana.
- Columnas del listado ya validadas por el equipo, sin cambios pendientes salvo uno ya aplicado (agregar "creado por").
- En Acciones: falta probar "crear operación a partir de selección de referencias" (la funcionalidad ya está implementada, falta prueba) y revisar las validaciones que se meten a nivel individual tras el cambio de DODA y operación.

**Limpieza de la pantalla de detalle de referencia (pedido explícito de German):** preparar la vista para presentarla en el sprint de aquí a dos semanas, sin botones o elementos que no funcionen o no le sirvan al cliente ("genera mucho ruido"). Encabezado con valor USD, antigüedad, actualizar, acciones, borrador e importación se mantienen en uso. Decisiones tomadas en vivo:
- El bloque de **"área de trabajo"** se elimina, dejando solo la línea de tiempo — en la visita de Querétaro dijeron que se les hacía innecesario y no ha vuelto a surgir como necesidad en reuniones posteriores.
- Reestructurar niveles del encabezado: quitando borrador e importación de su nivel actual, dejar **importación, valor, antigüedad, actualizar y acciones en una sola línea**, al mismo nivel que borrador.
- Ángel generará un ticket para hacer este ajuste de UI rápidamente.

**Detalle exacto del reacomodo de header de referencia (precisión de Ángel sobre el ticket anterior):** en el header hay borrador y el tipo de referencia; abajo, en otra línea, van valor, antigüedad, botón actualizar y acciones. Todo eso se deja a un solo nivel, dejando aparte solo tipo de referencia y valor (necesarios donde están).

**Resumen de detalle de referencia:** en el mismo ticket de reacomodo de header (no se genera ticket aparte), se cambia la sección "resumen" para que solo se muestre la línea de tiempo — el resto de "resumen" se elimina.

**Idea de menú estilo consola de Google (Ángel), aparcada:** Ángel propuso un menú de dos niveles como el de Google Cloud Console (el menú base persiste y se anidan submenús por sección, ej. dashboard) para resolver el exceso de márgenes/espacio que señaló Carlos Alexis en el panel derecho. German no conocía esa referencia; se acordó no invertir tiempo ahí ahora — queda como idea de diseño sin decisión ni ticket.
- Alternativa más simple que sí propuso Carlos Alexis: quitar márgenes (pegar el menú izquierdo más a la izquierda, quitar el margen del área de trabajo/referencia) y agregar un control para colapsar el menú lateral y ganar espacio limpio.
- Hay un ticket ya existente del sprint anterior para acomodar el orden del menú lateral en un orden lógico — sigue en pie, sin cambios.

**Recap de pantallas de referencia (recorrido en vivo, para saber qué falta antes de la demo de Querétaro):**
- **Movimientos:** la pestaña ya funciona; probada en la reunión pasada y también por Elian junto con lo de previos. Confirmado funcionando para **terrestre y aéreo**; **marítimo aún no se ha probado**.
- **Pendiente para marítimo:** falta una pantalla de **contenedores** (análoga a la ya construida de guías) y la asociación entre guías y contenedores, para poder administrarlos — no es prioridad de este sprint, pero si alcanza tiempo, adelante. La prioridad actual es dejar funcionando la versión básica de **aéreo para Querétaro**.
- **Expediente:** lo está trabajando Charlie (Carlos Alexis), sigue en curso.
- **Instrucciones:** pendiente y **sin definir del todo**. Contexto de negocio (aportado por Carlos Alexis, referenciando feedback de Alejandro Canales): gran parte de lo que dispara o guía la glosa son las instrucciones que da el cliente al ejecutivo. No es solo cambiar el campo de texto libre existente — a futuro se necesitan dropdowns/parámetros que configuren el tipo de operación (import/export, etc., parámetros exactos aún no definidos, hay que revisar el "RIT" y una reunión previa que Ángel pidió que le compartieran) para poder **auditar** la instrucción contra lo que realmente se está haciendo (ej. detectar que se instruyó una importación pero se está tramitando una exportación). Por ahora Canales solo pide poder **guardar el texto de la instrucción tal como se dio** (no accionable todavía; a futuro se procesaría con IA para extraer acciones). Decisión para la demo del viernes: como no está definida la lógica completa y no puede quedar "inservible" en la demo, Ángel **quita esta sección por ahora** y se retoma en el siguiente sprint.
- **Citas:** decisión de **quitarla para la demo de Querétaro**, por ahora. Alcance original: solo mostrar las citas ya generadas desde la app (no generar citas nuevas desde aquí). En la presentación a Querétaro se acordó dejarla solo para terrestre, y luego se mencionó agregar también marítimo — pero como no ha vuelto a pedirse, se quita por ahora y se reincorpora si el cliente la pide o la extraña. Este alcance ya se trabajó y se **terminó** (Jesús lo confirma: los PRs ya se subieron y están pendientes de aceptar; Elian trabajó una parte separada de precios). Carlos Alexis pidió **no aceptar los PRs de front todavía** porque está corrigiendo algo que rompió en backend — se revisan y aceptan al día siguiente antes de una revisión conjunta. Decisión final: **no quitar la funcionalidad de citas del código**, solo dejar pendiente que se acepten los PRs.
- Jesús menciona que tiene más cambios pendientes por conectar con lo que tiene Ángel para completar el flujo de la operación (relacionado con lo discutido en bloques anteriores sobre stepper/manifestación de valor).
- **Operaciones:** ya está. Ángel está probando la solicitud del número de pedimento (ya funcionaba, hay un error menor por revisar, parece sencillo). Falta probar la **solicitud de fondos a nivel operación**.
- **Solicitud de fondos → Sigac 2:** el flujo apunta todavía hacia Sigac 2 (ya trabajado por Miguel y en su momento conectado y funcionando en esta interfaz), pero **se rompió con el refactor**. Ángel necesita que Miguel haga un cambio, pero aún no ha definido cuál es exactamente — queda pendiente que Ángel le especifique el cambio a Miguel.
- **Recinto:** **no está** construida. Jesús traía tareas sobre recintos pero no alcanzó a trabajarlas. Queda pendiente identificar el ticket y asignarlo dentro de este sprint.
- **Verificación:** ya está, salvo que a Elian le pidieran algún cambio (se revisa mañana).
- **DODA — pendiente que probablemente pida Elian:** generar el **formato de verificación en PDF descargable**, para poder enviarlo al cliente.

**Formato de verificación en PDF descargable (DODA):** German pide levantar un ticket para esto; no tiene prioridad alta ahora pero es importante no olvidarlo.

**Campos del pedimento en DODA (los 189 campos identificados):** Ángel ya los metió, pero el DODA **no muestra todos** porque el formato varía según el tipo de operación (una operación sencilla no necesita mostrarlos todos). Confirmado: el DODA **no contempla todavía todas las variantes** del pedimento — queda pendiente completarlo para las demás variantes/campos.

**Cambios pedidos por Enrique en el pedimento (a definir en el daily de mañana, Fernando invitado):**
- El campo **IGI** estaba mal ubicado — va en otra parte del formato, no donde está ahora.
- En agente aduanal / apoderado aduanal: se están mostrando los datos de la agencia (ej. "carga Forwarder") pero no los del **propietario/licenciado** (ej. licenciado Carmona). Aclaración de Brenda: normalmente se pone la razón social de la **patente** (la del licenciado) y, aparte, la empresa que factura los servicios (carga Forwarders) — son dos datos distintos que deben mostrarse ambos.
- Sigue en revisión conjunta con Fernando en el daily de mañana (tema "payment").

**Botón de información del pedimento:** falta que Fernando agregue, en la parte del pedimento, un botón de información que al hacer clic muestre la pantalla que Ángel ya construyó con todas las partes/campos del pedimento — Fernando debe vincularlo con el enlace de bloques que le comparta Ángel.

**Código QR en certificaciones del pedimento:** falta agregar un QR en la sección de certificaciones que, al imprimirse y escanearse, lleve al **link público de la referencia** donde el cliente puede consultarla. German levantará el ticket y lo detallará después.

**Pendientes en la sección de facturas del DODA:**
- Vincular, cuando Carlos Alexis termine el expediente, la subida de documentos desde el expediente con los **datos extraídos vs. datos manuales** capturados en el DODA — para saber que la factura subida corresponde a lo que se está capturando ahí. La factura subida desde el DODA también debe reflejarse en el expediente.
- **Falta conectar a "Zeus"** (integración/servicio, mencionado igual en bloques previos) la factura en la parte del DODA.
- **Formato "multifactura" de Querétaro:** las facturas que dio Querétaro para pruebas con "Zeus" venían en un formato de una sola partida que en realidad representa varias facturas (multifactura) — cada ítem es una factura distinta. Es un **endpoint distinto** en la API (endpoint de multifactura, no el de factura normal). Ángel ya tiene esto aclarado.
- Falta agregar en el DODA el campo de **cuentas aduaneras** (aportado por Fernando) — German aclara que es la "cuenta de garantía": una cuenta del cliente (banco del cliente) usada para dejar dinero en garantía antes de una importación/exportación. Actualmente no existe este campo. Discusión sobre a qué nivel va: Brenda aclara que en Sigac 2 va tanto a **nivel encabezado del pedimento** (datos de la cuenta, monto) como a **nivel detalle de partida** (con campos distintos, ej. precio estimado) — son campos diferentes, no el mismo dato replicado. Ángel va a generar el ticket y analizarlo de nuevo contra lo que existe en Sigac 2; su hipótesis inicial es que podría bastar con manejarlo solo a nivel partida, pendiente de confirmar.

**Iconografía visual por tipo de tráfico (pedido de German):** lo único que hoy cambia según el tipo de tráfico (terrestre/marítimo/aéreo) es la sección de **movimientos** (no la referencia en sí): terrestre maneja movimientos de entrada como antes, aéreo se genera a partir de guías. German pide agregar un ícono visible (camioncito, avioncito, barquito) junto al número de referencia para identificar el tipo de tráfico de un vistazo, no como algo sutil.

**Módulo de usuarios y equipos (Elian) — estado:**
- Login: ya funciona del lado de Odin. El primer ticket (sincronizar Sigac 3 con "Saco"/Sigac 2) se completó, pero al guardar la contraseña sincronizada en Sigac 2, este a veces no responde — Elian implementó que, aunque Sigac 2 no responda, la contraseña se guarde igualmente en Odin y se pueda entrar a Sigac 3.
- Equipos: ya está la asignación de un **rol** que permite solo a ciertos usuarios seleccionados crear equipos (no todos, para evitar caos). Si un usuario ya pertenece a un equipo y se intenta agregar a otro, el sistema lanza una **advertencia** y exige una **justificación** para el cambio.
- **Pendiente de revisar (Carlos Alexis):** el diseño actual de equipos quedó ligado a usuarios y sucursales; hay que revisarlo para que los equipos sean **reutilizables** y un usuario pueda pertenecer a más de un equipo — falla de conexión impidió cerrar el detalle en la reunión, queda pendiente revisarlo en el código por separado con Carlos Alexis.
- **Duda abierta de German:** si el rediseño de usuarios ya está aplicado al login general de toda la aplicación, incluyendo la barra superior (icono de usuario, avatar, icono de empresa y selector de empresas) — sin resolver al cierre de este bloque.

**Selector de compañías y usuarios por compañía (Elian) — estado:**
- Login: funciona salvo el guardado en Sigac 2 (ver bloque anterior); vía correo con código de verificación se puede cambiar contraseña y acceder a Sigac 3.
- Selección de compañías: ya está, y ya apunta a las compañías dadas de alta en la base de datos de **Odin** (ya no a Sigac 2), confirmado con German.
- Pendiente relacionado con el perfil de cliente de Fernando: gestionar usuarios por compañía de forma direccional.
- Carlos Alexis: solo falta aceptar el pull request para poder usar ya los usuarios de Sigac 3 en Odin.
- **Recomendación técnica de Carlos Alexis:** que Elian consuma directamente el servicio de **Identity** en Odin para el cambio de contraseña, en vez de reimplementarlo — da retrocompatibilidad y evita duplicar lógica que ya migrará cuando se elimine Sigac 2.

**Discusión de arquitectura: "contactos" (perfil de cliente) vs. "usuarios" — decisión de diseño:**
- Elian mostró un campo en el perfil de cliente donde se listan los usuarios de esa compañía y se les pueden asignar roles. Carlos Alexis señaló el riesgo: esto duplicaría el concepto de "contactos" ya existente, dando de alta la misma persona dos veces (en contactos y en usuarios).
- **Aclaración conceptual de German (importante para no repetir el error heredado de Sigac 2):** "Contactos" es un concepto de **Sigac 2** que se usaba para nombrar a los usuarios externos de la aplicación (clientes y proveedores); "personas" era la tabla de alta de usuarios en general. En **Sigac 3 todos son usuarios** — en la tabla de usuarios de Identity ya existe una bandera que distingue usuario **externo** (cliente) de usuario **interno** (Carmi).
- **Decisión:** contactos y usuarios con acceso deben ser **el mismo objeto/tabla por detrás**, nunca datos replicados. La UI puede diferir según quién lo usa y desde dónde (dos vistas del mismo dato), pero el mecanismo de fondo debe ser uno solo: una relación usuario↔empresa (con su rol/permisos en esa empresa).
  - Si es de **calidad**: entra a la empresa y da de alta ahí a las personas que pueden verla.
  - Si es **gerente de cuenta**: entra a la vista de usuario y le asigna a sus empleados qué clientes pueden ver.
  - Ambos flujos deben terminar en la misma estructura de datos (relación usuario-empresa), solo cambia la UI de entrada.
- Carlos Alexis confirma que el lado de "un usuario con varias empresas y su rol por empresa" **ya está construido así** (lo hizo él al inicio). Lo que falta es verificar que el mecanismo que armó Fernando para "contactos" use ese mismo mecanismo por detrás, solo visto desde el otro lado.
- Efecto esperado y deseado por German: al dar de alta un contacto del lado del cliente, debe **generarse automáticamente el usuario** (mismo dato, un solo paso) para que ya pueda hacer login — sin un paso separado de "crear contacto" y luego "generarle usuario". Carlos Alexis cree que así estaba construido originalmente y que habría que revisar si eso se perdió; si es así, es un ajuste menor ("cambiar dónde se guarda y dónde sale"), es la pestaña de contactos la que se ajusta, no la relación de fondo.
- Acción: German quiere una **reunión dedicada** para cerrar esto con Elian, Carlos Alexis y Fer(nando), porque German no tiene visibilidad completa de cómo quedó la modificación que hicieron ellos. La implementación final la hará Elian como parte de la planificación de usuarios, aunque toque también el perfil de cliente, porque es quien mejor conoce esa parte.

**Webhook de aceptación de solicitud de fondos (duda de Ángel, resuelta en la reunión):** Ángel no recordaba cómo el ejecutivo se entera de que una solicitud de fondos fue aceptada (en Sigac 2 era por correo) para poder continuar con el pago. German le recuerda que sí quedó definido un **webhook**: cuando "Tony" (persona/rol en Sigac 2 que aprueba) acepta la solicitud, Sigac 2 dispara ese webhook para que Sigac 3 continúe automáticamente el flujo del stepper en Temporal. Ángel confirma que ya lo recordó, que quedó hecho, y cree que Miguel ya lo conectó — pendiente de que Ángel lo verifique.

## Decisión de cierre — cadencia de reuniones
Para reducir retrabajo por falta de visibilidad entre lo que cada quien construye (señalado por Carlos Alexis: "muchas cosas ya están hechas... trabajamos [duplicado]"), German decide fijar una cadencia: **las reuniones de los martes son internas del equipo** (como esta, para alinear a todos en la misma dirección) y **las de los viernes se dejan como demos**, coincidiendo con el lanzamiento del sprint.

## Tareas
- (pendiente completar tras procesar toda la transcripción — ver reconciliación al final)

## Tickets
- (pendiente — ver: caso 4 de manifestación de valor; ajuste de niveles del encabezado + resumen de detalle de referencia (un solo ticket); recinto sin ticket asignado; PDF descargable de verificación; QR en certificaciones del pedimento; cuentas aduaneras en DODA)

## Temas descartados por irrelevantes
- Problemas técnicos del meet (micrófonos, "no te escucho", confusión de quién hablaba) en torno al minuto 00:53-00:54 y falla de internet de Elian cerca del minuto 01:14.
- Despedida y small talk de cierre ("voy a comer", "provecho", etc.) al final de la reunión.
