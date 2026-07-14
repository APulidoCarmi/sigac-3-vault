# Reunión 2026-07-13 — Revisión de pantallas

Continuación de la revisión de diseño/UX de SIGAC 3.0 entre **Ángel**
(desarrollo, presenta mockups de Figma) y **German Castro** (producto/
arquitectura, dirige la revisión). Se cubre la bandeja del ejecutivo, el
listado de referencias, el detalle de referencia (DGO, movimientos,
instrucciones/tickets) y se retoma el diseño del **expediente aduanero**
con un ejemplo dibujado por German. Continúa directamente
[[2026-07-07 - Revision de pantallas flujo operación]] (mismo par, misma
línea de diseño: expediente aduanero, DGO como fuente de verdad, CEUS).

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo — presenta los mockups)
- German Castro (producto / arquitectura — dirige la revisión)

## Decisiones (con su porqué)

### Bandeja de entrada / bandeja del ejecutivo
- **Se reestructuran las 4 cards de la bandeja del ejecutivo:**
  1. Cualquier referencia **esperando información del cliente o del
     transportista**.
  2. "Cartas sin firmar" se **renombra a "documentos que requieren
     atención"** (o listar los documentos específicos que faltan).
     Porqué: el nombre original generaba confusión (¿proformas? ¿hoja de
     recibido que firma el cliente?) y German prefiere que muestre
     faltantes concretos de documentos.
  3. **Arribos**: referencias con cita ya generada y por llegar (a la
     izquierda lo no arribado, a la derecha lo arribado).
  4. Abajo, aparte: **guías sin identificar** — embarques que ya
     llegaron a almacén pero no se sabe a qué cliente pertenecen (se
     diferencia de "por identificar", que si acaso sería para
     movimientos que ya tienen cliente pero no referencia asignada).
- **Se descarta rehacer la bandeja desde cero**: se usa el mockup de
  Figma que Ángel ya tenía ("bandeja del ejecutivo"), combinando lo
  mejor de ambas versiones vistas: tabs de **alertas**, **accionable
  ahora** (ordenado por compromiso de despacho — requieren seguimiento),
  **en curso automático** (operación automatizada; avisa si se detiene o
  necesita algo), **esperando a terceros** (no requiere acción todavía),
  **por identificar** (entradas sin referencia) — y se **agrega** el tab
  de documentos ("documentos que requieren atención"). A German le
  gustó especialmente el seguimiento a operaciones automáticas.
- **Se elimina la pantalla separada "bandeja de entrada"** y se fusiona
  con "listado de referencias" para no repetir la misma información dos
  veces. Queda: pantalla **"listado de referencias"** con dos tabs —
  **"bandeja de entrada"** (el tablero/cards ya descrito) y **"tabla"**
  (ya no "tabla clásica", solo "tabla", vista tipo SIGAC 2 con filtros/
  reportes/búsqueda) — mientras los ejecutivos se acostumbran al tablero
  nuevo.

### Detalle de referencia — menú y orden de pestañas
- **El expediente aduanero va primero y el DGO al final**, no al revés
  como estaba mockeado. Porqué (aclarado por German con un dibujo): el
  flujo natural es documento → expediente → (validado) → DGO; el DGO es
  la fuente de la verdad ya glosada, no el punto de entrada.
- **Menú "accionable" del detalle** (reemplaza los tabs previos): DGO,
  movimientos, instrucciones, expediente aduanero, operaciones, citas,
  recinto, previo. Dos abiertas:
  - **Instrucciones**: duda de Ángel sobre si sigue haciendo falta a
    este nivel, porque entiende que las instrucciones ya se capturan al
    crear la referencia; agregarlas aquí sumaría complejidad — pendiente
    de que German confirme si sí o no.
  - **Citas**: duda de si necesita tab propio, porque un movimiento de
    entrada generaría automáticamente una cita hacia almacén (el
    ejecutivo no genera citas directamente, genera el movimiento y ese
    movimiento *es* la cita).
- **La línea de tiempo (timeline) no es pestaña propia**: se ve a un
  lado dentro de **Resumen**, junto con ID de referencia, descripción,
  ruta, valor total, cliente, embarques, documentos, operaciones,
  eventos, fecha de creación y última actualización (confirma lo ya
  decidido el 2026-07-07).
- **El asistente de creación de referencia** queda con datos básicos
  (cliente, tipo de operación, tipo de tráfico, ETA, orden de compra,
  etc.) y ya no incluye el paso de "facturas" (se movió, ver más abajo).

### DGO (Datos Glosados para Operación)
- **Crear factura manualmente se mueve del DGO al expediente
  aduanero.** Porqué: el DGO es la fuente única de verdad — ya
  validado y con destino a pedimento. La factura se da de alta primero
  como documento comercial en el expediente; solo cuando CEUS la valida/
  glosa y los datos son correctos, la factura pasa al DGO. Si hay más de
  un DGO en la referencia, el usuario selecciona a cuál se envía esa
  factura (desde la vista de documentos del expediente, donde se puede
  elegir "cuál se va a documento y cuál se va a DGO").
- **Los datos extraídos por CEUS y los datos del DGO coexisten**: no se
  sobreescriben. CEUS compara lo extraído contra lo guardado en el DGO;
  si hay incoherencia entre ambos, salta discrepancia. No pasa nada si
  el número de factura no existe todavía en uno de los dos lados — solo
  se marca error cuando ambos existen y no coinciden.
- **Se puede vaciar/reconstruir una factura DGO manualmente** (crear
  partidas solo con el número de parte) antes de tener el documento
  real; al subir después el documento, CEUS extrae y glosa el JSON
  extraído contra el JSON del DGO, marcando discrepancias; se puede
  sobrescribir el DGO marcando la factura subida como activa.
- Acciones ya definidas por factura DGO: configurar mercancía/partidas
  (número de parte, descripción, UMC, NICO), preferencias arancelarias
  (TLC vía carta TLC, PROSEC), identificadores a nivel partida, campos
  básicos, incrementables/decrementables a nivel pedimento ("globales")
  e identificadores a nivel pedimento. Ángel señala que la UI de esta
  parte se puede mejorar, pero queda fuera de alcance por ahora.
- "Crear factura desde packing list" se deshabilita si ese packing list
  ya fue facturado.

### Movimientos
- **Un movimiento se crea desde el DGO y da entrada a facturas
  específicas del DGO** (no al DGO completo). Al dar entrada, el
  movimiento queda vinculado al DGO por medio de esa factura. Caso que
  lo justifica: un DGO con una sola factura puede requerir **dos
  movimientos de entrada** si el cliente avisa que la mercancía llega en
  dos partes (mitad un día, mitad otro).
- **Tipos de movimiento dependen del tráfico de la referencia** y deben
  mostrarse como tabs fijos aunque no todos se generen desde ahí (se
  monitorean/agrupan igual):
  - **Terrestre:** entrada, previo, salida, subdivisión (4 tabs). Reglas
    seriales: no se puede subdividir sin haber entrado, no se puede
    hacer previo sin haber entrado, no se puede hacer salida sin haber
    entrado — la entrada es crítica en terrestre.
  - **Marítimo:** no requiere "entrada" como en terrestre; empieza con
    un movimiento de "revalidar" mientras el buque arriba (nombre exacto
    no recordado en la reunión, pendiente de confirmar contra el
    documento de entendimiento/descripción de pantallas ya existente).
  - Acuerdo general: **todos los movimientos son seriales**, pero queda
    pendiente revisar caso por caso qué es paralelo y qué es serial por
    tipo de tráfico (German pidió checarlo contra lo ya documentado
    antes de decidir la UI final).
- **Bug detectado en vivo**: al crear movimiento en una referencia
  marítima (referencia de prueba **138**) el sistema mostró "movimiento
  de entrada" (opción de terrestre) en vez de las opciones correctas
  para marítimo — no debería preguntar el tipo de movimiento si la
  referencia ya define su tráfico.

### Instrucciones / Tickets
- **"Instrucciones" sale del detalle de referencia** y vive por fuera
  como una entidad propia de **tickets de servicio**; en el detalle de
  referencia solo se vería la asociación (listado de tickets/
  instrucciones vinculados a esa referencia), de forma similar a como
  se ven los movimientos.
- **Relación aclarada:** un ticket puede estar vinculado a **varias**
  referencias, pero una referencia **no puede** estar vinculada a varios
  tickets a la vez (Ángel lo entendía al revés).
- En el **formulario de creación de referencia** debe quedar la opción
  de vincular un ticket existente o crear uno nuevo ligado a esa
  referencia. Si no se generó ahí, en el **detalle de referencia**
  también se podría asociar a un ticket ya existente, mostrando el
  listado de tickets del mismo cliente de la referencia.

### Expediente aduanero (retomado con dibujo de German)
- **No se quita la sección de documentos** del expediente (propuesta
  de Ángel de simplificar el asistente a solo datos+documentos fue
  descartada). En su lugar: **un mismo componente reutilizable de subida
  de documentos**, usado tanto en el asistente de creación (2 pasos:
  datos de la referencia → subir documentos) como en el detalle/
  expediente aduanero.
- **Cada documento del expediente tiene 3 partes** (ejemplo dibujado:
  factura 1, factura 2, packing list, NOM): **PDF**, **datos
  extraídos**, y **resultado del glosado** (verde/completo si está bien;
  clic para ver el detalle si hay incoherencia). Un documento sin ninguna
  de las tres partes (ej. NOM aún no subida) se muestra vacío, señalando
  que falta — como huecos de un álbum de estampitas.
- **Falta el checklist completo de documentos requeridos** (los "huecos"
  vacíos): debe combinar los documentos obligatorios para todos los
  clientes con los específicos del perfil de ese cliente (leído del
  MOM/perfil de cliente), no solo mostrar lo ya subido.
- **Pendiente por falta de tiempo**: cómo se refleja en UI exactamente
  el momento de "glosar" los datos extraídos contra el DGO (quedó
  explicado a nivel conceptual, ver sección DGO, pero no cerrado en
  pantalla). German tuvo que cortar la sesión para otra reunión.

## Lógica de negocio
- **"Por identificar" (bandeja) vs "guías sin identificar":** por
  identificar hoy muestra todas las referencias sin filtrar por cliente,
  lo cual se considera innecesario tal cual; guías sin identificar es
  específicamente cuando llega un embarque a almacén y no se sabe ni el
  cliente al que pertenece.
- **DGO sigue siendo la fuente única de verdad**: solo lo validado y
  glosado llega ahí; de ahí sale el pedimento.
- **CEUS** (mencionado en la transcripción también como "Zeus/SEUS" por
  error de dictado) es el motor de extracción/glosa con IA, ya usado en
  el detalle de referencia para comparar datos extraídos contra el DGO.

## Tareas
> Detectadas, no creadas. Continúan directamente el rediseño de
> [[2026-07-07 - Revision de pantallas flujo operación]].
- (Ángel) **Rediseñar bandeja del ejecutivo**: renombrar "cartas sin
  firmar" a "documentos que requieren atención", agregar tab de
  documentos, y fusionar con "listado de referencias" en dos tabs
  (bandeja de entrada / tabla).
- (Ángel) **Reordenar pestañas del detalle de referencia**: expediente
  aduanero primero, DGO al final; definir pestañas intermedias.
- (Ángel) **Mover "crear factura manual" del DGO al expediente
  aduanero**, con selección de a qué DGO se vincula cuando hay más de
  uno.
- (Ángel) **Diseñar el checklist completo de documentos requeridos**
  en el expediente aduanero (obligatorios + específicos del perfil de
  cliente) con las 3 partes por documento (PDF / datos extraídos /
  resultado de glosa).
- (Ángel) **Sacar "instrucciones" del detalle de referencia** hacia una
  entidad de tickets de servicio por fuera, dejando solo la asociación
  en el detalle; ajustar el formulario de creación de referencia para
  vincular/crear ticket.
- (Ángel) **Revisar y corregir el bug de tipo de movimiento**: no debe
  preguntar el tipo si la referencia ya define su tráfico (visto en
  referencia de prueba 138, marítima).
- (Ángel) **Definir tabs de movimiento por tráfico** (terrestre: entrada/
  previo/salida/subdivisión; marítimo: iniciar con "revalidar" u otro
  nombre a confirmar) y validar contra el documento de entendimiento
  existente qué movimientos son paralelos vs seriales.
- (Ángel) **Resolver, en una próxima sesión con German, cómo se ve en UI
  el disparo de glosa de datos extraídos contra el DGO** (quedó pendiente
  por corte de tiempo).
- (Ángel) **Confirmar con German** si el tab de "instrucciones" y el de
  "citas" siguen siendo necesarios a nivel detalle de referencia.

## Temas descartados por irrelevantes
- Arranque de la reunión ("a ver, empiezo de nuevo") y muletillas/
  repeticiones propias de la transcripción automática.
- Coordinación de agenda al cierre (German se va a otra reunión de una
  hora, acuerdan retomar después) y despedidas.
