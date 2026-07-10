jul 7, 2026

## **Revision de pantallas flujo operación**

Invitado [German Castro](mailto:german.castro@carmi.com) [Angel Huberto Pulido Burgos](mailto:apulido@carmi.com)

Archivos adjuntos [Revision de pantallas flujo operación](https://calendar.google.com/calendar/event?eid=NnFtb3RzNDhyYWNsMTBzZWF0cjY2dWNiNXEgYXB1bGlkb0BjYXJtaS5jb20)

Registros de la reunión [Transcripción](https://docs.google.com/document/d/1IwKqTSnS6hfKuty8m1jQI4vAQXZ-88Jz2w2lzpZAdN8/edit?usp=drive_web&tab=t.wnjctqqdyst3) 

### **Resumen**

Optimización del flujo de creación de referencia mediante la reestructuración de la interfaz y la centralización de datos.

**Simplificación creación de referencia**  
Se eliminó la rigidez del asistente de documentos comerciales inicial para agilizar el proceso enfocándose exclusivamente en datos básicos. La sección de documentos comerciales se reubicará en el detalle de la operación.

**Estructura expediente y glosa**  
Se decidió organizar la interfaz mediante 2 pestañas principales: una para el expediente aduanero y otra para los datos glosados como fuente única de verdad. Este diseño permite validar discrepancias en tiempo real entre archivos.

**Integración de flujos operativos**  
Se estandarizó el uso de datos validados para generar declaraciones electrónicas de valor y pedimentos. El sistema permitirá gestionar discrepancias entre carga manual y documentos reales mediante indicadores de validación.

### **Próximos pasos**

- [ ] \[El grupo\] Ajustar flujo referencia: Ajustar el formulario de creación de referencias a 3 pasos en lugar de 4 para optimizar el proceso.

- [ ] \[El grupo\] Agregar Expediente Aduanero: Implementar una sección de expediente aduanero en el detalle de la operación para gestionar los documentos requeridos.

- [ ] \[El grupo\] Incluir orden compra: Agregar el campo de orden de compra en la pantalla de referencias para facilitar la búsqueda del cliente.

- [ ] \[Angel Huberto Pulido Burgos\] Combinar pestañas workflow: Fusionar el paso 4 del flujo de trabajo con la pestaña de mercancías.

- [ ] \[Angel Huberto Pulido Burgos\] Configurar herramientas DGO: Implementar las pestañas adicionales como herramientas o acciones dentro de la sección de facturas DGO.

- [ ] \[Angel Huberto Pulido Burgos\] Agregar campos valor: Integrar los campos necesarios para completar la manifestación de valor electrónica en el nivel de factura.

- [ ] \[Angel Huberto Pulido Burgos\] Implementar Happy Path: Desarrollar el caso sencillo definiendo los documentos GO y asegurando su funcionamiento inicial.

- [ ] \[Angel Huberto Pulido Burgos\] Configurar pestañas: Añadir una pestaña principal junto a una secundaria para visualizar tablas de referencia.

- [ ] \[Angel Huberto Pulido Burgos\] Renombrar secciones: Modificar los nombres de las secciones de gastos globales a incrementables y decrementables.

- [ ] \[Angel Huberto Pulido Burgos\] Etiquetar identificadores: Establecer la sección de configuración legal bajo la denominación de identificadores.

- [ ] \[Angel Huberto Pulido Burgos\] Evaluar navegación: Analizar el esquema del menú de clientes para considerar su implementación.

- [ ] \[Angel Huberto Pulido Burgos\] Rehacer interfaz: Reestructurar el prototipo de usuario según lo discutido y presentarlo.

### **Detalles**

* **Arranque de referencia y flujo de documentos**: Angel Huberto Pulido Burgos describe el proceso actual de creación de referencia, que incluye la subida de documentos, la extracción de datos por SEUS (un sistema similar al existente) y la validación por parte del usuario ([00:00:00](?tab=t.wnjctqqdyst3#heading=h.632daz6nwio6)). German Castro cuestiona la necesidad de un "wizard" de referencia que obligue a pasar por el paso de documentos comerciales de forma tan rígida, sugiriendo que la creación debería iniciar con "datos básicos" para agilizar el proceso, permitiendo que la auditoría y los documentos comerciales se gestionen posteriormente ([00:01:08](?tab=t.wnjctqqdyst3#heading=h.1jqs7ch9ym5c)).

* **Reubicación de documentos comerciales**: Se acuerda que la sección de "documentos comerciales glosados" debe retirarse del formulario de creación de referencia, ya que el paso actual es demasiado extenso y sugiere una obligatoriedad que no siempre aplica. German Castro propone pasar esta sección al "detalle de la referencia", donde se gestionaría junto con el expediente y un tablero de operaciones, manteniendo la creación de referencia rápida y centrada en datos básicos ([00:05:07](?tab=t.wnjctqqdyst3#heading=h.wzly4o6y1m49)).

* **Experiencia del usuario y contexto de pantalla**: Angel Huberto Pulido Burgos señala que los ejecutivos sienten que pierden el contexto de la pantalla inicial cuando el sistema los redirige a formularios aislados o nuevas ventanas, lo cual dificulta el flujo de trabajo ([00:06:27](?tab=t.wnjctqqdyst3#heading=h.tdu0kdnl9nu)). Se discute la necesidad de mantener la navegabilidad sin perder la visibilidad de los menús de tráfico, clasificación y fotos ([00:07:45](?tab=t.wnjctqqdyst3#heading=h.dxo32x5m2fbs)).

* **Interfaz de bandeja de entrada**: Angel Huberto Pulido Burgos presenta el diseño actual en Figma, que incluye una auditoría de SEUS para detectar faltantes, así como categorías de operaciones: en curso (automáticas), en espera (clasificación, almacén, arribos) y operaciones pendientes por identificar, como guías de terceros ([00:10:15](?tab=t.wnjctqqdyst3#heading=h.jfpslvob9mmw)).

* **Detalle de referencia y flujo de movimiento**: Se detalla el proceso tras la creación de la referencia, donde el sistema solicita documentos si es necesario o permite iniciar directamente ([00:11:05](?tab=t.wnjctqqdyst3#heading=h.nx0gv4av61tr)). Angel Huberto Pulido Burgos muestra el flujo de "detalle de referencia", que incluye información del cliente, tipo de tráfico (aéreo), documentos completos y el siguiente paso para crear el movimiento de entrada ([00:12:25](?tab=t.wnjctqqdyst3#heading=h.q7y1d33gaxax)).

* **Estructura de pestañas en la referencia**: German Castro sugiere reestructurar los nombres de las secciones para mayor claridad: "Expediente aduanero" (en lugar de documentos), "Mercancía" (donde se incluirían los datos comerciales glosados) y otras secciones de operaciones ([00:17:49](?tab=t.wnjctqqdyst3#heading=h.ljycqjpej96a)). Angel Huberto Pulido Burgos acepta consolidar estos apartados para evitar confusiones entre documentos generales y comerciales ([00:18:40](?tab=t.wnjctqqdyst3#heading=h.rk5xkbjeqxj6)).

* **Concepto de expediente aduanero**: German Castro explica que el "expediente aduanero" debe funcionar como un conjunto de documentos obligatorios definidos por el perfil del cliente, la operación y el perfil numérico ([00:19:50](?tab=t.wnjctqqdyst3#heading=h.a8igvoq0t6a5)) ([00:25:35](?tab=t.wnjctqqdyst3#heading=h.ewb8t6xjkxy0)). Este debe incluir un checklist con validaciones, fundamento legal y estado de glosa, similar al proceso de onboarding de clientes ([00:21:34](?tab=t.wnjctqqdyst3#heading=h.81dzz8xz8plz)).

* **Definición de glosa digital**: German Castro aclara que la "glosa digital" es el proceso de transformar los documentos en bruto en una representación digital, la cual sirve como fuente de verdad para la operación ([00:26:43](?tab=t.wnjctqqdyst3#heading=h.eortfpc03roi)). Este proceso incluye extraer datos relevantes y fundamentar por qué se solicitan ciertos documentos ([00:28:55](?tab=t.wnjctqqdyst3#heading=h.ih0z4jhzyazx)).

* **Documentos en operaciones vs. referencias**: German Castro enfatiza que el expediente aduanero debe ser visible tanto en el detalle de la referencia como en la operación ([00:34:38](?tab=t.wnjctqqdyst3#heading=h.z0iek7o5fzg8)). Este expediente debe contener todos los archivos utilizados, incluyendo documentos internos generados por el sistema (como XML del pedimento) y documentos de embarque, garantizando trazabilidad para auditorías ([00:32:52](?tab=t.wnjctqqdyst3#heading=h.awtdwr2iz7xx)) ([00:35:58](?tab=t.wnjctqqdyst3#heading=h.jcjvg0mgqmjx)).

* **Validación continua en el glosado**: Se establece que cada vez que se agrega un documento nuevo al expediente, el sistema debe realizar una re-validación ("glosar") contra el conjunto completo, asegurando que no existan discrepancias entre documentos como la factura, packing list o manifestación de valor ([00:38:29](?tab=t.wnjctqqdyst3#heading=h.djrccu510zcm)).

* **Organización de los tabs en la interfaz**: Se define que el flujo debe dividirse en tabs claros: uno para el "Expediente aduanero" (donde residen los documentos) y otro para "Datos glosados" (la fuente de verdad extraída), evitando confundir al usuario con pantallas innecesarias ([00:41:20](?tab=t.wnjctqqdyst3#heading=h.p4dhtzmhpfi6)).

* **Gestión de historial y versiones**: German Castro indica que el expediente aduanero debe permitir ver el estado de glosa (verde/rojo), fechas y versiones ([00:45:50](?tab=t.wnjctqqdyst3#heading=h.lore6br0yr52)). Respecto a las múltiples versiones de un documento (ej. varias facturas), se acuerda que no es necesario un historial por archivo, sino un historial a nivel expediente, permitiendo marcar cuál documento es el "primario" o activo, mientras se preserva el historial de las versiones anteriores para auditoría ([00:50:33](?tab=t.wnjctqqdyst3#heading=h.bqwjwpohwgn2)).

* **Lógica final de construcción de operación**: German Castro concluye que, una vez consolidada la fuente de verdad en los "datos glosados", se procede a la operación, utilizando estos datos validados para generar COVES, manifestaciones de valor electrónicas y el pedimento, asegurando así una validación sólida en cada etapa ([00:53:48](?tab=t.wnjctqqdyst3#heading=h.7pi3kqo38hf1)).

* **Definición y función de los Datos Glosados por Operación (DGO)**: German Castro y Angel Huberto Pulido Burgos definen los DGO como la "fuente única de verdad" dentro del sistema. El trabajo del glosador consiste en tomar el pedimento y verificar hacia atrás que todo cuadre con la documentación, fundamentando la operación. Establecen que existen dos pestañas fundamentales para la referencia: la de expediente aduanero y la de datos glosados por operación ([00:57:25](?tab=t.wnjctqqdyst3#heading=h.jxb5ph24gw8g)).

* **Estructura y nivel de detalle de los DGO**: Angel Huberto Pulido Burgos y German Castro discuten cómo los DGO consolidan datos crudos provenientes de diversas fuentes, incluyendo el paso cuatro del asistente, la pestaña de mercancías, y datos de facturación ([00:58:51](?tab=t.wnjctqqdyst3#heading=h.97yaoh2j7o2c)). Acuerdan que, aunque en sistemas como Sigac 2 se visualizan como "facturas", la estructura debe permitir gestionar datos a nivel de partida. Esto incluye información crítica como factores de conversión, pesos, fracciones arancelarias, identificadores, prosec y la aplicación de tratados de libre comercio, los cuales se aplican a nivel de partida y no únicamente a nivel de factura comercial ([01:00:19](?tab=t.wnjctqqdyst3#heading=h.6b512uaqfjil)) ([01:02:57](?tab=t.wnjctqqdyst3#heading=h.k816bkk7gzd)).

* **Proceso de registro manual y validación**: Angel Huberto Pulido Burgos plantea dudas sobre cómo manejar la carga manual cuando el cliente aún no envía la factura comercial. German Castro explica que los usuarios pueden crear una factura DGO manualmente con los datos disponibles, como el número de parte ([01:07:48](?tab=t.wnjctqqdyst3#heading=h.hgvayhw8j96e)). Cuando posteriormente se suba la factura comercial, el sistema comparará el JSON de la factura DGO con el JSON extraído del documento real; si no coinciden, el sistema marcará discrepancias en color rojo para que el usuario las corrija o valide ([01:08:53](?tab=t.wnjctqqdyst3#heading=h.8scuwtt4uuek)). German Castro enfatiza que el proceso de glosa digital debe ser validado y firmado por alguien antes de proceder con la operación para garantizar el cumplimiento legal y reducir el riesgo de multas ([01:11:03](?tab=t.wnjctqqdyst3#heading=h.x1su1vnc58nm)) ([01:14:01](?tab=t.wnjctqqdyst3#heading=h.oqm9vgkltu24)).

* **Implementación de la Manifestación de Valor Electrónica**: German Castro señala la necesidad de integrar campos adicionales dentro de la factura DGO para completar la manifestación de valor electrónica. Estos datos, que incluyen términos de pago, días de crédito y método de pago, deben configurarse a nivel de factura ([01:16:21](?tab=t.wnjctqqdyst3#heading=h.syv7q14zytr6)).

* **Gestión de documentos y mapeo en operaciones complejas**: Angel Huberto Pulido Burgos expresa preocupación sobre cómo asignar documentos específicos (como permisos de transporte) a pedimentos particulares cuando una sola referencia contiene múltiples facturas y operaciones ([01:17:20](?tab=t.wnjctqqdyst3#heading=h.oxl783cyrtmu)) ([01:22:09](?tab=t.wnjctqqdyst3#heading=h.8l9y781zu7fd)). German Castro sugiere no complicar el proceso inicialmente y enfocarse en el "Happy Path" (el camino principal), definiendo primero el caso sencillo para luego abordar los escenarios complejos ([01:26:31](?tab=t.wnjctqqdyst3#heading=h.2hwjni6ebqcv)). Acuerdan que el sistema debe permitir a los usuarios seleccionar qué documentos pertenecen a cada operación durante la generación del pedimento, utilizando la relación lógica que el DGO ya proporciona para prellenar esta información ([01:20:52](?tab=t.wnjctqqdyst3#heading=h.7gtevdmnnb88)).

* **Diseño de interfaz y transición de usuarios**: German Castro recomienda mantener dos pestañas: una con el diseño nuevo (más útil) y otra con el formato de lista tradicional al que los usuarios están acostumbrados, facilitando así la adopción del sistema ([01:32:15](?tab=t.wnjctqqdyst3#heading=h.31qy1brfyq4e)). Analizan la interfaz del paso cuatro, donde se da de alta la referencia, confirmando que debe permitir la visualización y gestión de múltiples facturas DGO ([01:40:37](?tab=t.wnjctqqdyst3#heading=h.s9djxuxth25v)).

* **Integración de herramientas dentro del DGO**: German Castro propone que funcionalidades como la conversión de mercancías, la configuración de identificadores y campos básicos deben integrarse como acciones dentro de la factura DGO, accesibles mediante un menú de acciones o un panel lateral (drawer). Esto permite que el usuario enriquezca la partida con toda la información necesaria sin cambiar de contexto ([01:42:26](?tab=t.wnjctqqdyst3#heading=h.l7orlgy61la8)).

* **Reestructuración de tabs y navegación final**: Angel Huberto Pulido Burgos y German Castro acuerdan renombrar ciertos apartados para mejorar la claridad: "costos globales" cambiará a "incrementables y decrementables", y "configuración legal" se denominará "identificadores" ([01:46:39](?tab=t.wnjctqqdyst3#heading=h.44ho8rqz1zfl)). Discuten la posibilidad de utilizar un menú lateral para organizar la gran cantidad de tabs, similar a otras secciones de la plataforma, evaluando si esto obstruye la navegación o mejora la experiencia de usuario, con el objetivo de finalizar estos ajustes en los próximos 4 días ([01:47:58](?tab=t.wnjctqqdyst3#heading=h.f5s94z9ixo22)).

*Revisa las notas de Gemini para asegurarte de que sean precisas. [Obtén sugerencias y descubre cómo Gemini toma notas](https://support.google.com/meet/answer/14754931)*

*Cómo es la calidad de **estas notas específicas?** [Responde una breve encuesta](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=ZnkBvhYazsz7vMMzx1iFDxIWOBABMgUIigIgABgBCA&detailid=standard&screenshot=false) para darnos tu opinión; por ejemplo, cuán útiles te resultaron las notas.*