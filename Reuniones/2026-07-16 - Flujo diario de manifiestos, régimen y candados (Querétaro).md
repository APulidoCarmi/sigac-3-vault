# Reunión 2026-07-16 — Flujo diario de manifiestos, régimen y candados (Querétaro)

> **Nota de calidad de la transcripción**: el archivo original venía muy
> degradado, sobre todo en la primera ~hora y media (00:00–01:50): las
> intervenciones de "Angel Huberto Pulido Burgos" y "Sala de juntas
> Queretaro" aparecían duplicadas y desfasadas (dos micrófonos
> transcribiendo la misma frase, cortada de forma distinta cada vez). Esa
> parte se reconstruyó intercalando ambas transcripciones. A partir de que
> Carlos hace la demo en pantalla (~01:50) el texto es sensiblemente más
> claro. Hay además un **hueco de ~7 minutos sin contenido recuperable**
> entre 01:36:37 y 01:43:34. Una cifra (candados de número de parte) se
> mencionó de forma ambigua en el audio; se resolvió a criterio propio con
> su razonamiento explícito (ver "Candado de número de parte") en vez de
> dejarla sin definir.

## Asistentes
- Angel Huberto Pulido Burgos — conduce la sesión, entrevista al equipo de Querétaro sobre su proceso diario.
- Carlos Leobardo Mera Ponce — ejecutivo de tráfico/trámite; hace la demo principal del flujo (manifiestos, referencia, previo, documentos, DODA).
- Iván — ejecutivo (captura, factor de conversión, correos de trámite).
- Dana / Daniela ("DM" en el chat) — ejecutiva.
- Miriam — de Core Global (mencionada, corrigiendo un "Arco" inicial); no se le distingue hablando directamente.
- Daniel — explica en detalle el cálculo de impuestos, TLC, PROSEC, incrementables/decrementables y reglas de manifestación/certificación OA.
- Gilberto Luna Rivera — ejecutivo de la cuenta GKN (Celaya, Guanajuato); hace una demo separada al final (desde 03:53) de un flujo distinto.

> Buena parte de la reunión quedó bajo la etiqueta genérica "Sala de juntas
> Queretaro", que mezcla a Carlos/Iván/Dana sin que siempre se pueda saber
> quién habla en cada turno.

Personas referidas en el proceso (no hablan en la sesión):
- **Adolfo** — almacén/despacho en Querétaro, confirma piezas físicas y da semáforo verde/rojo.
- **Alejandro Canales** — despachador equivalente a Adolfo, en cruce terrestre.
- **Israel** y **Mariana Laguna** — autoridades de clasificación; deben aprobar cuando cambian fracción **y** descripción a la vez (el clasificador solo no puede).
- **Brenda** (ingeniera) — a cargo de la lógica del "candado" de número de parte; Ángel dice ya haberla consultado.
- **Germán** — otro responsable del proyecto, se retomarán ideas con él ("en la tarde lo vemos con Germán").
- **Juan Flores** (ingeniero) — destinatario de correos sobre clasificación.
- **Erica** — reportó errores de captura (factor UMT en Guadalajara; fecha de factura en Manzanillo).
- **Verónica ("Vero")** — contacto de transporte.
- **Yaqueline Jiménez** — contacto de GKN en Celaya, avisa anticipadamente de cargas.
- Cliente referido indistintamente como "Taiko"/"Tyco"/"Tico"/"Tao" — casi con certeza **Tyco Electronics México / TE Connectivity**.

## Decisiones (con su porqué)

### El correo de "prealerta" es exclusivo de Tyco; el de "manifiesto" es el que opera todo
El correo de la aerolínea (DHL, ~7:00 am, ejemplo: manifiesto de lo que sale de Cincinnati hacia Querétaro con ~99% de probabilidad de arribar ese día) es un adelanto dirigido solo al cliente — "eso nada más pasa con Taiko". El correo de "manifiesto de carga" de Terminal/almacén (~8:00 am, ejemplo puntual recibido a las 7:34 am) incluye a **todos** los clientes y es el que realmente dispara el trabajo de Carmi. Se manda a `carmiqro@carmi.com`, con Carlos, Iván y Dana en copia.

### No se puede mezclar régimen temporal y definitivo en una misma operación
Regla dura del sistema aduanero (no una limitación técnica arbitraria): cada régimen requiere su propio pedimento/referencia/operación.

### Tampoco se pueden mezclar guías de distinta guía máster en la misma referencia
Aunque compartan régimen. Ejemplo: dos consolidados temporales del mismo día, de vuelos/máster distintos, requieren dos referencias/pedimentos separados.

### Regla de Tyco: guía "parcial" no se libera hasta estar completa
Salvo autorización explícita del cliente para liberar lo parcial. Motivo: evitar declarar mercancía que aún no ha arribado físicamente. Caso real trabajado en la sesión: guía llegada el miércoles 15 de julio con 31 de 32 bultos — el complemento (1 bulto, 1.395 kg) **seguía sin llegar el día de la reunión** (16 de julio), 8 días después. Es un caso operativo abierto, no resuelto en la sesión.

### Mapeo 1 referencia = 1 pedimento = 1 operación, confirmado explícitamente
Se aclaró una confusión larga sobre por qué los tres ejecutivos generan operaciones "separadas": es correcto, porque corresponden a regímenes o guías máster distintas, no a un problema de proceso. El sistema **no soporta relación padre-hijo entre referencias**: cuando una guía llega parcial y ya existía una referencia completa, la solución manual es "subdividir" — crear una segunda referencia independiente (convención de nombre: sufijo tipo `referencia_1`) para lo que falta, y hacer pedimento aparte de lo que sí llegó. Cuando llega el complemento, se pedimenta la segunda referencia.

### Documentos: se prefiere subir agrupados, no uno por uno, salvo excepciones
Aunque el sistema podría permitir subir documento por documento con ID propio, el equipo prefiere un solo ID consolidado por tipo de documento (ej. unir todas las facturas de un pedimento en un solo PDF), porque declarar muchos IDs de documento por pedimento (ej. 16 facturas = 16 IDs) sería impráctico en trámite. Se acordó explorar que el sistema automatice esa unión al generar el pedimento, dejando la opción de envío individual para casos como la manifestación de valor (que usa 3 documentos que pueden unirse o no). **No quedó como decisión de diseño cerrada**, sigue abierto.

### La unidad de transporte solo se solicita cuando ya hay certeza de despacho
Es decir, cuando ya hay al menos un pedimento pagado (o certeza total de que habrá despacho ese día), nunca antes. Motivo explícito: si se solicita la unidad y luego no hay despacho por causas externas (ej. caída del sistema bancario/BUEL), el transportista reclama el costo de haber puesto la unidad sin uso.

### GKN pide manifestación de valor aun estando certificado OA
No están obligados por su certificación — es una decisión propia del cliente, no un requisito legal.

### Validador distinto por cliente
GKN usa "ANIC"/"Anique" en vez de "Karem" porque el cliente lo exige explícitamente (servicio de un tercero externo); Tyco usa Karem.

## Lógica de negocio

### Flujo diario de recepción de manifiestos (DHL / Terminal)
1. ~7:00 am: correo de DHL con manifiesto (tabla + ZIP con guía y factura por partida) — "prealerta", exclusivo de Tyco.
2. ~8:00 am (ejemplo puntual: 7:34 am): correo de "manifiesto de carga" de Terminal, incluye a todos los clientes — este es el que dispara el trabajo real.
3. ~8:30 am: con ambos correos en mano, arranca el proceso.
4. Ese día llegaron dos vuelos: uno de DHL (Cincinnati, EE.UU.) y ocasionalmente uno de FedEx ("Fedes"/FX, empezó a llegar hace ~2-3 meses, sin día fijo — puede ser miércoles y/o sábado). También se mencionó un vuelo de Iberia (España), que llega miércoles y sábado o lunes, sin patrón fijo. Vuelo puntual mencionado: **783, procedente de Madrid**.
5. El asunto del segundo correo trae una referencia interna de DHL (ejemplo: **referencia 3894**) — es control interno de DHL, no algo que Carmi comparte ni usa.

### Filtros manuales antes de poder capturar una referencia
1. **Destinatario/cliente**: se descartan del manifiesto las guías que no son de Tyco (ejemplo: se distinguen guías de "Tyco Electronics México" de otras con destinatario "CM").
2. **Régimen (temporal vs. A1/definitivo)**: el manifiesto de DHL no trae indicador directo; se determina número de parte por número de parte consultando un catálogo interno (cargado originalmente "por los ingenieros hace no sé cuántos años"). Se abre el archivo de "detalle" del cliente (entidad "CRL"/"CB" de Tyco Electronics de México) y se revisa la pestaña de "rotación temporal".
3. **Completitud de guía (parcial vs. completa)**: se revisa manualmente en la página de rastreo de DHL si el número de bultos documentado coincide con lo recibido físicamente (ejemplo: guía con 7 de 7 bultos completa vs. la guía de 31/32 mencionada arriba).
4. **Candado de número de parte** (detalle abajo).

### Candado de catálogo de número de parte
El catálogo de números de parte tiene un "candado" cuando la clasificación (fracción arancelaria + descripción) está desactualizada.
- **Escala aproximada**: sobre un catálogo de ~1 millón de números de parte, el orden de magnitud con candado es de **~5,000** (0.5% del catálogo). En el audio se alcanza a escuchar primero "500,000", pero es casi con certeza un traspié/autocorrección de quien hablaba — la propia intervención se corrige a "5000" acto seguido, y la cifra de 5,000 es consistente con el resto de la conversación (los candados se describen como una excepción puntual que se resuelve por correo caso por caso, no como un bloqueo de la mitad del catálogo, lo cual haría inviable la operación diaria descrita). **Confirmar con el cliente de todas formas** antes de usar esta cifra en cualquier estimación de negocio.
- Al detectar candado durante la verificación de régimen, se manda correo al clasificador y al equipo de Tyco (copiando al ingeniero Juan Flores) para liberarlo. Solo el clasificador puede aprobar cambios de fracción por sí solo; si también cambia la **descripción**, se requiere aprobación adicional de Israel o Mariana Laguna, lo que alarga el proceso porque no siempre responden rápido.
- **Regla de tiempo crítica**: el corte de despacho es 2-3 pm (último pedimento pagable ese día). Para pagar hay que validar antes, y para validar hay que tener ya la proforma. Si a la 1 pm algo no está clasificado, se descarta para el día y se reintenta al siguiente — genera almacenaje extra para el cliente.
- **Duda abierta planteada por Ángel, sin resolver**: cuestiona si el paso de aprobación humana realmente aporta valor, dado que en la práctica el cambio de clasificación casi siempre termina autorizándose, y el paso solo agrega demora (horas o incluso "todo el día") al despacho.
- **Idea discutida, no implementada**: que el sistema haga un barrido periódico automático (diario/semanal) de todo el catálogo para detectar candados antes de que llegue la mercancía, notificando proactivamente a Tyco y al clasificador. También se mencionó que Tyco ya tiene un sistema propio para solicitudes de aceptación de números de parte, pero es complicado de usar ("cruzas un laberinto"); se propuso que SIGAC muestre directamente esa info (partes con candado) en una pantalla simple para que el cliente decida.

### Estructura de referencia / pedimento / operación
- Dentro de una referencia pueden convivir varias guías/facturas de distintos proveedores; el equipo las organiza visualmente con colores (ej. verde para un vuelo/máster, rosa para otro) como método manual, no como función del sistema.
- Ejemplo de captura trabajado: un número de parte con dos posibles proveedores dentro de la misma factura (uno de Brasil, otro identificado como "de Suiza"/Connectivity Solutions, aunque el exportador en documentos puede decir "Estados Unidos"). Conclusión: **el campo "guía" no puede ir a nivel referencia si hay más de un proveedor** — debe ir a nivel factura o partida.
- Campos comparados contra el "previo" (verificación física en almacén): peso, cantidad de bultos, piezas, marca, modelo (si aplica, ej. motores), origen. El número de serie solo se valida si la mercancía es "susceptible de identificarse individualmente" (ver reglas de Daniel abajo). Se discutió automatizar esta comparación para que el sistema solo alerte discrepancias.
- Se identificó, sin definirse a detalle, la necesidad de un concepto de **"movimiento de salida"** en SIGAC 3 que agrupe varios pedimentos/operaciones en un solo DODA/aviso de cruce: un mismo camión puede llevar mercancía de hasta 10 pedimentos distintos (ejemplo: 10 pedimentos, 160 bultos, un solo transporte), y el sistema actual está diseñado para sacar "una operación a la vez", generando fricción. Diseño abierto, sin solución definida.

### Documentos y digitalización
Los documentos llegan mezclados (no separados por guía); el equipo abre cada archivo manualmente y detecta a qué guía/referencia corresponde antes de subirlo. Ejemplo real: 7 facturas de temporal unidas en un solo archivo, incluyendo una de Brasil y una de Corea. Documentos típicos: factura, guía, póliza, "regla 3.1.8" (anexo obligatorio para cierto cliente), factura de fletes.

### Reglas aduaneras (explicadas por Daniel)
- Marca/serie/modelo/país de origen solo son obligatorios para mercancía "susceptible de identificarse individualmente" (maquinaria, laptops, celulares, motores con número de serie); no aplica a materia prima genérica, aunque a veces la etiqueta física trae un código similar a un número de serie (caso raro reportado por Gilberto, pendiente de revisar con casos concretos).
- Para régimen **temporal**, adicionalmente no se exige declarar marca/modelo/serie (dispensa específica del régimen, aunque la propia explicación de Daniel quedó algo ambigua sobre si aplica de forma general).
- En **temporal**, el número de parte debe coincidir exactamente entre lo físico, la factura y lo capturado (afecta Anexos 24 y 31 — entradas/salidas de mercancía temporal). En **definitivo** puede variar sin consecuencia.
- **Cálculo de impuestos del pedimento**:
  - **IGI** (arancel): según fracción (0/5/10/15/20/25%), reducible por TLC o por PROSEC (requiere RFC del cliente autorizado ante el validador, si no el validador rechaza el pedimento). Tyco no aplica PROSEC.
  - **DTA** (art. 49 Ley Federal de Derechos): valor en aduana × 8 al millar, con cuota fija mínima de **~$462 pesos** si el cálculo da menos.
  - **IVA**: (valor en aduana + DTA) × 16%.
  - **Valor en aduana** = valor de mercancía en USD × tipo de cambio del **día de entrada/cruce físico** (no el día de pago) + incrementables (fletes/seguros previos al cruce).
  - **No incrementables** (ej. flete contratado por el cliente después del cruce) no forman parte de la base gravable (art. 65 — "ni la misma autoridad lo entiende", según Daniel).
  - **Decrementables**: prácticamente nadie los usa ni los entiende bien, según Daniel.
- Errores de captura reales reportados por Erica: factor UMT incorrecto en Guadalajara; fecha de factura mal capturada (20 de marzo en vez de 20 de abril) que afectó la aplicación del TLC en Manzanillo.
- **Factor de conversión**: se usa cuando la unidad de la fracción arancelaria (ej. kilos) no coincide con la unidad de facturación del cliente (ej. piezas o "juegos"/sets). Ejemplo: Tyco factura casi siempre en "piezas" aunque venga en "juegos" (conectores en sets); al detectar que es un "juego", hay que definir el factor manualmente (ejemplo: 1 juego = 2 piezas), lo que a veces genera el error "falta factor de conversión" porque el sistema no lo recalcula solo tras el cambio manual de unidad.

### Secuencia de despacho posterior al pago
Glosa → Manifestación (si aplica) → Validación (Karem para la mayoría; ANIC/Anique para GKN) → Pago (cuenta de orden/cuenta PC; normalmente Tyco financia directo, con insuficiencia de fondos ~1 vez al año) → Correo a trámite y despacho avisando pedimentos pagados (uno por uno conforme se pagan, no en batch) → Trámite comparte datos del transportista (solicitado solo cuando ya hay al menos un pedimento pagado) → Se anexa la guía liberada al Excel concentrado entre los tres ejecutivos, que eventualmente se envía al cliente → DODA / Aviso de Cruce (el Aviso de Cruce debía reemplazar al DODA pero falla frecuentemente — "cada rato se cae" — por lo que se sigue usando el DODA como respaldo) → Espera de modulación (semáforo), confirmada por el despachador en el puente (Adolfo en Querétaro; Alejandro Canales en cruce terrestre, rol equivalente pero "más macro").
- Límite físico de transporte: ~14 palets por unidad; si sobran bultos, se consulta al cliente si prefiere una segunda unidad el mismo día o esperar al siguiente. Regla práctica: bulto de más de 30 kg cuenta como "palet".

### Reportes manuales del equipo (~4-6, sin documentación formal completa)
- **Reporte de parcialidades**: Excel con consecutivo, fecha de llegada, fecha de llegada del complemento, estatus (liberada=verde, pendiente=blanco). Se propuso convertirlo en histórico/dashboard dentro de SIGAC 3, visible para el cliente, en vez de reporte enviado (es acumulativo, no puntual).
- **Reporte de daños**: cuando llega carga dañada/mojada, Terminal manda correo con foto y folio propio; Carmi renombra con folio consecutivo interno y reenvía al cliente. **No es formalmente responsabilidad de Carmi** (problema de manejo de la aerolínea) — lo hacen como cortesía. Propuesta: automatizar detección del correo, folio consecutivo estandarizado por cliente y reenvío automático.
- **Listado/concentrado diario de guías a salir**: cada ejecutivo anexa sus guías liberadas a un Excel compartido; una vez completo se envía al cliente. Parece exclusivo del equipo aéreo (Ángel no lo ha visto en marítimo/terrestre).
- **Reporte de status/pendientes de despacho** con días de almacenaje acumulado, enviado periódicamente al cliente.
- **Reporte específico de GKN**: quedó pendiente de que Carlos/Gilberto lo expliquen columna por columna (no se alcanzó en la sesión).

### Flujo GKN (Gilberto Luna Rivera) — contraste con el flujo Tyco
- GKN fabrica flechas homocinéticas de autopartes ("el proveedor más grande a nivel mundial" de ese componente, usado por todas las marcas de autos).
- En ~10% de los casos, el cliente (contacto: Yaqueline Jiménez, planta de Celaya) avisa antes de que llegue la carga, indicando incluso el régimen y compartiendo documentos (facturas, orden de compra) anticipadamente. El otro ~90% depende 100% del manifiesto de Terminal, igual que Tyco.
- GKN es descrito como "moroso" para dar instrucciones de despacho: puede tardar 2 días en el mejor caso y hasta 20-50 días en instruir el despacho de facturas atoradas, hasta que surge urgencia y piden liberar todo de golpe. A diferencia de Tyco, GKN mantiene inventario propio en planta, por lo que no depende de un flujo diario ("hoy llega, hoy sale").
- GKN usa transportista propio asignado por convenio a nivel planta; Carmi no puede usar otro transportista sin autorización, y solo despacha cuando GKN manda explícitamente "instrucciones de despacho", incluso si la documentación ya está lista para salir el mismo día.
- Riesgo real mencionado: mercancía de GKN en Laredo con flechas represadas hasta 5 años pagando almacenaje, al punto que resultó más barato **destruir** la mercancía que seguir pagando almacenaje.
- Órdenes de compra de GKN pueden repartirse en múltiples referencias/operaciones (ejemplo: una orden de 1 millón de piezas repartida en varias entradas parciales).
- GKN pidió tener visibilidad del número de referencia interno de Carmi desde el inicio de la operación, para consultarlo ellos mismos en la plataforma (en vez de depender de que Carmi comparta el pedimento provisional por correo). Pendiente de resolver directamente con el cliente, con apoyo de Germán.

## Problemas y riesgos abiertos

- **Cifra de candados a confirmar con el cliente**: se estima ~5,000 números de parte con candado sobre un catálogo de ~1 millón (ver razonamiento en "Candado de catálogo de número de parte"); el audio mencionó de pasada "500,000" pero todo indica que fue un traspié corregido en la misma intervención.
- **Valor del paso de aprobación humana en candados de clasificación (Israel/Mariana Laguna)**: Ángel cuestiona si aporta valor real dado que casi siempre termina aprobándose, solo agregando demora — sin decisión tomada.
- **Diseño no cerrado**: cómo implementar el "movimiento de salida" que agrupe varios pedimentos en un solo DODA/aviso de cruce sin romper el modelo actual de "una operación a la vez".
- **Ambigüedad sin aclarar del todo**: si la dispensa de declarar marca/modelo/serie aplica de forma general al régimen temporal o depende también de la naturaleza de la mercancía.
- **Documentos unificados vs. individuales en VUCEM/glosa**: sin decisión formal de diseño, solo una idea de punto medio (automatizar unión para pedimento, permitir individual para manifestación).
- **Flujo completo de reportes de GKN y el tema del "vuelo charter"**: no se alcanzaron a revisar por corte de tiempo (el equipo se fue a comer).
- **Supuesto sin confirmar**: que el Aviso de Cruce reemplazará definitivamente al DODA — actualmente falla con frecuencia, no está claro cuándo se estabilizará.
- **Caso operativo real sin resolver** (no es tarea de sistema): la guía parcial con complemento de 1.395 kg seguía sin llegar 8 días después, al momento de la reunión (16 de julio).

## Tareas
> Detectadas, no creadas (este cliente gestiona tareas fuera de `/reunion`); esta reunión se usa solo como contexto, por indicación explícita del usuario.

- Automatizar el cotejo del manifiesto de DHL vs. el manifiesto de Terminal (comparación de guías) — sin responsable asignado; Ángel lo revisaría con Germán.
- Revisar/refinar la lógica del "candado" de número de parte con la ingeniera Brenda (ya en marcha).
- Correo pendiente al clasificador + equipo Tyco (copiando a Juan Flores) para liberar candados de fracción detectados.
- Automatizar folio consecutivo estandarizado para el reporte de daños y su reenvío automático al cliente.
- Convertir el reporte de parcialidades en histórico/dashboard dentro de SIGAC 3, visible para el cliente.
- Pedir a Carlos/Iván/Dana que documenten columna por columna cada uno de los ~4-6 reportes manuales que generan.
- Diseñar el concepto de "movimiento de salida" que agrupe varios pedimentos bajo un solo DODA/aviso de cruce.
- Agregar botón "solicitar información sobre la guía" desde la pantalla de guía, con trazabilidad de cuándo se solicitó y cuánto tardó el cliente en responder (propuesto para GKN, aplicable en general).
- Revisar con GKN compartir el número de referencia interno desde el inicio de la operación, con apoyo de Germán.
- Revisar casos concretos de etiquetas con número similar a serie sin que debiera requerirse (Gilberto se comprometió a compartirlos).
- Retomar el tema del "vuelo charter" (mencionado al cierre, sin desarrollar).
- Explicar formalmente el reporte específico de GKN, columna por columna.
- Explorar uso de IA/Gemini para automatizar cotejo de manifiestos y generación de reportes por cliente (se mencionó precedente de Cristina Aguilar en Laredo, procesando ~700 requerimientos/semana con un chat de Gemini con contexto propio).

## Temas descartados por irrelevantes
- Fallas técnicas recurrentes de audio/conexión/pantalla compartida durante toda la sesión.
- Charla sobre pedidos de café/bebidas del equipo.
- Charla sobre setup de monitores/oficina en casa.
- Logística de la comida del equipo (a qué restaurante ir).
- Comentarios sobre calidad de señal WiFi/5G en la sala.
- Bromas y comentarios personales sueltos.
- Duplicaciones de habla del patrón "Angel Huberto Pulido Burgos" / "Sala de juntas Queretaro" (mismo audio transcrito dos veces), una vez reconstruida la frase real.
- Hueco de ~7 minutos (01:36:37–01:43:34) sin contenido recuperable en el archivo original.
