# Reunión 2026-07-31 — Revisión de Movimientos - Guía

## Asistentes
- Angel Huberto Pulido Burgos
- German Castro
- Carlos Alexis Galaviz Rosas

## Decisiones (con su porqué)

- **Se generará un "movimiento de entrada" por cada guía House automáticamente en backend**, transparente en la UI para Querétaro (front sigue mostrando "guías", no "movimientos de entrada"; a criterio si se deja o quita la nomenclatura).
  - Porqué: en Querétaro (aéreo) no existe hoy un movimiento de entrada como tal — la entrada se da directo a la guía y no hay forma de vincularle transporte u otra cosa; simplemente se muestran los datos del manifiesto a nivel guía. Un avión trae una guía Master que engloba N guías House, y cada guía House equivale a un previo — por eso el movimiento de entrada debe generarse por guía House, no por el manifiesto completo.
  - Alternativa considerada y descartada: Carlos propuso manejar todo a nivel manifiesto (recepción de manifiesto) agregando metadata a las guías, sin generar movimientos. Se descartó porque introducir el concepto de "movimiento de entrada" de forma visible en Querétaro rompería la costumbre operativa del cliente ("si les metemos esa idea... van a saltar"); se prefirió generarlo por detrás sin que lo vean.

- **En el módulo de Guías se agregará la posibilidad de subir/vincular facturas directamente a la guía House**, reutilizando el formulario de factura ya existente (documento + datos de factura), para que a almacén le aparezca esa factura y su documento ya vinculados al movimiento.
  - Porqué: en Querétaro no vinculan factura↔guía al recibir el manifiesto; el tramitador tiene que buscar manualmente la factura para poder hacer el previo (caso mencionado: problema recurrente de Adolfo), generando doble trabajo entre el ejecutivo y el tramitador.
  - Alternativa considerada (parcialmente compatible, no implementada aún): Carlos propuso cargar todas las facturas del manifiesto de golpe (ej. 25) ancladas al manifiesto, y después darles la posibilidad de asignarlas una por una a cada guía (ej. 10), mostrando cuántas faltan por asignar — así cualquiera de los ejecutivos del cliente puede ver el avance sin duplicar trabajo ni tener que coordinarse por fuera. Angel estuvo de acuerdo con la idea de fondo pero decidió resolverlo directamente en el formulario de guía en vez de un paso de asignación aparte.
  - Responsable de vincular factura↔guía: el ejecutivo, porque es quien hace el "anticipo" de la referencia y da de alta las facturas primero.
  - Si la factura se sube desde el módulo de referencia/DGO (sin saber a qué guía pertenece), el sistema debe preguntar a qué guía House pertenece para vincularla.
  - Las facturas subidas desde guía deben integrarse al mismo objeto de "expediente" asociado al movimiento y a la referencia (confirmado por German: debe ser el mismo objeto de expediente).
  - A futuro (no ahora, falta Zeus): buscar criterios de identificación (ej. guía) para automatizar el matching documento↔información manual, similar al concepto de "CDP homologado" ya usado en otro punto del proyecto.

- **En el formulario de factura (DGO) se debe poder subir un documento genérico** (no solo imágenes) o **ligar un documento ya existente del expediente** a una factura manual.
  - Porqué: hoy para cargar un documento hay que entrar por una ruta de "25 clics"; se busca reducir esa fricción y evitar duplicar cargas cuando el documento ya está en el expediente.

- **La subdivisión sigue siendo sobre el movimiento, no sobre el DGO** (se corrige una idea de reuniones anteriores donde se había mencionado dividir "sobre un DGO").
  - Porqué / aclaración de German: la subdivisión tiene dos partes — la afectación física (a nivel movimiento, tracking de la separación real de mercancía) y la división documental (a nivel DGO/pedimento, reparto de partidas de la factura entre DGOs). El DGO es por pedimento, no por movimiento: un mismo DGO puede agrupar varios movimientos de entrada/guías si solo se separan físicamente pero no documentalmente (caso Querétaro: dos guías House que se separan físicamente pero viajan en un solo DGO).

- **Citas marítimas/aéreas: solución rápida acordada** — agregar los campos de citas que hoy registran en Excel (cita de entrada a puerto, cita de previo, cita de recolección de vacío, cita de retorno de vacío) + historial de citas, en vez de esperar la integración completa.
  - Porqué: Jesús y Elian preguntaron cómo se van a mostrar las citas marítimas/aéreas y hoy no existen en el sistema. El "deber ser" (a futuro) es que el usuario pueda consultar desde la plataforma el sitio donde se sacan las citas y que un agente (browser automation) las busque y registre solas — pero por ahora se resuelve con captura manual de los campos + historial.
  - Importante: cuando hay un cambio/reprogramación de cita se debe registrar la cita anterior y el motivo del cambio. Las reprogramaciones son un KPI a favor de Carmi (evidencian que un retraso no es responsabilidad de Carmi sino de la naviera o falta de citas) — salvo cuando el cambio de cita lo provoca un error propio, ahí les pega a ellos.
  - Carlos se encarga de platicarlo con Jesús/Elian y traer el detalle.

- **Se agregará un "avatar" de responsable en la cabecera de la referencia**, editable entre ejecutivos/miembros de un equipo ("team"), con historial de cambios.
  - Porqué (German): reduce los correos de "ya te avisé y no me contestas"; permite medir tiempos muertos y costear cuánto cuesta cada referencia en tiempos sin movimiento. German ya lo implementó en otro proyecto y confirma que ayudó a eliminar ese tipo de correos.
  - Se descartó (por ahora) automatizar el reparto de responsable de clasificación por cola/equidad de carga como se había propuesto en una sesión anterior — Carlos señaló que un clasificador es un cuello de botella (10-15 expedientes) y no sabe cuál urge, por lo que no se le puede estar liberando/reasignando trabajo libremente; ese reparto se retomará junto con el tema de "cola de trabajo" más adelante.
  - Dependencia: el módulo de perfil de usuario / equipos lo está terminando Elian (generando "equipos"); se necesita que él termine esa parte antes de poder implementar el avatar de responsable.

- **Se retoma la acción de marcar una referencia como "urgente"** para que aparezca en la bandeja de entrada (pedido original de Enrique).
  - Porqué: para una primera versión, cualquiera puede marcarla urgente; después se ajustarán criterios/restricciones si el gerente los pide (German anticipa que van a pedir cambios, pero se libera así para la v1).

## Lógica de negocio

- Estructura de una guía House capturada del manifiesto (campos base): guía Master, guía House, régimen, clave de pedimento (no viene en manifiesto, la agrega el cliente), comentarios, fecha, origen, destinatario/cliente, descripción de mercancía, peso, unidad, piezas, bulto, valor.
- Relación guía Master ↔ guía House: una guía Master puede englobar N guías House (ejemplo usado: hasta 20); cada guía House es la unidad sobre la que se hace un previo.
- Una misma guía House puede llegar en distintos vuelos y con varias facturas (llegadas parciales).
- El "movimiento" para Carlos es más un contenedor para identificar información del cliente y tomar decisiones documentales — la separación entre lo documental (referencia/almacén) y lo operativo (warehouse management) es intencional y ya existente en el diseño actual.

## Tareas

1. **Guías (front):** agregar en el módulo de Guías el formulario de facturas para poder subir factura (documento + datos) vinculada directamente a la guía House.
2. **Guías (backend):** al crear una guía House, generar automáticamente un movimiento de entrada asociado (oculto en UI para Querétaro), visible para almacén (para sus previos) y visible desde referencia (para ver el previo por guía House).
3. **Expediente:** al subir una factura desde Guías o desde DGO, debe integrarse al mismo objeto de expediente asociado al movimiento/referencia.
4. **DGO/Referencia:** cuando se sube una factura desde ahí sin guía asociada, preguntar a qué guía House pertenece para vincularla.
5. **Factura (DGO):** permitir subir un documento genérico adicional vinculado a la factura (no solo imágenes) y/o ligar un documento ya existente del expediente a una factura manual.
6. **Citas:** diseñar/implementar formulario de citas marítimas y aéreas con los campos de cita de entrada a puerto, cita de previo, cita de recolección de vacío, cita de retorno de vacío + historial de citas + registro de motivo de reprogramación cuando cambie una cita.
7. **Referencia:** agregar "avatar" de responsable editable en la cabecera de la referencia, con historial de cambios (depende de que Elian termine el módulo de equipos/perfil de usuario).
8. **Bandeja de entrada / Referencia:** agregar acción para marcar una referencia como "urgente" y que aparezca en la bandeja de entrada.

## Tickets

Ninguno mencionado explícitamente en esta reunión.

## Temas descartados por irrelevantes

- Comentarios sobre "borrar todos los proyectos y empezar desde cero" al inicio (broma).
- Charla sobre herramientas de IA (Deep Agents, comparación de verbosidad entre modelos, "artifactor", componentes atomizados de interfaz de chat) — exploración personal de Carlos, no una decisión de proyecto.
- Comentarios sobre OCR/escaneo de PDFs de pedimentos y firma digital como posible solución — exploración de Carlos sobre una prueba con un tercero ("Roberto"), sin decisión ni tarea concreta derivada.
- Problemas técnicos de audio/conexión durante la llamada.
- Bromas y comentarios personales (Deep Agents, "niño obediente", "vamos a gamificarlo", planes de Carlos para la próxima semana con el "glosador").
