jul 8, 2026

## **Aéreo SIGAC 3.0**

Invitado [Ana Rocio Angeles Marquez](mailto:aangeles@carmi.com) [Andres Munguia Lopez](mailto:amunguia@carmi.com) [Angel Huberto Pulido Burgos](mailto:apulido@carmi.com) [Ana Laura Rivera Alonso](mailto:arivera@carmi.com) [Carlos Alexis Galaviz Rosas](mailto:cgalaviz@carmi.com) [Carlos Leobardo Mera Ponce](mailto:cmera@carmi.com) [Daniel Aguila](mailto:daguila@carmi.com) [Diana Hernandez Martinez](mailto:dhernandez@carmi.com) [Dizahab Vargas Montiel](mailto:diza.vargasm@carmi.com) [German Castro](mailto:german.castro@carmi.com) [Gabriela Lugo Castro](mailto:glugo@carmi.com) [Gilberto Luna Rivera](mailto:gluna@carmi.com) [Geovanny Manuel Martinez](mailto:gmartinez@carmi.com) [Gigliola Quintanar Sandoval](mailto:gquintanar@carmi.com) [Jennifer Fuentes Sanchez](mailto:jfuentes@carmi.com) [Juan Ivan Gonzalez Fuerte](mailto:jugonzalez@carmi.com) [Karla Paola Hernandez Angeles](mailto:karlahernandez@carmi.com) [Lucero Muñoz Chilino](mailto:lmunoz@carmi.com) [Marco Antonio Castillo Gonzalez](mailto:mcastillo@carmi.com) [Danae Gómez Rios](mailto:migomez@carmi.com) [Maria Maythe Resendiz](mailto:mresendiz@carmi.com) [Miriam Sanchez Ojeda](mailto:msanchez@carmi.com) [Maria Elena Vera Olivares](mailto:mvera@carmi.com) [Santa Cecilia Mora Guerrero](mailto:scecilia@carmi.com) [Tramite y Despacho Queretaro](mailto:tramiteqro@carmi.com) [Valeria Hernandez Tovar](mailto:v.hernandez@carmi.com) [VANESSA CHANTAL MONTESINOS SALDIVAR](mailto:vmontesinos@carmi.com) [Xochitl Reyes Vazquez](mailto:xreyes@carmi.com) [Yolanda Gomez Espinosa](mailto:yolanda.gomez@carmi.com) ~~[Salvador Cervantes Franco](mailto:scervantes@carmi.com)~~

Archivos adjuntos [Aéreo SIGAC 3.0](https://calendar.google.com/calendar/event?eid=MzRkNWNndTBqbzE5NDBoa2xla3RraXIzMGJfMjAyNjA3MDhUMTczMDAwWiBnZXJtYW4uY2FzdHJvQGNhcm1pLmNvbQ)

Registros de la reunión [Transcripción](https://docs.google.com/document/d/1R_IPUMyukUF3271qFWr_K0_DmQO-mkefd3XWQRkuf7s/edit?usp=drive_web&tab=t.c1zduwnbyso4) 

### **Resumen**

La reunión analizó los flujos operativos, identificando necesidades de automatización para optimizar la gestión aduanera.

**Análisis del flujo operativo**  
El equipo revisó los procesos de prealerta, apertura de referencias y manejo de consolidados para Taiko. Se aclaró la distinción entre guías Master y House en los registros.

**Desafíos de control manual**  
La dependencia actual de hojas de cálculo externas para gestionar regímenes aduaneros y datos de tráfico genera ineficiencias. Es obligatorio registrar datos precisos de facturas para evitar rechazos.

**Mejoras y automatización propuestas**  
Se decidió integrar el control operativo al sistema SIGAC 3 para automatizar la asignación de regímenes y notificaciones. Esto reemplazará los procesos manuales actuales y mejorará la comunicación.

### **Próximos pasos**

- [ ] \[Juan Ivan Gonzalez Fuerte\] Compartir diagrama: Compartir el diagrama de flujo operativo que describe el proceso de prealerta y consolidación mencionado durante la reunión.

- [ ] \[El grupo\] Mejorar Sigac 3: Desarrollar en el perfil de número de parte la capacidad de detectar automáticamente el régimen correspondiente. Automatizar esta validación para evitar consultas manuales por cada partida.

- [ ] \[Carlos\] Crear modulo previo: Desarrollar un modulo en la aplicacion movil que permita gestionar dos tipos de previo, uno basado en datos cargados por ejecutivos y otro para captura manual desde cero.

- [ ] \[Juan Ivan Gonzalez Fuerte\] Compartir diagrama: Enviar el diagrama del proceso analizado a Angel Huberto Pulido Burgos.

- [ ] \[Juan Ivan Gonzalez Fuerte\] Compartir archivos Excel: Facilitar acceso a los dos archivos Excel que contienen los procesos actualmente utilizados por el equipo.

- [ ] \[Juan Ivan Gonzalez Fuerte\] Compartir ejemplos guias: Enviar cinco ejemplos de guias House y ejemplos de correos asociados para que Angel Huberto Pulido Burgos pueda analizar el flujo de trabajo.

### **Detalles**

* **Flujo de trabajo de prealerta**: Juan Ivan Gonzalez Fuerte detalló que la operación aérea comienza con la recepción de una prealerta o notificación por parte del cliente, la aerolínea o el almacén. En Querétaro, el almacén de Terminal Logistic es el punto de referencia y el personal monitorea las notificaciones enviadas por aerolíneas como DHL y FedEx ([00:01:33](?tab=t.c1zduwnbyso4#heading=h.83ghertdi5k3)).

* **Revisión del diagrama de procesos**: Durante la sesión, Angel Huberto Pulido Burgos solicitó revisar el diagrama operativo, el cual incluye las etapas de prealerta, apertura de referencia, procesamiento de facturas, clasificación, creación de la operación, asignación de pedimento, validación, pago, despacho y salida. El personal aclaró en qué puntos específicos intervienen el almacén y el sistema SIGAC ([00:04:47](?tab=t.c1zduwnbyso4#heading=h.4172kyludab)).

* **Manejo de consolidados y regímenes**: Se discutió que los consolidados para el cliente Taiko no operan mediante un ciclo tradicional de apertura y cierre, sino como un pedimento acumulativo. El personal separa las guías según el régimen, ya sea importación definitiva o temporal. Angel Huberto Pulido Burgos sugirió implementar perfiles de "Número de Parte" en SIGAC 3 para automatizar la asignación del régimen y evitar procesos manuales ([00:05:56](?tab=t.c1zduwnbyso4#heading=h.7avn1srs1iyr)) ([00:09:17](?tab=t.c1zduwnbyso4#heading=h.rl0v2e65zlvk)).

* **Control mediante hojas de cálculo y automatización**: El equipo utiliza hojas de cálculo externas para gestionar las guías y separarlas por régimen. German Castro y Angel Huberto Pulido Burgos discutieron la posibilidad de extraer datos directamente de los correos de prealerta hacia el sistema para reducir la dependencia de archivos externos y agilizar el control ([00:12:50](?tab=t.c1zduwnbyso4#heading=h.g584y4xoahm7)).

* **Apertura de referencia y certificación de peso**: La referencia se apertura utilizando la guía Master. El personal no captura el peso final inicialmente, ya que espera la "certificación de peso" proveniente de la báscula del almacén tras el reconocimiento físico, dato que posee validez ante la autoridad ([00:15:10](?tab=t.c1zduwnbyso4#heading=h.ect1w9xq3iac)).

* **Gestión de guías Master y House**: Se aclaró la distinción entre las guías Master, vinculadas a cada vuelo, y las guías House, que corresponden a las unidades individuales. Esta distinción es fundamental para el control operativo y la organización de los consolidados ([00:18:56](?tab=t.c1zduwnbyso4#heading=h.mb4nc19wh8l3)).

* **Criterios para la separación de referencias**: Juan Ivan Gonzalez Fuerte explicó que la decisión de separar las guías en distintas referencias se basa principalmente en el régimen aduanero. Otros factores incluyen la recepción de mercancía parcial, donde el personal debe esperar el complemento, y consideraciones de volumen ([00:21:53](?tab=t.c1zduwnbyso4#heading=h.km7wyre63t85)).

* **Requerimientos de datos en facturas**: Para cumplir con las exigencias del cliente Taiko, es obligatorio ingresar la guía House y el número de entidad del exportador en cada partida dentro del sistema SIGAC 2\. El personal advirtió que omitir esta información provoca el rechazo de los archivos por parte del cliente ([00:26:30](?tab=t.c1zduwnbyso4#heading=h.g1gh4afn51ne)) ([00:41:04](?tab=t.c1zduwnbyso4#heading=h.dx0x1aqlm2lu)).

* **Captura de datos de tráfico y operación**: Tras capturar los datos de facturas, el personal ingresa la información de tráfico y crea la operación con los datos del cliente. Debido a que el sistema no suma automáticamente los totales de flete en consolidados, el personal realiza estos cálculos manualmente en hojas de apoyo ([00:28:44](?tab=t.c1zduwnbyso4#heading=h.b22yiqksy6pw)).

* **Reconocimiento previo y comunicación**: El personal está a la espera del reconocimiento previo realizado por el almacén para validar la mercancía contra la factura. Se discutió la necesidad de mejorar la comunicación entre el equipo de trámite en el almacén y los ejecutivos, posiblemente mediante notificaciones de estado en la aplicación para agilizar la revisión ([00:42:33](?tab=t.c1zduwnbyso4#heading=h.9v5wr7semucy)).

* **Conclusiones y mejoras propuestas**: Angel Huberto Pulido Burgos concluyó que las mejoras críticas incluyen un módulo específico de reconocimiento previo en SIGAC 3, la integración de los controles manuales de Excel al sistema y la automatización de notificaciones de estado entre el almacén y el equipo ejecutivo ([00:47:05](?tab=t.c1zduwnbyso4#heading=h.mihc2hklevp8)).

* **Acuerdos de documentación**: German Castro solicitó que el personal comparta ejemplos reales de guías Master, guías House y los archivos de Excel utilizados para el control operativo con Angel Huberto Pulido Burgos, con el fin de unificar los conceptos operativos de todo el equipo ([00:51:48](?tab=t.c1zduwnbyso4#heading=h.v24ypg8c02ex)).

*Revisa las notas de Gemini para asegurarte de que sean precisas. [Obtén sugerencias y descubre cómo Gemini toma notas](https://support.google.com/meet/answer/14754931)*

*Cómo es la calidad de **estas notas específicas?** [Responde una breve encuesta](https://google.qualtrics.com/jfe/form/SV_9vK3UZEaIQKKE7A?confid=Z06-B9EF-UQhcekRR08tDxIQOAIIigIgABgBCA&detailid=standard&screenshot=false) para darnos tu opinión; por ejemplo, cuán útiles te resultaron las notas.*