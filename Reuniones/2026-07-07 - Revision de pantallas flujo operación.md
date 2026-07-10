# Reunión 2026-07-07 — Revisión de pantallas del flujo de operación

Revisión de diseño/UX de las pantallas de SIGAC 3.0 para el flujo de operación
entre **Ángel** (desarrollo) y **German Castro** (producto/arquitectura). German
guía cómo reestructurar la creación de referencia, el **expediente aduanero**, la
**glosa digital** y el modelo de **Datos Glosados para Operación (DGO)** como
fuente única de verdad. Relacionada con [[2026-07-02 - Aéreo Sigac 3.0]]
(referencia sin facturas obligatorias, CEUS).

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo — presenta los mockups)
- German Castro (producto / arquitectura — dirige la revisión)
- Mencionados (no participan activos): Elián (arregla el inicio de sesión),
  Adriana (valida documentos de onboarding), Brenda (datos de manifestación de
  valor, de una reunión previa), Carlos (clasificador).

## Decisiones (con su porqué)
- **La creación de referencia debe ser rápida: 3 pasos, no 4.** Nuevo orden:
  (1) **datos básicos**, (2) subir documentos a **CEUS**, (3) **auditoría de la
  extracción**. Porqué: el paso 4 actual ("documentos comerciales") es muy
  extenso y hace parecer obligatoria toda esa captura; además la regla de
  auditoría exige datos básicos primero. Ese paso 4 **se saca del asistente y se
  mueve al detalle de la referencia** como sección.
- **Dos sabores de creación:** de cara al **cliente** (portales tipo Nestlé:
  arrastra documentos → se audita → se crea la referencia y devuelve número) y de
  cara al **ejecutivo** (arranca con cliente, tráfico e instrucciones, luego sube
  documentos y audita). Las opciones de arranque siguen siendo **con documentos /
  anticipada / virtual**.
- **El detalle de referencia se estructura en dos pestañas principales:**
  **Expediente aduanero** y **Datos glosados para operación (DGO)**, además de
  **Resumen**. Porqué: separar "todos los documentos" de "la fuente de verdad
  que alimenta la operación".
- **Hay que glosar TODO el expediente aduanero, no solo documentos elegidos.**
  Porqué: hoy la autoridad **glosa con IA todo lo que se le envíe**; en SIGAC 2
  el ejecutivo elige sobre qué glosar, y eso ya no basta.
- **La glosa se dispara sobre todo el expediente cada vez que entra o se genera
  un documento** (trigger). Cada documento nuevo (COVE, MV, pedimento) se
  **regresa al expediente** y se re-glosa contra todo, para cazar discrepancias
  (ej.: un COVE que trae 1.006 cuando debía ser 1.00).
- **La glosa se hace sobre la representación digital (JSON) extraída, no sobre
  los PDF/fotos.** Esa representación es la que se mete al contexto de CEUS.
- **DGO como fuente única de verdad para armar la operación.** De ahí salen
  e-documents, COVEs, manifestación de valor electrónica y pedimento; al final se
  valida que el pedimento concuerde con MV, COVEs, e-documents, factura, packing
  list, NOM y verificación.
- **Se mantiene la glosa manual como respaldo.** Si CEUS falla/no está, el
  ejecutivo llena el DGO manualmente y **alguien (glosador) valida y firma** un
  checklist antes de operar (como hoy: se pasa a un glosador que devuelve un
  formato de glosa). El cliente quiere a futuro **glosa solo con CEUS** (caro,
  pero más barato que multas / notas de crédito).
- **Primero el "happy path".** Definir los DGO y hacer funcionar el caso simple,
  probarlo entre Ángel y German **antes** de validar con ejecutivos, y luego ir
  agregando los casos complejos. No intentar resolver todo de golpe.
- **Se mantienen ambas vistas del listado de referencias en curso:** el nuevo
  **tablero/lista** de Ángel (top de las más urgentes) como primaria y la **tabla
  clásica** (estilo SIGAC 2, con filtros, reportes, búsqueda) como secundaria, en
  dos tabs, mientras los ejecutivos se acostumbran.
- **Renombrar pestañas para que se entiendan:** "documentos" → **expediente
  aduanero**; "mercancía" → **documentos comerciales glosados / DGO**; "costos
  globales" → **gastos** (con incrementables/decrementables); "configuración
  legal" → **identificadores**. Evitar tener a la vez "documentos" y "documentos
  comerciales" (confunde).
- **Agregar el campo "orden de compra" a la referencia.** Porqué: el cliente
  busca sus operaciones por su orden de compra en la vista de cliente.
- **La línea de tiempo desaparece como pestaña** y se muestra dentro de Resumen.
  Breadcrumbs a corregir: "referencias › GKN Drive › referencia" debe ser solo
  "referencias › referencia".

## Lógica de negocio y conceptos

### Terminología acordada
- **CEUS:** motor de extracción y glosa con IA (en la transcripción aparece como
  "Zeus/SEUS/CEDUS" por errores de dictado).
- **Expediente aduanero:** conjunto de **todos** los documentos de la operación
  (los que da el cliente y los que genera Carmi). Es un checklist que se arma
  juntando **perfil de cliente + perfil del número de parte + operación**; marca
  qué es requerido/opcional y si está cargado.
- **DGO (Datos Glosados para Operación) / "factura DGO":** la **fuente única de
  verdad**. Representación de la verdad de las partidas, la factura y datos
  comerciales anexos (los que se necesitan para operar), ya glosados.
- **Documentos de embarque vs documentos internos** (vista de cliente): los de
  embarque son válidos para auditoría de la autoridad (los usados + los que
  genera Carmi con validez); los internos no tienen validez legal (oficios
  emitidos, glosa del pedimento, conteo/inventario — solo operativos de Carmi).

### Expediente aduanero (pestaña)
- Cada documento muestra: **tipo**, **por qué se pide** (fundamento legal / perfil
  de cliente / perfil del número de parte), **requisitos de validación**,
  **vigencia**, campo para subirlo, **botón de historial**, **estatus de glosa**
  (pendiente / glosado; fases 1-4; verde = ok, rojo = error con motivo),
  **timestamp** y la **representación digital** de lo extraído.
- **UI dinámica** (no tarjetas fijas como en onboarding): se avientan N
  documentos en una sola bolsa y **CEUS los clasifica/tipifica**; puede ser un
  file-system/lista. Las tarjetas fijas sirven en **onboarding** porque ahí los
  documentos requeridos son fijos (como llenar un álbum), pero aquí la operación
  es dinámica.
- **Selección primaria/secundaria** entre documentos del mismo tipo (ej. 3
  facturas subidas distintos días; se marca la que sí se usa como primaria/
  activa). Se pueden **ligar** documentos (uno reemplaza a otro, como en tickets).
- **Historial a nivel expediente**, no por documento: se guardan logs
  (se subió, se glosó, se marcó primaria, se desactivó…).
- **Borrado lógico:** aunque el ejecutivo borre un documento de su vista, queda
  atrás para auditoría (quién/cuándo lo subió, si estaba mal, cuándo se corrigió).
- **Hay documentos sin extracción de datos** (van en el expediente igual).
- No se puede iniciar una operación si la factura enviada tiene algún problema.

### DGO / factura DGO (pestaña)
- **Un DGO por factura.** Cada factura DGO tiene **partidas**; la partida
  concentra **todos** los datos, sin importar de qué documento salieron: número
  de parte, fracción arancelaria, descripción, si aplica **TLC**, valor,
  **PROSEC**, **identificadores**, **factor de conversión**, precio, etc.
- Los datos se **glosan contra los documentos del expediente**. El ejecutivo
  puede meter datos manualmente y cada cambio se glosa (ej.: si teclea un
  proveedor que no coincide con el de la factura extraída, salta foco rojo).
- **Ejemplos de niveles:** TLC se aplica **a nivel partida** y viene de una
  **carta TLC** (no de la factura); el clasificador marca que aplica. El **factor
  de conversión** va a nivel partida. **Costos globales** van a **nivel
  referencia**; **identificadores** existen a nivel referencia **y** a nivel
  partida.
- Se puede **reconstruir una factura DGO manualmente** (crear factura, crear
  partidas con solo el número de parte) antes de tener el documento; al subir
  luego la factura del cliente, CEUS extrae y **glosa el JSON del DGO contra el
  JSON extraído** y marca discrepancias. Se puede sobrescribir el DGO al marcar
  la factura subida como activa.
- **"Acciones" por factura DGO** = herramientas en drawer para enriquecerla:
  editar partidas (campos básicos, identificadores), **conversión de mercancías**,
  **gastos** (incrementables/decrementables), **configuración legal /
  identificadores**. Los datos a nivel referencia se dejan a nivel referencia.

### Generación de la operación
- La operación se sigue generando **desde el movimiento de entrada**, que a su vez
  se crea a partir de las **facturas (DGO)** — esto no cambia.
- **Nuevo paso al crear la operación: confirmar qué documentos van a esa
  operación/pedimento.** El ligue del DGO pre-llena los documentos relacionados;
  el sistema pide confirmar y, si se agregan otros, **re-glosa** en ese paso antes
  de arrancar. Resuelve casos como: una referencia con varias facturas/
  movimientos donde se genera **un pedimento por factura** o se **divide** (un
  pedimento para la factura con permiso, otro para el resto).
- **Ligar cada documento (p. ej. un permiso) a la factura/partida a la que
  pertenece.** Hoy el sistema nunca sabe a qué pertenece un permiso; SIGAC 3 debe
  rastrearlo para llevarlo al pedimento correcto. (Ángel insistió en este caso;
  German pidió no complicarlo y dejarlo para después del happy path.)

### Tablero / bandeja de entrada (le gustó a German)
- Arriba, **auditoría de CEUS** de lo que falta (folios, etc.).
- **Referencias en curso** (tabs: automático / temporal / avanzado), operaciones
  automáticas y las que **esperan a terceros**: clasificación (ref. 47), previo/
  almacén (ref. 81), arribo (ref. 68), **cartas sin firmar 3 días** del cliente
  **Enalco**, "por identificar".
- **Guías llegadas sin identificar** que se le muestran a todos para reconocerlas
  (ej. posible GKN) y **ligarlas o crear una referencia**.

### Vista de cliente y expediente completo
- Pantalla de operaciones → operación reciente → **drawer** con dos tabs:
  **documentos de embarque** y **documentos internos**.
- El cliente **busca por orden de compra**.
- **"Expediente completo"** descarga un **ZIP** con: XML del pedimento,
  comprobantes de depósito del cliente, factura emitida por Carmi, hoja de cálculo
  del SAT para el valor aduanal, archivos enviados al SAT, formato de pedimento,
  COVEs y relación de COVEs, BL, etc.

### Onboarding (mostrado como referencia de diseño)
- Link que se comparte al cliente para dar de alta documentos. Tarjetas por
  documento: evidencia de no existencia, ver detalle, fundamento legal, requisitos
  de validación, vigencia, subir, historial. **Adriana** valida (estados: en
  revisión / listo / error). Aplica a alta de clientes; su patrón de
  checklist/fundamento inspira el expediente aduanero.

## Tareas
> Detectadas, no creadas. Son, sobre todo, el rediseño que **Ángel** rehará en
> los mockups. Guardan relación con las tareas ya detectadas en
> [[2026-07-02 - Aéreo Sigac 3.0]] (referencia sin facturas obligatorias, CEUS).
- (Ángel) **Rehacer los mockups del flujo de operación** con todo lo acordado y
  mostrárselos a German. **Restan 4 días.**
- (Ángel) **Reestructurar la creación de referencia a 3 pasos** (datos básicos →
  subir a CEUS → auditoría de extracción) y **mover el paso 4** ("documentos
  comerciales") al detalle de la referencia.
- (Ángel) **Diseñar el detalle de referencia** con pestañas **Resumen /
  Expediente aduanero / DGO**, aplicando los renombres (expediente aduanero,
  documentos comerciales glosados/DGO, gastos, identificadores) y corrigiendo
  breadcrumbs y línea de tiempo.
- (Ángel) **Expediente aduanero:** repositorio dinámico con metadatos por
  documento (tipo, fundamento, requisitos, vigencia), estatus de glosa, historial
  a nivel expediente, borrado lógico y selección primaria/secundaria.
- (Ángel) **Glosa digital sobre todo el expediente** con trigger de re-glosa al
  entrar/generar cada documento; representación digital (JSON) a CEUS.
- (Ángel) **Modelo DGO / factura DGO** con partidas que concentran todos los
  datos y "acciones" por factura (editar partidas, conversión de mercancías,
  gastos, identificadores).
- (Ángel) **Nuevo paso de selección de documentos por operación/pedimento** y
  **ligado permiso→factura/partida** (después del happy path).
- (Ángel) **Glosa manual de respaldo** con checklist de validación y firma cuando
  CEUS no esté disponible.
- (Ángel) **Vista de cliente** con documentos de embarque vs internos y descarga
  de "expediente completo" (ZIP).
- (Ángel) **Agregar el campo "orden de compra"** a la referencia.
- (Ángel) **Mantener la tabla clásica** como tab secundario junto al tablero
  nuevo.

## Temas descartados por irrelevantes
- Problemas técnicos del meet (audio cortado "no te escucho", "ya puedes ver mi
  pantalla") e interrupción personal de German a mitad de reunión.
- Incidencia del **límite de llamadas del MCP de Figma** que dejó a medias los
  mockups, y el arreglo del **inicio de sesión por Elián** (estado de tooling; se
  conserva como contexto de por qué la demo quedó incompleta).
- Small talk, despedidas y pie de página/encuesta de Gemini.
