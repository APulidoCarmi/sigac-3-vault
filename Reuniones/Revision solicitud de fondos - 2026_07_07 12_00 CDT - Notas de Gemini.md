jul 7, 2026

## **Revision solicitud de fondos**

Invitado [Miguel Gomez](mailto:mgomez@carmi.com) [Angel Huberto Pulido Burgos](mailto:apulido@carmi.com)

Archivos adjuntos [Revision solicitud de fondos](https://calendar.google.com/calendar/event?eid=NnA5ZThpNGk3aWNnY3FqZ2VoNGtxNnQ4NjAgbWdvbWV6QGNhcm1pLmNvbQ)

Registros de la reunión [Transcripción](https://docs.google.com/document/d/1UyDCq7IMnKzS7-UszXH_d82QnJV8qQsXQtSIxx2psTc/edit?usp=drive_web&tab=t.2233h5adl4mh) [Grabación](https://drive.google.com/file/d/1-9_WmJ1iy4urNcjSVeyuNzHSFkLgrAiO/view?usp=drive_web) 

### **Resumen**

La sesión revisó el flujo de fondos en Sigac 3 y definió la integración centralizada del sistema.

**Categorización de fondos operativos**  
Los fondos se clasificaron en 3 categorías para integrar las solicitudes de operación. La tesorería mantuvo su rol de control sobre los pagos bancarios.

**Automatización y ajustes financieros**  
El sistema automatizó el cálculo de conceptos para eliminar entradas manuales. Los ejecutivos gestionaron manualmente los servicios duplicados en operaciones con múltiples pedimentos.

**Centralización del sistema**  
Se decidió centralizar la solicitud de fondos dentro del módulo operativo para optimizar el flujo. La propuesta incluyó notificaciones directas al cliente para agilizar la recepción.

### **Próximos pasos**

- [ ] \[El grupo\] Desarrollar lógica cobros: Desarrollar una funcionalidad en el sistema para prevenir el cobro duplicado de servicios cuando una operación contiene múltiples pedimentos. Esta mejora automatizará la gestión de conceptos al cruzar referencias entre procesos.

- [ ] \[El grupo\] Implementar flujo fondos: Implementar la estructura de tres conceptos para las solicitudes de fondos que incluye gastos comprobados, impuestos y honorarios en el nuevo sistema Sigac 3\. Este proceso replicará y optimizará la funcionalidad existente en Sigac 2\.

- [ ] \[Angel Huberto Pulido Burgos\] Validar modulo solicitud: Validar la ubicación técnica del módulo de solicitud de fondos dentro del sistema de operaciones o referencias.

- [ ] \[El grupo\] Integrar elementos solicitud: Determinar la viabilidad de integrar los elementos de impuestos gastos y honorarios en el proceso de solicitud de fondos.

### **Detalles**

* **Gestión de fondos en Sigac 3**: Miguel Gomez y Angel Huberto Pulido Burgos analizaron el manejo de las solicitudes de fondos para la operación de clientes, identificando tres categorías principales: gastos comprobados, impuestos de pedimento y honorarios. Miguel Gomez explicó que estas tres vertientes deben integrarse para crear una solicitud de fondos completa y coherente dentro del sistema ([00:00:01](?tab=t.2233h5adl4mh#heading=h.m354tus6rc6c)) ([00:24:20](?tab=t.2233h5adl4mh#heading=h.m19kuh3hcok7)).

* **Proceso de gastos comprobados**: Miguel Gomez describió que el ejecutivo es responsable de identificar si una operación requiere gastos comprobados, como lavado de contenedores o maniobras. El ejecutivo registra esta solicitud en una pantalla específica en el sistema, la cual es compartida con tesorería, permitiendo un flujo de comunicación para la transferencia de fondos al proveedor ([00:02:14](?tab=t.2233h5adl4mh#heading=h.eq2a73dr9pgn)) ([00:07:31](?tab=t.2233h5adl4mh#heading=h.n86cf0tgrzlt)).

* **Rol de tesorería y validación**: Se aclaró que la pantalla de solicitud es informativa para efectos de control interno. Tesorería es quien autoriza y ejecuta las transferencias bancarias en línea una vez que revisan la solicitud creada por el ejecutivo. Angel Huberto Pulido Burgos y Miguel Gomez coincidieron en que el sistema funciona como un medio de control, mientras que la comunicación sobre la ejecución de los pagos entre departamentos suele ocurrir por canales externos como correo o chat ([00:06:30](?tab=t.2233h5adl4mh#heading=h.xzqd54ry29rr)) ([00:08:39](?tab=t.2233h5adl4mh#heading=h.1pphc3ei1p2m)).

* **Impuestos de pedimento**: Se discutió cómo se determinan los montos para los impuestos, los cuales se basan en la sección de "efectivo" dentro del cuadro de liquidación del pedimento. Miguel Gomez enfatizó que este es el segundo punto fundamental para la solicitud de fondos y que la información se extrae directamente de dicha sección del pedimento ([00:13:11](?tab=t.2233h5adl4mh#heading=h.vprq3dinwvjo)).

* **Cálculo de honorarios y servicios**: Miguel Gomez explicó que el tercer componente, los honorarios, se genera automáticamente basándose en la configuración de tarifas en Sigac 2 y próximamente en Sigac 3\. Estas tarifas determinan los montos a cobrar por servicio según el peso, porcentaje o criterio configurado, eliminando la necesidad de entrada manual de datos para cada prefactura ([00:16:48](?tab=t.2233h5adl4mh#heading=h.4n1vuplbs1od)) ([00:22:42](?tab=t.2233h5adl4mh#heading=h.t5xohzlsqktt)).

* **Consolidación en la solicitud de fondos**: Miguel Gomez mostró cómo los tres elementos (gastos, impuestos y honorarios) convergen en la pantalla de solicitud de fondos. En Sigac 2, esta solicitud se vincula directamente al pedimento, permitiendo recalcular y totalizar automáticamente los montos requeridos para el cliente, asegurando que todos los cargos estén reflejados antes de enviar la solicitud ([00:24:20](?tab=t.2233h5adl4mh#heading=h.m19kuh3hcok7)) ([00:36:10](?tab=t.2233h5adl4mh#heading=h.61hdvnc5nwb2)).

* **Gestión de operaciones con múltiples pedimentos**: Angel Huberto Pulido Burgos y Miguel Gomez abordaron el desafío de operaciones que contienen varios pedimentos. Se determinó que el ejecutivo debe gestionar manualmente la eliminación de servicios duplicados en las solicitudes de fondos, ya que algunos servicios son por operación y no por pedimento, lo que requiere cuidado para evitar cobrar dos veces el mismo concepto ([00:27:52](?tab=t.2233h5adl4mh#heading=h.urv7qswzog8s)) ([00:32:48](?tab=t.2233h5adl4mh#heading=h.8vxicqc5lepf)).

* **Manejo de excepciones y pagos parciales**: Se discutió cómo manejar casos donde los montos cambian o se realizan pagos parciales. Miguel Gomez explicó que, ante estas situaciones, el ejecutivo puede realizar una solicitud complementaria o ajustar los montos, enfatizando que es responsabilidad del ejecutivo realizar estas correcciones para mantener la precisión financiera ([00:31:19](?tab=t.2233h5adl4mh#heading=h.i5kwb69c4s2h)).

* **Automatización de datos**: Miguel Gomez demostró cómo, al recalcular la solicitud, el sistema trae automáticamente los valores correspondientes a impuestos, gastos comprobados y servicios. Esto garantiza que la información sea consistente con lo registrado en las otras secciones del sistema, permitiendo que el ejecutivo genere el documento de solicitud con los datos precisos ([00:36:10](?tab=t.2233h5adl4mh#heading=h.61hdvnc5nwb2)).

* **Propuesta de notificación al cliente**: Angel Huberto Pulido Burgos propuso una mejora al proceso actual, que consiste en enviar una notificación directa al cliente vía una URL al generar la solicitud. Esta mejora permitiría que el cliente, al realizar el depósito, acceda al sistema para adjuntar su comprobante, notificando automáticamente tanto al ejecutivo como a tesorería sobre la recepción del dinero ([00:44:29](?tab=t.2233h5adl4mh#heading=h.hzm437qkn3z7)).

* **Validación de tesorería**: Se acordó que, tras la notificación del cliente, tesorería debe realizar una validación final en el banco para confirmar que el dinero ha sido recibido y que el monto es correcto. Este paso garantiza la integridad financiera antes de autorizar el pago del pedimento y avanzar con la operación ([00:43:41](?tab=t.2233h5adl4mh#heading=h.urr9yf48egmq)).

* **Refactorización del sistema y diseño futuro**: Angel Huberto Pulido Burgos y Miguel Gomez concluyeron que, para optimizar el flujo de trabajo en Sigac 3, la solicitud de fondos debe integrarse directamente en el módulo de operación o referencia, en lugar de depender de una pantalla separada. Esta modificación facilitará la gestión de las tres categorías de fondos (gastos, impuestos y honorarios) desde un punto centralizado, mejorando la usabilidad para el ejecutivo ([00:46:52](?tab=t.2233h5adl4mh#heading=h.rj287zqil2ko)).

*Revisa las notas de Gemini para asegurarte de que sean precisas. [Obtén sugerencias y descubre cómo Gemini toma notas](https://support.google.com/meet/answer/14754931)*

*Cómo es la calidad de **estas notas específicas?** [Responde una breve encuesta](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=VpCSmcVk4BV1nkhgYc_3DxIYOAIIigIgABgBCA&detailid=standard&screenshot=false) para darnos tu opinión; por ejemplo, cuán útiles te resultaron las notas.*