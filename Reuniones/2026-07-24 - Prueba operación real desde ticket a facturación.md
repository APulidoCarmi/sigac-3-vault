# Reunión 2026-07-24 — Prueba operación real desde ticket a facturación (aéreo, Querétaro/Taico + Marítimo/Manzanillo)

## Asistentes
- Angel Huberto Pulido Burgos (Ángel) — demo en pantalla
- German Castro (Germán)
- Daniel Aguila
- Carlos Leobardo Mera Ponce
- Danae Gómez Rios
- Cesar Aguirre
- Rocio Teresa Cortes Salazar
- Carlos Alexis Galaviz Rosas
- Diana Laura Figueroa Jimenez
- Adriana Margarita Eguia Guerrero, Brenda Rentería, Elian Shair Armendariz Puch, Enrique Lopez, Fernando Angel Lopez Soto, Jesus Vega Portillo, Juana Maria Gonzalez, Lilia Candelaria Jimenez Serrano, Maria Maythe Resendiz, Mariana Lagunes Moreno, Miguel Gomez, Perla Lopez (conectados, sin intervención relevante registrada)

Nota: la sesión cubrió dos bloques con público distinto — la primera mitad con el equipo de Taico/aéreo (Daniel, Carlos Leobardo, Danae, Cesar), la segunda con Rocío de marítimo/Manzanillo (GKN).

## Decisiones (con su porqué)

### Régimen vs. clave de pedimento — doble select
Se mantiene el campo régimen a nivel guía, pero se agrega **clave de pedimento** como campo dependiente (dos selects en cascada: régimen → claves de pedimento disponibles para ese régimen).

**Por qué:** Daniel explicó que un régimen (ej. definitivo, temporal) puede corresponder a varias claves de pedimento del apéndice 2 del anexo 22 (ej. definitivo: A1, F4, A3, A1 exportación; temporal: IN, AF, B1, V1 — pedimentos virtuales). Taico en su Excel solo registra "régimen", pero Ángel señaló que un mismo régimen no siempre implica la misma clave, por lo que el campo régimen solo no basta. Germán propuso resolverlo igual que en SIGAC 2: el perfil de entidad del cliente ya restringe qué régimen/claves puede usar cada cliente (de ~50 claves posibles solo se habilitan las que ese cliente usa), y desde ahí se filtra el segundo select.

### Un DGO = un pedimento; una selección de guías con régimen mixto genera múltiples DGOs
Al crear referencia desde una selección de guías, si todas comparten régimen se genera **un solo DGO**; si hay regímenes distintos en la selección, se genera **un DGO por régimen** (ej. IMD + ITR → 2 DGOs).

**Por qué:** Carlos Leobardo señaló que esto no cuadraba con lo acordado hace 8 días ("una operación = un solo pedimento" para Taico), porque Taico reparte las guías/pedimentos por régimen entre su equipo. Ángel ajustó la demo al caso real de Taico: seleccionar solo las guías de un mismo régimen (las que le tocan a ese ejecutivo) y generar un único DGO con un solo régimen — confirmando que el comportamiento se ajusta según cómo cada cliente/equipo reparte su operación (Carlos: "que se ajuste a la operación de cada equipo, de cada cliente").

### Rastreo de guías↔referencia visible en ambos lados
Se agregará la trazabilidad de qué guías componen un DGO/referencia: (a) visible dentro del propio DGO, y (b) como pestaña/apartado en el menú de la referencia mostrando guía + factura + número de parte relacionados.

**Por qué:** Carlos Leobardo indicó que necesita ver qué guías están relacionadas a qué número de factura y qué número de parte — sobre todo por factura, porque de ahí el sistema de Taico ("Cigaki"/archivo M) dispara información al equipo. Hoy en SIGAC 2 no existe esa vista; solo se puede entrar factura por factura, lo cual es lento cuando hay hasta 20 facturas por operación. Germán confirmó ambos lados con el feedback de Carlos.

### Relación guía↔factura: cardinalidad
Una guía house puede traer de 1 a 20 facturas comerciales, cada una con hasta 20 partidas (Carlos Leobardo). El caso inverso — una factura repartida en varias guías — es raro y normalmente un error de captura (Danae), pero el sistema debe soportarlo igual (Germán).

### Documentos permitidos en un mismo "document" de digitalización
Se puede subir más de un documento del mismo tipo en una sola carga de "document" (ej. varios TLC/certificados juntos, varios packing list juntos), siempre que se declare correctamente el tipo (ej. "certificados", "packing list").

**Por qué:** Confirmado por Daniel y Cesar como práctica real del anexo 22 (documentos "llamados", ej. guía aérea = documento "guía"). Hoy la pantalla de Ángel truena al intentar subir varios juntos — queda como ajuste pendiente. Danae explicó además que ellos ya consolidan así (ej. 7-8 facturas en un solo archivo) para no generar un document por cada guía/factura.

### Guía master se guarda pero no se muestra; solo guía house es visible
En la tabla de guías, se deben guardar en BD tanto guía master como guía house, pero **mostrar solo la guía house** en la interfaz.

**Por qué:** Daniel confirmó (salvo corrección de Danae/Carlos) que la guía house es la que da la directriz operativa real para Taico.

### Guías: pestañas separadas por tipo de tráfico dentro de la misma sección
En vez de una sola tabla de guías, habrá tabs (aérea / marítima / terrestre) dentro de la misma sección "Guías", visibles según el perfil/tráfico asignado al usuario (a un ejecutivo de puro aéreo solo le aparece el tab aéreo; a uno con aéreo+terrestre, ambos; etc.).

**Por qué:** Al comparar el Excel de control de Rocío (marítimo/Manzanillo) contra la tabla de guías aéreas ya construida, los campos necesarios son sustancialmente distintos (ver Lógica de negocio). Germán y Rocío confirmaron el enfoque.

### Documentos obligatorios "document" deben basarse en anexo 22, no en criterio ad hoc
Hoy Ángel preseleccionó qué documentos son obligatorios como "document" según respuestas que le dieron en reuniones pasadas, no según una referencia formal al anexo 22. Se corrige: si el catálogo de tipos de documento viene del anexo 22, no se debe modificar libremente; si es un catálogo interno de Carmi, sí se puede ajustar.

**Por qué:** Al probar con documentos reales de Taico (carta 318, anexo, factura de fletes, póliza de seguros) aparecieron como forzados dentro de la categoría genérica "anexo" por no existir un tipo específico. Diana propuso agregar conceptos específicos al catálogo (factura, lista de empaque, BL, carta 318, etc.) para ubicarlos más fácilmente; Germán estuvo de acuerdo ("eso está padre").

### Nuevo tipo de documento: "Previo" (distinto de factura comercial)
Se creará un tipo de documento "previo" (reporte de verificación previa), hoy inexistente en el catálogo — actualmente se sube marcado como factura comercial a falta de opción.

**Por qué:** Danae/Carlos compartieron un reporte de previo etiquetado como factura por no tener mejor opción; Germán señaló que para la glosa digital es necesario poder contrastar el previo contra la factura como documentos distintos, no confundirlos.

## Lógica de negocio

### DGO (Datos Glosados para Operación)
Consolida en una sola vista: configuración por pedimento individual (aduana, clave de pedimento, patente, régimen, destino), alta de facturas (con sus propios números de parte), configuración de partidas/números de parte (clasificación, TLC, prosec), incrementables/decrementables e identificadores a nivel pedimento. Campos que serán iguales entre todos los pedimentos de una misma operación (ej. algunos identificadores) van a nivel **operación**, no a nivel DGO individual — de ahí que se pueda seleccionar 2+ DGOs y crear una sola operación con varios pedimentos.

### Glosado digital continuo (Zeus)
Cada vez que se agrega/edita un documento o dato en el expediente/DGO, corre un proceso de glosa digital que:
1. Contrasta el documento nuevo contra los ya existentes (ej. factura vs. packing list: cantidades deben cuadrar).
2. Contrasta también contra los datos que se van cargando en el DGO (ej. régimen A1 vs. número de parte incompatible).
3. Incorpora el reporte del previo/verificación al expediente para solidificar la consistencia contra lo verificado físicamente.

Cuatro niveles de glosado identificados por Germán:
1. **Integridad por documento**: consistencia interna + contraste contra ley/anexo 22.
2. **Integridad del expediente**: que no falte ningún documento requerido según perfil de cliente, número de parte, aduana o agente aduanal.
3. **Documentos vs. realidad física**: verificación previa (app móvil) contrastada contra el expediente.
4. **Expediente vs. operación generada**: todo lo que Carmi genera (COVE, pedimento, etc.) se regresa al expediente para el chequeo final.

Pendiente de agregar un **5º nivel/perfil**: "perfil de operación / tipo de operación", porque distintos tipos de operación requieren distintas fechas para medir el tiempo de facturación (métrica de turnaround) — detectado en vivo por Germán, sin resolver en la sesión.

Objetivo de negocio explícito: si todos los DGO quedan en verde, se debe poder generar pedimento y continuar la operación sin errores sorpresa — a diferencia de SIGAC 2, donde el glosado se hacía al final y los errores se descubrían tarde (a veces 2-3 días de operación después), obligando a regresar con el cliente por documentos ya vencidos en plazo.

### Evidencia de recepción de documentos (accountability)
El expediente debe registrar cuándo llegó cada versión de un documento, no solo la vigente. Caso de negocio: un cliente a veces envía primero una factura con error y días después la correcta; en SIGAC 2 el KPI de "entrega de documento" se marca cumplido desde la primera recepción (aunque estuviera mal), sin forma de demostrar que la versión correcta llegó tarde. El expediente de Carmi debe permitir esa evidencia (fecha real de recepción editable, no solo timestamp de subida al sistema).

### Marítimo (Manzanillo) — campos de guía/contenedor
A partir del Excel de control de Rocío (llevado por cada ejecutivo, ej. "Sonia"), los campos necesarios para el tab marítimo de guías son: referencia cliente, referencia SIGAC, proveedor, bultos o contenedores (uno de los dos, no ambos), tipo de contenedor (dry cargo "DC", high cube "HC", u otros — catálogo abierto), buque, recinto (ya existe catálogo de recintos de Manzanillo en el sistema), transportista, ETA (tomado del correo del cliente), estatus de captura (pendiente/capturada — si la factura tipo "A24" ya se digitalizó en sistema), observaciones libres, "envío de POD" (documento que exige el cliente GKN para recibir mercancía en planta: factura + BL + pedimento pagado + reporte), y fecha de cita.

Para el cliente GKN, la factura formal ante autoridad se llama internamente **"A24"** (equivalente al DGO/factura comercial); la factura comercial que comparte el cliente sirve solo de referencia/confirmación de mercancía, bultos y cantidades — el A24 es lo que se digitaliza y usa realmente.

### Citas en marítimo (distintas de terrestre)
Cuatro citas propias del flujo marítimo: **previo, despacho y retorno de vacío** (Veracruz — Rocío), más **PIT** para Manzanillo (Lilia) — que no es una cita per se sino la vinculación del transporte a la terminal vía AIPA (trámite y despacho), condicionando el ingreso del transporte a cualquier terminal una vez vinculado el RFC.

### Validación de duplicados al importar guías
Se valida por combinación de guía master + guía house + número de vuelo: si esos tres campos coinciden con una guía ya existente, no permite generar duplicado.

### Formato de datos de importación de manifiesto
El flujo soporta pegar el correo/manifiesto crudo del cliente (ej. reporte de DHL a las 7-8am), previsualizar, y el sistema separa automáticamente las guías nuevas vs. existentes agrupadas por cliente, antes de confirmar la importación.

## Tareas

Revisadas y dadas de alta en `Tareas.md` (algunas como nota adicional a tareas ya abiertas, ver detalle):

1. Clave de pedimento dependiente de régimen en guías — **en progreso** (ya se agregó la posibilidad de capturar la clave, falta que sea select filtrado) → sección Régimen y DGO.
2. Rastreo guías↔factura↔número de parte en DGO y pestaña de referencia — nueva → sección Referencia, DGO y Operaciones (core).
3. Subir varios documentos del mismo tipo en una sola carga — nueva → Expediente Aduanero.
4. Mostrar solo guía house en tabla (guardando ambas en BD) — nueva → Tablero de Guías.
5. Tabs de guías por tipo de tráfico con visibilidad por perfil — nueva → Tablero de Guías.
6. Ampliar catálogo de tipos de documento según anexo 22 — nueva → Expediente Aduanero.
7. Crear tipo de documento "Previo" (debe reflejarse en Expediente Aduanero) — nueva → Expediente Aduanero.
8. Mejoras UX de carga de documentos (bundle) — nueva → Expediente Aduanero.
9. Marcar versión vigente de un documento para el DGO — nueva → Expediente Aduanero.
10. Datos extraídos por Zeus visibles por documento, marcados con ícono de Zeus — nota agregada a la tarea ya abierta de UI de Expediente Aduanero/Zeus → sección Régimen y DGO.
11. Nivel de glosado "perfil de operación / tipo de operación" — nueva, **pendiente de definir con Germán** qué tipos de operación y fechas aplican → sección Referencia, DGO y Operaciones (core).
12. Campos marítimos de guía-contenedor — nota agregada a la tarea ya abierta "Definir equivalente marítimo del tablero de guías" → sección Diferidas/pendientes de análisis.
13. Sección de citas propia para marítimo (previo, despacho, retorno de vacío, PIT) — nueva → nueva sección Marítimo (Manzanillo/Veracruz).
14. Revisión con Fer el lunes 2026-07-27 sobre métodos de valoración/incoterms por proveedor (Taico) — nueva → nueva sección Seguimientos puntuales.
15. Nuevos estatus de guía aérea: Revalidado, Peso certificado, Digitalizada — nueva → Tablero de Guías.
16. Evaluar API de tracking de buques con Rocío, Mariana y Lilia — nueva → sección Marítimo (Manzanillo/Veracruz).

## Tickets
*(Ninguno mencionado en esta reunión.)*

## Temas descartados por irrelevantes
- Problemas técnicos de conexión/audio (Daniel y Danae reconectándose, "no te escucho", "comparte pantalla")
- Saludos y presentaciones de asistentes que se conectaron tarde (Danae, Cesar)
- Comentario sobre el tráfico de Culiacán y la ruta de vuelo Culiacán-Querétaro (contexto anecdótico de Ángel/Germán)
- Bromas/comentarios estéticos sobre el Excel de Sonia ("qué bonito")
- Confirmaciones triviales repetidas ("ya, ya", "sí", "okay", "ajá") sin contenido adicional
