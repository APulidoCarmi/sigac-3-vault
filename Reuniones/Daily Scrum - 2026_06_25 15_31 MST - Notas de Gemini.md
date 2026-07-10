jun 25, 2026

## **Daily Scrum**

Invitado [Cesar Aguirre](mailto:caguirre@carmi.com) [Angel Huberto Pulido Burgos](mailto:apulido@carmi.com) [Enrique Lopez](mailto:enlopez@carmi.com) [Claudia Andrade](mailto:candrade@carmi.com) [Enrique Garza](mailto:cgarza@carmi.com) [Daniel Peña](mailto:dpena@carmi.com) [German Castro](mailto:german.castro@carmi.com) [Perla Lopez](mailto:plopez@carmi.com)

Archivos adjuntos [Daily Scrum](https://calendar.google.com/calendar/event?eid=MW5iOW9tb2s0Z2llZjNyYmh1MzA1NHUycjdfMjAyNjA2MjVUMjIzMDAwWiBhcHVsaWRvQGNhcm1pLmNvbQ)

### **Resumen**

Simplificación de flujos de importación mediante integración de inteligencia artificial y gestión dinámica de documentos digitales estructurados.

**Optimización de procesos operativos**  
El enfoque principal es simplificar la interfaz mediante inteligencia artificial y establecer un flujo de datos estructurado. Se requiere implementar listas de verificación dinámicas que se ajusten según los requisitos específicos del cliente.

**Validación y bloqueos sistémicos**  
La falta de revisión física bloquea procesos críticos como la validación de pedimentos, afectando la eficiencia operativa. El sistema debe permitir excepciones bajo autorización gerencial en casos donde se requiera despacho inmediato.

**Definición del flujo operativo**  
Se definió la ruta crítica para documentos, validaciones y pagos necesarios para completar la operación. Se estableció la creación de un esquema gráfico del flujo ideal para estandarizar la ejecución del sistema.

### **Próximos pasos**

- [ ] \[Angel Huberto Pulido Burgos\] Documentar flujo operativo: Refrescar y documentar los procesos operativos actuales para optimizar el diseno de la interfaz mediante inteligencia artificial.

- [ ] \[German Castro\] Realizar verificacion cruzada: Hacer una verificacion cruzada con Charlie en el momento en que el sistema de verificacion saque los documentos para asegurar que todo caiga en la referencia correctamente.

- [ ] \[Angel Huberto Pulido Burgos\] Analizar proceso: Crear un esquema del flujo de trabajo actual y presentarlo en la reunión diaria de mañana.

### **Detalles**

* **Introducción y objetivos de la reunión**: Angel Huberto Pulido Burgos convoca la reunión con el objetivo de simplificar la interfaz de usuario (UI) de la aplicación y crear un flujo de datos estructurado que se adapte mejor a las necesidades de operación. El equipo busca entender a fondo los procesos actuales, incluyendo las operaciones marítimas, aéreas y terrestres, con el fin de utilizar la inteligencia artificial para optimizar el diseño y la funcionalidad del sistema.

* **Ejemplo de operación de importación terrestre**: Perla Lopez presenta un ejemplo de una operación de importación terrestre del cliente NALCO. El proceso inicia cuando la línea de transporte solicita una cita al almacén proporcionando un número de BOL (Bill of Lading) y otros datos necesarios. Este paso es fundamental para evitar costos de almacenaje, ya que el almacén requiere documentación previa para recibir la mercancía.

* **Documentos obligatorios para la importación**: El equipo identifica la factura comercial y el "packing list" como los documentos básicos obligatorios para la mayoría de las importaciones. Perla Lopez aclara que, aunque otros documentos como el Certificado de Análisis (COA) o el certificado de origen pueden ser necesarios, su obligatoriedad depende de las especificaciones del cliente o de la naturaleza de la mercancía.

* **Concepto de lista de verificación dinámica**: German Castro explica que los requisitos de documentación no son estáticos, sino que resultan de una combinación de perfiles del cliente (MOMP), del número de parte, de la aduana correspondiente y de los tratados aplicables. El sistema debe gestionar un "checklist" dinámico que crece o se ajusta según el contexto de la operación. Por ejemplo, ciertos materiales como el acero o el aluminio activan requisitos específicos, como certificados de molino o avisos automáticos.

* **Validación de documentos en el sistema**: Angel Huberto Pulido Burgos enfatiza la necesidad de que la aplicación valide los documentos obligatorios (factura y packing list) desde el momento en que se crea la referencia. Se busca que el sistema advierta automáticamente al usuario sobre la falta de documentos o sobre requisitos adicionales derivados de las características de la mercancía, asegurando así el cumplimiento normativo.

* **Registro de la referencia en SIGAC 2**: Perla Lopez describe cómo se registra una referencia en el sistema SIGAC 2\. Tras recibir la notificación, se capturan datos como el importador, la línea transportista y la información de arribo. Perla indica que, aunque se hace una captura preliminar, datos como el peso o la cantidad de bultos son confirmados posteriormente por el almacén.

* **Captura de factura y digitalización**: Al capturar la información, Perla Lopez explica que si inicialmente solo se dispone del packing list, este se registra para anticipar la referencia y se convierte a factura posteriormente cuando esta llega. Mientras que la factura y el packing list son estándar, otros documentos adicionales se anexan según aplique y la digitalización de los mismos depende de los requerimientos específicos de cada cliente.

* **Clasificación de números de parte**: En el proceso de alta de partidas, si un número de parte ya está clasificado, el sistema despliega automáticamente su descripción. Si no está clasificado, la tarea se deriva al clasificador. Perla Lopez destaca que, para una clasificación precisa, el equipo recopila información técnica detallada del cliente, como fichas técnicas, fotos o descripciones de uso.

* **Revisión física y expediente**: Perla Lopez detalla que, tras ingresar las facturas y partidas, el equipo espera el "expediente completo" del almacén, el cual contiene los resultados de la revisión física de la mercancía. Este expediente incluye observaciones sobre el estado de los bienes, como daños o discrepancias detectadas. La falta de este archivo bloquea acciones en el sistema, como la generación de una orden de carga.

* **Digitalización del proceso de verificación física**: German Castro y Angel Huberto Pulido Burgos discuten la posibilidad de integrar el reporte de verificación física directamente en la aplicación, sustituyendo los archivos PDF externos. El objetivo es que el sistema permita comparar los datos reales de la verificación contra la información de la factura, facilitando la detección automática de discrepancias y mejorando la coherencia del expediente digital.

* **Controles y bloqueos del sistema**: Perla Lopez confirma que la revisión física es obligatoria para las importaciones y que el sistema impone restricciones si esta no ha sido validada. Por ejemplo, el sistema impide continuar con procesos como la validación del pedimento o el envío de archivos al pre-validado (como Karem) mientras exista un bloqueo por falta de revisión física o discrepancias no resueltas.

* **Restricciones de validación y revisión física**: Perla Lopez identifica un error en el sistema que impide la validación de pedimentos debido a la falta de revisión física, lo cual bloquea el proceso hasta que se resuelva. Se aclara que el proceso de validación inicial se realiza a través de Karem antes de proceder con el pago del pedimento y otros pasos ante la asociación, donde pueden surgir errores que requieren justificación o liberación de archivos.

* **Proceso de gestión documental y captura**: Angel Huberto Pulido Burgos recapitula el flujo de trabajo que comienza con la instrucción del cliente, la creación de la referencia, y la captura de datos como bultos y guías. Perla Lopez confirma que la factura comercial y el packing list son obligatorios para la digitalización de documentos, pero no necesariamente para la captura inicial, permitiendo que almacén procese la solicitud de revisión mientras se avanza en el resto de la operación.

* **Solicitud de fondos y gestión financiera**: Perla Lopez detalla que, una vez generado el expediente, se programa la mercancía según las necesidades del cliente y se trabaja en la solicitud de fondos para clientes de contado. Se especifican montos de 44,000 pesos para la cuenta mexicana y 425 dólares para la cuenta americana, cubriendo servicios como embarque consolidado, almacén, chiper y transfer, además de impuestos por 25,257 pesos e impuestos de honorarios.

* **Cálculo de impuestos y aplicación de tipos de cambio**: Se explica que los impuestos, que incluyen el IVA, la prevalidación y el IVA de la prevalidación, se calculan automáticamente mediante el sistema. Perla Lopez señala que, de no aplicar certificado de origen, el sistema calcula automáticamente el IGI y el DTA, utilizando el tipo de cambio vigente para el día siguiente según lo publicado, con el fin de realizar el pago. Enrique Lopez añade que el sistema realiza estos cálculos automáticamente, aunque el personal debe conocer los detalles para explicar al cliente el origen de los cobros.

* **Generación de la operación y requisitos de clasificación**: Perla Lopez indica que la operación puede generarse antes de finalizar la clasificación, pero no permite actualizar datos ni realizar cálculos correctos hasta que la clasificación esté completa. Se aclara que, aunque el sistema permita crear la operación, es indispensable contar con la clasificación para calcular los impuestos correctamente y proceder con la validación.

* **Gestión de referencias y pedimentos en el sistema**: Perla Lopez explica la función de la pestaña de referencias y pedimentos, donde se gestiona el inventario y se vinculan múltiples pedimentos a una sola operación, como en el caso de referencias consolidadas. Se menciona que el régimen se ingresa manualmente en tráfico y que este tab permite visualizar la relación entre las referencias y los pedimentos asociados.

* **Funcionalidad de las pestañas de transporte y seguimiento**: Se revisan diversas secciones del sistema, incluyendo datos del transporte, vehículos, eventos de seguimiento para la lluvia de información al cliente, y el módulo de gastos comprobados. Perla Lopez aclara que la pestaña de características vinculares es utilizada principalmente por bodega, y que existen alertas específicas que indican errores si faltan eventos de revisión física o digitalización.

* **Bitácora de estatus y notificaciones de almacén**: Perla Lopez muestra cómo se utiliza la sección de notificación de estatus para llevar una bitácora detallada de por qué la mercancía no ha salido del almacén. Esta herramienta permite registrar cuándo se notificó al cliente sobre los pendientes de instrucciones de despacho, asegurando un registro histórico del proceso.

* **Debate sobre bloqueos del sistema y autorizaciones**: Se discute la posibilidad de que el sistema bloquee la generación de documents, COVE y manifestaciones de valor si no se ha completado la revisión física. Perla Lopez argumenta que no debería ser un impedimento total, ya que existen casos donde el cliente solicita el despacho inmediato bajo su propia responsabilidad, por lo que se propone que el sistema permita continuar bajo la autorización de un gerente o del mismo cliente.

* **Flujo final de operación y cierre de análisis**: Se define el flujo de trabajo final: generación de documentos, COVE, glosa, manifestación de valor, transmisión, validación y pago del pedimento, seguido de la generación del DODA y, si aplica según la cantidad (2,500 dólares), el chiper. Angel Huberto Pulido Burgos concluye que se encargará de realizar un esquema gráfico de este proceso ("happy path") para presentarlo en la reunión diaria del día siguiente.

*Revisa las notas de Gemini para asegurarte de que sean precisas. [Obtén sugerencias y descubre cómo Gemini toma notas](https://support.google.com/meet/answer/14754931)*

*Cómo es la calidad de **estas notas específicas?** [Responde una breve encuesta](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=l66A5v9EW8of8TTCBWW2DxIVOBABMgUIigIgABgBCA&detailid=standard&screenshot=false) para darnos tu opinión; por ejemplo, cuán útiles te resultaron las notas.*