# Reunión 2026-07-20 — Recap visita Querétaro

## Asistentes
- Carlos Alexis Galaviz Rosas (Charlie/Carlos)
- German Castro (Germán)
- Angel Huberto Pulido Burgos (Ángel)

## Decisiones (con su porqué)

### Alcance SIG 3 vs. aplicación móvil
- **SIG 3** se queda con el módulo de **manifiestos** (ingesta del Excel/correo
  que envía la paquetería/logística) y con el tema de **revalidación**.
- La **aplicación móvil** se queda con todo el tema de **previo** y la
  **gestión de salida del embargo/mercancía**.
- Porqué: son los dos frentes de trabajo que corren en paralelo en Querétaro
  (trámite en SIG 3, campo/terminal en la app), y separarlos evita mezclar
  specs de una interfaz táctil/móvil con las de escritorio.

### Flujo de ingreso de manifiesto → expediente (SIG 3)
El manifiesto de carga que envía la paquetería (Excel) debe ingresarse a un
módulo de manifiestos: capturar la información, buscar el expediente del
cliente, y verificar si tiene facturas completas; si no las tiene, se carga
como INA1.

### Trabajo en paralelo desde la app móvil
El empleado inicia en la app, descarga las guías del vuelo y separa aparte
las que van "en rojo" en pantalla (pedimento). En simultáneo corre la
gestión del revalidador para hacer la revalidación de esa mercancía en las
oficinas de terminal. Se necesita una **glosa** que despliegue todo lo
relacionado a cada partida: pesos reales, medidas, país, fotografías
adjuntas, tema arancelario e impuestos.

### Flujo de salida de mercancía (app móvil)
Cuando el pedimento queda libre, terminal no se entera automáticamente: el
tramitador va físicamente a la ventanilla de Terminal Logistics a decir que
está pagado, liquida maniobras y almacenaje, y le entregan un ticket de
salida en papel. La app debe permitir que el revalidador abra el módulo,
escriba manualmente el folio del ticket **o le tome una foto para
extraerlo**, genere el pase de salida y notifique al chófer por WhatsApp
para que se acerque a la puerta.
- **Pendiente/abierto**: aún no está definido en qué momento se sabe a qué
  puerta hay que ir a recolectar, para poder notificarle al chófer la puerta
  correcta (Carlos lo marcó explícitamente como algo que falta).

### Los tres tipos de "previo" que debe soportar la app móvil
German insistió en que es importante porque de ahí se dispara todo el
expediente/glosa, y en que deben cubrirse los tres casos tanto en aéreo como
en marítimo y terrestre (aunque cada modo tenga campos distintos — p. ej. en
marítimo no está claro si el dato ancla es la guía o el número de
contenedor, queda pendiente de confirmar):

1. **Previo 1**: el ejecutivo puede subir las facturas antes de que se
   realice el previo.
2. **Previo 2** (la revisión física que hace trámite en el aeropuerto/app
   móvil): el tramitador abre el paquete y ahí adentro viene la factura
   comercial — le toman foto, se extraen los datos automáticamente y el
   tramitador solo verifica/corrige lo prellenado.
3. **Previo 3**: dentro del paquete **no** viene la factura comercial, y
   usan el formato propio de ellos ("la hojita") en su lugar.

Cuando sí llega la factura, trámite hace un proceso de verificación y
apunta manualmente: número de guía, marca, modelo, serie, país de origen,
peso, bultos y número de parte (no la fracción arancelaria, porque no la
tienen en ese punto) — es exactamente lo que piden los ejecutivos de
terrestre. Regla de negocio clave: **el dato que llegue primero es el que
alimenta el expediente/referencia** — si la foto del previo se procesa
antes de que la factura llegue por el canal normal, esos son los primeros
datos con los que se guarda la referencia.

### Manifiesto de carga → alta de guías → generación de referencia
- El manifiesto de carga llega por correo; hay que capturarlo automáticamente
  y generar una sección de "manifiestos de carga". Carlos y Ángel confirmaron
  que ya tienen los elementos para automatizar esta captura del correo.
- Esa sección **es en realidad el alta de guías**: cada guía del manifiesto
  se da de alta con un campo (o ID) de manifiesto de carga.
- Verificar si el correo trae un ID de manifiesto: Carlos y Ángel creen que sí
  (vendría en el nombre del correo o en la guía HL), pero no está confirmado
  al 100%. **Si no trae ID**, se arma uno combinando vuelo + día (cuidando que
  el número de vuelo se puede repetir, así que solo vuelo no basta).
- Se construirá un **tablero de control de guías** (nombre elegido por
  German — no "manifiesto de carga", porque el cliente ya insistió en que
  para ellos son "guías", no manifiestos). Desde ahí se seleccionan varias
  guías (ej. 4) y con esa selección se **inicia la creación de una
  referencia**, asignando cliente, destinatario y **régimen** — el régimen es
  el criterio con el que ellos ya separan guías en su Excel (equivalente a la
  "subdivisión virtual" que se usa en warehouse, según comentó Carlos).
- Querétaro (aéreo) **no tiene movimiento de entrada** — eso es exclusivo de
  terrestre. Su punto de partida son directamente las guías del día.
- Decisión de arquitectura importante (defendida por German ante una
  objeción anticipada del cliente): aunque el cliente diga "solo damos
  seguimiento a referencias, no a guías", en la práctica llevan un Excel
  paralelo de seguimiento por guía. Por eso **la guía debe tener su propio
  ciclo de vida y seguimiento**, no ser solo un campo dentro de la
  referencia — pero el alta de guía y la generación de referencia deben
  sentirse casi automáticas/encadenadas para el usuario. Regla dura: **no
  se puede generar una operación a partir de una guía suelta**, siempre debe
  partir de una referencia.

### Campos reales del manifiesto de carga (ejemplo: DH Express México)
Fecha, guía master, guía house, origen, descripción de la mercancía, peso,
piezas, bultos, valor, destinatario, domicilio destinatario, remitente,
domicilio remitente, número de vuelo.
- Reto identificado: destinatario y domicilio deben asociarse contra
  catálogos propios (ID de cliente, catálogo de domicilios). Un mismo
  destinatario (ej. Tyco Electronics) puede tener varias razones
  sociales/RFC por sucursal (México, Guadalajara, Monterrey). Solución
  propuesta: dar al LLM la capacidad de hacer matching tipo regex/fuzzy
  sobre el catálogo (salen ~3 candidatos) y que el LLM seleccione el
  correcto. German comentó que esto debería ser más sencillo que cuando
  extraían datos en el sistema anterior ("SEUS"/"CUS 2.0").

### Tablero de control de guías — estatus y comentarios
Se mostró en vivo una referencia visual (herramienta "Tasquility"): columna
de datos de guía, comentarios libres, y un campo de **estatus tipo kanban**
(colores azul/rojo/amarillo) que el usuario puede mover manualmente para
darle seguimiento al ciclo de vida completo de la guía.
- Los estatus concretos **no están definidos todavía**: German ha
  preguntado en ~10 juntas anteriores al cliente sin obtener una respuesta
  clara. Decisión: proponer 4-5 estatus iniciales derivados del Excel actual
  del cliente, e iterar según el feedback real de Querétaro.
- Al cambiar un estatus manualmente se debe pedir **como mínimo fecha y
  comentario** (la fecha no siempre coincide con el timestamp del cambio —
  ej. la mercancía pudo arribar una hora antes de que alguien actualice el
  sistema).
- Algunos estatus sí se podrán actualizar automáticamente desde la app móvil
  (inicio/fin de previo); otros van a requerir información adicional (etal
  carry, container, etc.).
- Envío de correo bajo demanda, no automático por evento: ya existe este
  patrón en otro cliente/proyecto porque antes se mandaban ~100 correos por
  embarque; ahora son notificaciones dentro de la app y el correo solo se
  dispara si el usuario lo pide explícitamente.

### Demo de referencia (plataforma "Tasquility", no exclusivo de este cliente)
German mostró una plataforma propia como inspiración de UX para varias de
las ideas anteriores: IA conversacional dentro del embarque para crear
rutas/costos, tarifario de costos ya integrado, y un módulo de
**manifestación de valor** que junta documentos automáticamente, detecta
documentos faltantes, permite configurar qué debe subir cada tipo de
proveedor (logístico, de mercancía, de incrementables), extrae documentos
recibidos por correo, los valida y genera un reporte de aprobación/rechazo
por documento. Además organiza los correos en carpetas por embarque y los
sincroniza a SharePoint/Google Drive si el cliente lo conecta. La interfaz
se mantiene deliberadamente limpia ("trato de no meterle muchas cosas").
Los clientes reaccionan muy bien porque no suelen ver su operación
organizada y validada de esta forma.

### Reparto de trabajo confirmado
- **Carlos**: app móvil (los tres previos + módulo de salida) y cerrar el
  tema de "Zeus" en paralelo. El módulo de salida para terrestre es grande
  (checklists extensos); para Querétaro (aéreo) es mucho más simple: subir
  el ticket de salida (dos formatos que el cliente ya mostró), generar la
  hoja de salida, y notificar. El chófer y el equipo interno tendrían el
  ticket digitalizado (aunque se siga llevando impreso); se puede validar
  contra el sistema que la mercancía realmente salió, y mostrar cuánto
  tiempo pasó entre "ticket generado" y "salida confirmada" (dato que el
  cliente quiere ver, porque a veces el ticket ya está pero tardan 20-40 min
  en llevarlo al chófer).
- **Ángel**: generación de la tabla/tablero de control de guías; dentro de
  la referencia, agregar en el **DGO** la proforma de pedimiento y sus tabs;
  customizar el menú para operaciones aéreas quitando pantallas que no
  aplican (ej. citas — falta precisar cuál era la otra pantalla mencionada,
  no se recordó en la sesión); y en la pantalla de previo, poder comparar el
  previo de las guías de esa referencia contra lo ya capturado en el
  sistema.
- **Pendiente**: mañana (2026-07-21) hay revisión de marítimo con el
  cliente — hay que definir junto con ellos cuál es el equivalente al
  manifiesto de carga para marítimo (no van a ser "guías").

## Lógica de negocio

### Ciclo de vida de la guía vs. la referencia
La guía es la unidad con la que el cliente habla con sus proveedores y
transportistas ("número de guía"); la referencia es la unidad interna de
Carmi/SIG. Una referencia se genera a partir de una o varias guías. La guía
necesita su propio tracking porque puede pasar tiempo entre que se da de
alta y que se agrupa en una referencia, y porque el cliente la sigue
categorizando por régimen antes de agruparla.

### Datos que apunta trámite al recibir la factura comercial
Número de guía, marca, modelo, serie, país de origen, peso, bultos, número
de parte (sin fracción arancelaria, porque en ese punto del proceso no la
tienen). Esta es la misma información que después piden los ejecutivos de
terrestre.

## Tareas
> Detectadas, no creadas (este cliente gestiona tareas fuera de `/reunion`).

- (Carlos) **App móvil — módulo de previo**: implementar los 3 tipos de
  previo (subida de factura anticipada, foto de factura comercial con
  extracción automática, formato propio "la hojita"), cubriendo aéreo,
  marítimo y terrestre con sus campos específicos por modo.
- (Carlos) **App móvil — módulo de salida**: folio de ticket manual o por
  foto con extracción, generación de pase de salida, notificación al chófer
  por WhatsApp, y validación de tiempo entre ticket generado y salida
  confirmada.
- (Carlos) **Definir notificación de puerta de recolección**: falta resolver
  en qué momento se conoce la puerta correcta para notificar al chófer.
- (Carlos) **Cerrar el tema de "Zeus"** (mencionado como pendiente en
  paralelo a la app móvil).
- (Ángel) **Captura automática del correo de manifiesto de carga** y alta
  de guías a partir de él (confirmar si el correo trae ID de manifiesto; si
  no, componerlo con vuelo + día).
- (Ángel) **Tablero de control de guías**: tabla con datos de guía,
  comentarios y estatus tipo kanban movible; selección múltiple de guías
  para iniciar la creación de una referencia (cliente, destinatario,
  régimen).
- (Ángel) **Definir estatus del tablero de guías**: proponer 4-5 estatus
  iniciales a partir del Excel actual del cliente e iterar con Querétaro.
- (Ángel) **DGO de referencia**: agregar proforma de pedimiento y sus tabs.
- (Ángel) **Customizar menú aéreo**: quitar pantallas no aplicables (ej.
  citas; falta precisar la segunda pantalla mencionada).
- (Ángel) **Pantalla de previo**: comparar el previo de las guías de la
  referencia contra lo ya capturado en el sistema.
- (Equipo) **Reunión de marítimo (2026-07-21)**: definir con el cliente el
  equivalente al manifiesto de carga para marítimo.
- (Equipo) **Matching de destinatario/domicilio contra catálogo**: resolver
  con LLM + búsqueda tipo regex/fuzzy sobre el catálogo de clientes/RFC por
  sucursal.

## Temas descartados por irrelevantes
- Problemas técnicos del meet (pantalla congelada, llamada entrante de
  German, reconexión de Ángel).
- Muletillas, confirmaciones genéricas y interrupciones cruzadas
  ("okay", "ajá", "sí", "mm").
- Comentario jocoso fuera de tema al cierre de la grabación ("Vai rico
  eso...").
