# Documento de Entendimiento — SIGAC 3.0 (v2)
## Refactor del flujo del Ejecutivo de Cliente

**Objetivo aclarado:** SIGAC 3 ya existe y está parcialmente construido. No se diseña desde cero: se **refactoriza** para resolver los dolores de SIGAC 2 y simplificar la vida del ejecutivo, usando como base lo que ya está implementado (D1) y evaluando la propuesta de rediseño de pantallas que German presentó (M9), todavía no decidida.

> 🔴 **Corrección importante respecto a la v1 de este documento:** en la v1 trataba el contenido de M1–M8, M10 y M11 como si fueran decisiones de diseño para SIGAC 3. Eso estaba mal a medias. **M1, M2, M3, M4, M5, M6, M7 y M11** sí son sesiones de **discovery con los ejecutivos de marítimo, aéreo, terrestre y virtual sobre cómo trabajan HOY en SIGAC 2** — sirven como contexto/requisitos, no como decisiones tomadas para SIGAC 3. **M8 y M10, en cambio, se confirman como propuesta de diseño para SIGAC 3** (mismo estatus que M9: no decidida). Lo que es SIGAC 3 son: **M0** (diseño de Movimientos, implementado), **M8, M9 y M10** (propuestas de refactor, no decididas) y **D1** (estado actual real del código). Todo lo de abajo está reescrito con esa separación.

## 0. Mapa de fuentes

| Ref. | Fuente | Fecha | Categoría |
|---|---|---|---|
| **M0** | Análisis de Referencia y Movimientos | 19-mar-2026 | **Diseño SIGAC 3** — introduce Movimientos, ya implementado |
| **M0b** | Prueba operación real desde ticket a facturación (revisión de campos, continuación de M0) | 28-abr-2026 | **Diseño SIGAC 3** — decisión confirmada: relación Movimiento↔Operación |
| M1 | Daily Scrum | 25-jun-2026 | Discovery SIGAC 2 — terrestre |
| M2 | Prueba operación real desde ticket a facturación | 26-jun-2026 | Discovery SIGAC 2 — terrestre |
| M3 | Daily Scrum | 30-jun-2026 (09:24) | Discovery SIGAC 2 — terrestre |
| M4 | Prueba operación real desde ticket a facturación | 30-jun-2026 (15:58) | Discovery SIGAC 2 — terrestre |
| M5 | Marítimo SIGAC 3.0 * | 1-jul-2026 | Discovery SIGAC 2 — marítimo |
| M6 | Virtual — Folios B1 * | 1-jul-2026 | Discovery SIGAC 2 — virtual/B1 |
| M7 | Aéreo SIGAC 3.0 * | 2-jul-2026 | Discovery SIGAC 2 — aéreo |
| M8 | Seguimiento de refactor | 6-jul-2026 | **Propuesta de refactor SIGAC 3** — NO decidida |
| **M9** | Revisión de pantallas flujo operación | 7-jul-2026 | **Propuesta de refactor SIGAC 3** — NO decidida |
| M10 | Revisión solicitud de fondos | 7-jul-2026 | **Propuesta de refactor SIGAC 3** — NO decidida |
| M11 | Aéreo SIGAC 3.0 | 8-jul-2026 | Discovery SIGAC 2 — aéreo |
| **D1** | "Flujo UX/UI de una Operación" (carmi-digital) | sin fecha — código actual | **Estado actual del código de SIGAC 3** |

*Los títulos de calendario dicen "Sigac 3.0" porque así se llama el proyecto/iniciativa general en el que se enmarcan estas sesiones de discovery, no porque el contenido tratado sea sobre el sistema nuevo.

> ℹ️ **Nota sobre M0 y M0b:** M0 (19-mar) es el meet más antiguo, donde se introduce el concepto de Movimiento. M0b (28-abr) es una continuación directa de esa conversación, con el mismo núcleo de personas (Angel, German, Carlos, Enrique) más especialistas de clasificación invitados — comparte título con las sesiones de discovery ("Prueba operación real desde ticket a facturación", como M2 y M4), pero **esta instancia específica es de diseño**, no de discovery: ahí se resuelve explícitamente la relación Movimiento↔Operación (ver glosario).

> ✅ **Confirmado sobre M8 y M10:** aunque cada uno tiene solo 2 asistentes (Angel+German; Angel+Miguel), me confirmaste que su contenido (factura glosada, expediente electrónico, portal de documentos, 3 categorías de solicitud de fondos) sí es **propuesta de diseño para SIGAC 3**, con el mismo estatus que M9: presentada, pero **no decidida ni implementada**. Los trato así en todo el documento.

---

## a) Glosario (en capas: qué decía SIGAC 2 / qué existe hoy en SIGAC 3 / qué propone M9)

### Referencia
- ✅ **Definición central (aclarada por ti — reemplaza el entendimiento anterior, que se quedaba solo en el wizard):** la Referencia tiene **3 funciones**:
  1. **Expediente digital aduanero** — el repositorio documental con su checklist y validaciones. **Cada vez que se agrega un documento, se lanza un proceso de glosado digital con Zeus** que valida: (a) que el documento es válido por sí solo y cumple con la ley aduanera; (b) que es consistente con todos los demás documentos y con los DGO; (c) que es consistente con la información de la verificación previa de la mercancía (**previo**, ver glosario).
  2. **Datos Glosados para Operación (DGO)** — la **fuente única de verdad**: se construye a partir de los documentos comerciales/facturas que se juntaron desde el expediente, ya **glosados (auditados)**. Una referencia trae **por default 1 DGO**, que se separa en más si hace falta (ver detalle en "DGO" abajo — 1 DGO = 1 pedimento). Con esos datos, la Operación agrupa el/los DGO/pedimento(s), asegurando el **blindaje legal**.
  3. **Punto de partida para asociar todos los demás elementos** de una importación o exportación: Operaciones, Movimientos, Pedimentos, Timeline, avances, comentarios, etc.
- ⚠️ **Nota de estructura, importante para el inventario de pantallas:** la Referencia **no es solo un wizard de 3 pasos**. El wizard es únicamente la forma de crearla rápido. Al final debe tener tres piezas de UI distintas: su **wizard** (creación), su **detalle** (donde viven las 3 funciones de arriba) y un **listado de referencias** (índice/tabla). Esto afecta el inventario de pantallas — vale la pena revisarlo ahí también.
- **SIGAC 2 (contexto):** registro que el ejecutivo abre al recibir el disparador de la operación (BOL, manifiesto, correo, folio); datos preliminares; peso/bultos se confirman después por almacén. *(M1, M2, M7, M11)*
- **SIGAC 3 hoy (D1):** wizard de **4 pasos** — Documentos → Documentos Aceptados → Datos Básicos → Documentos Comerciales (`references/createReference/page.tsx`). El paso 1 ya sube factura/BL/AWB y usa IA (Zeus) para extraer datos y crear la factura automáticamente, incluida búsqueda automática de proveedor.
- ✅ **Decidido (M9):** el wizard baja a **3 pasos**, quitando la obligatoriedad de documentos comerciales al crear; esa sección se mueve al detalle de la referencia (dentro de la función de DGO descrita arriba). D1 todavía no lo tiene implementado (hoy sigue siendo el wizard de 4 pasos, documento-primero); esto queda como trabajo pendiente del refactor, no como decisión abierta.
- **Rol consolidado:** Referencia = responsabilidad **documental** del ejecutivo (ver Movimiento). *(M0, coherente con D1)*

### Movimiento
- ✅ **Taxonomía completa (aclarada y corregida por German):** los movimientos se dividen en dos grupos:
  - **Estándar (trazabilidad)** — aplican en general para los tres tipos de tráfico (aéreo, marítimo y terrestre):
    - **Subdivisión** — segmenta mercancía de una factura DGO/entrada original para gestionar remanentes, separaciones físicas o ajustes por régimen/destino. (Antes lo tenía como una acción sobre la Entrada; se formaliza como tercer tipo de movimiento estándar.)
    - **Previo** — etapa de inspección de la carga antes del despacho (formaliza, para terrestre, aéreo y marítimo, lo que ya teníamos como el concepto general de "previo").
  - **Especializados (tráfico terrestre)** — en terrestre las unidades de seguimiento principales son las facturas, el Bill of Lading y los tractos con caja (vehículos de carga), y el inventario de mercancía en almacén:
    - **Importación Terrestre:**
      - **Entrada** — registra el ingreso de mercancía al almacén; no requiere estar asociada a una referencia en todas las aduanas (mayor granularidad — ver excepción Laredo/**Brownsville** en M0 — corrijo: no es "Bronzeville", es Brownsville). Sirve para el registro de mercancía en el módulo de WMS de SIGAC 3 de cada almacén (Laredo/Brownsville).
      - **Salida** — registra el despacho/salida; obligatorio para la trazabilidad externa y el control físico; todo movimiento de salida debe tener un movimiento de entrada asociado. Sirve para el registro de salida de mercancía en el módulo de WMS de cada almacén (Laredo/Brownsville).
    - **Exportación Terrestre:** falta definirlo con los ejecutivos.
  - **Especializados (tráfico aéreo)** — en aéreo la unidad de seguimiento son los manifiestos de carga y las guías master y house que contiene:
    - **Importación Aéreo:**
      - **Manifiesto de Carga** — registra los manifiestos de carga recibidos por día y los conjunta en un solo tablero de planeación diaria. Comienza con la recepción por correo de los manifiestos que envía almacén y compañías como FedEx y DHL. Da seguimiento desde que avisan que van a llegar hasta que llegan a la mesa de previo (etapas: llega manifiesto → ETA del avión → confirmar llegada → descargar avión → desconsolidar carga → asignar a mesa de previo → listo para previo).
      - **Revalidación Aérea** — proceso documental y de pago ante la aerolínea.
      - **Asignación de Transporte** — asignación de línea de transporte por el ejecutivo, con seguimiento de trámite y despacho para asegurar que el transporte llega a la aduana, carga la mercancía correcta y sale de la aduana (semáforo verde/rojo).
    - **Exportación Aéreo:** falta definirlo con los ejecutivos.
  - **Especializados (tráfico marítimo)** — en marítimo la unidad de seguimiento principal son las guías marítimas y los contenedores:
    - **Importación Marítimo:**
      - **Revalidación Marítima** — proceso documental y de pago ante la naviera.
      - **Asignación de Transporte** — mismo concepto que en aéreo (semáforo verde/rojo).
      - **Retorno de Vacío** — una vez entregada la carga al cliente, hay que regresar el contenedor a la terminal para recuperar la garantía pagada y poder sacar el contenedor del puerto. Rastrea desde la asignación de transporte, recolección del contenedor vacío, pago y ejecución de fumigación/lavado del contenedor, hasta la entrega del contenedor con el acuse de recepción de vacío. ✅ **El ejecutivo actúa aquí**: solicita el transporte, da seguimiento a fumigación/lavado, solicita los fondos para el pago.
      - **Recuperación de Garantía** — proceso documental para que devuelvan la garantía pagada. Rastrea desde la entrega del vacío hasta el depósito de la garantía en la cuenta de Carmi. ✅ **El ejecutivo actúa aquí**: hace el trámite documental y solicita el depósito/devolución.
    - **Exportación Marítimo:**
      - **Toma de Vacío** — rastrea desde el trámite de solicitud de vacío hasta que un transporte pasa por el contenedor vacío. ✅ **El ejecutivo actúa aquí**: solicita el transporte para el contenedor vacío.
      - **Carga de Mercancía** — rastrea el proceso de carga: desde que llega el contenedor vacío a donde está la carga, se carga, y sale a ruta para el ingreso de la mercancía a puerto. ✅ **El ejecutivo actúa aquí**: da seguimiento y genera citas si aplica.
      - **Ingreso de Mercancía** — rastrea el ingreso físico del contenedor con carga al puerto y a la terminal (en Manzanillo incluye el trámite del PIS, la cita para poder acceder al puerto). ✅ **El ejecutivo actúa aquí**: genera la cita (ej. PIS en Manzanillo).
      - **Modulación, Liberación y Zarpe** — rastrea la modulación de la mercancía, su liberación, carga en el buque y zarpe del buque.
  - ⚠️ **Pendiente:** Exportación Terrestre y Exportación Aéreo quedaron sin definir — falta esa sesión con los ejecutivos correspondientes.
- ✅ **Regla de clasificación (arquitectónica):** "Movimiento" se reserva estrictamente para procesos que requieren seguimiento detallado en el tiempo (tracking): entradas, salidas, subdivisiones, estados de contenedores. "Operación" es para tareas puramente administrativas (generación de pedimentos, modulación) — no se deben clasificar como movimientos, para evitar redundancias. Esto también aclara dónde caen OS&D y Transferencia (ver más abajo): son acciones/reportes sobre un movimiento existente, no tipos de movimiento en sí. ✅ **Aclarado sobre OS&D:** el ejecutivo **no reporta** el OS&D — ese reporte lo genera almacén durante el **Previo**. El ejecutivo solo **consulta** la discrepancia (tipo, cantidad, evidencia, a qué movimiento afecta) para decidir qué acción tomar del lado documental/DGO.
- ✅ **Rol "trámite y despacho" confirmado fuera de alcance:** aparece en Asignación de Transporte (seguimiento) y en Previo (ejecuta la verificación en marítimo/aéreo cuando el ejecutivo la solicita). Es un rol aparte, igual que almacén/clasificación/tesorería — aplica principalmente en marítimo y aéreo. El ejecutivo asigna/solicita, pero el rol de trámite y despacho es quien ejecuta/da seguimiento; para el ejecutivo, eso queda como solo lectura.
- ✅ **Relación con DGO — flexible, no 1 a 1 (definitivo, ver "DGO" abajo):** Movimiento vive en el plano físico-logístico (almacén); DGO vive en el plano documental-legal. Se vinculan, pero sin regla fija: un DGO puede cubrir varios movimientos (la entrada original + N subdivisiones), o se puede vincular un DGO por movimiento individual — lo que convenga al caso. Ejemplo: una factura con partidas de 2 claves de pedimento distintas puede llegar en 1 solo movimiento de entrada y, tras subdividirse en almacén, terminar vinculada a 2 DGO distintos (uno por clave).
- **Concepto nuevo en SIGAC 3** (no existía en SIGAC 2, según tu aclaración). Diseñado en **M0**: **la entrada notifica a almacén qué llega y cuándo** ("el movimiento de entrada es nada más para decirle al almacén que le va a llegar y cuándo", Angel, ~00:21:23); regla base "salida requiere entrada, entrada requiere referencia" configurable por aduana/cliente; Referencia = documental, Movimiento = físico/almacén.
- ✅ **Relación Movimiento↔Operación — confirmada en M0b (28-abr-2026):** encontré la reunión exacta que buscabas. Es una continuación de M0, mismo grupo (Angel, German, Carlos, Enrique). Lo interesante: **al momento de esta reunión, el flujo estaba al revés** de lo que terminaron decidiendo:
  - **Estado al arrancar la discusión (~01:52:31):** Angel describe que, en ese momento, "generamos el movimiento de salida y a partir de ese movimiento creamos la operación" — o sea, Salida → Operación.
  - **Debate (~01:58–02:02):** German cuestiona el orden lógico y propone invertirlo: "Pero podríamos asociar la generación de una operación a un movimiento de entrada o un movimiento de subdivisión. En vez de que tengas que jalar información de un movimiento de salida, que jales información de un movimiento de entrada" (~01:58:16). Y remata: "en realidad el movimiento de salida es solamente decirle a almacén, va a salir esto, y todos esos datos los jalas del movimiento de entrada o el movimiento de subdivisión y queda automatizado" (~01:59:45).
  - **Decisión final (~02:01:38–02:05:58):** Angel: "con la idea que tú tienes sería no crear la salida a partir de las entradas, sino **crear la operación a partir de las entradas**... y cuando yo tenga la operación desde aquí yo podría decir '**solicitar salida**' y ahí generar el movimiento... un minimodal con esos campos bien poquitos" (los datos de transporte se jalan automáticamente; solo se capturan sello, contenedor y tipo de contenedor). German confirma: "Eso queda más natural. ¿Estás de acuerdo?" Angel: "Sí." Enrique valida la simplificación (~02:03–02:05): "me parece todo lo que sea utilizar lo que ya tenemos, quitar pasos, está bien."
  - **Conclusión:** exactamente lo que recordabas — **la Operación se crea a partir de Movimientos de Entrada (o Subdivisión)**, y el **Movimiento de Salida se genera después, desde dentro de la Operación**, como aviso a almacén de que algo va a salir, con la mayoría de sus datos heredados automáticamente.
  - ✅ **Resuelto (ver "DGO" abajo):** la tensión entre "Entrada/Salida específicos de terrestre" y la regla de M0b se resuelve porque **el Step 0 del wizard de Operación pasa a ser "Seleccionar DGO(s)" directamente**, no movimientos — ya no hace falta un equivalente de "Entrada" por tráfico para poder crear la Operación.
  - **D1 (código actual) confirma que esta decisión se implementó así:** el wizard de Crear Operación arranca en el Step 0 "Seleccionar Movimientos" (selecciona INBOUND/Subdivisión), y `ShipmentCreationModal` en su flujo OUTBOUND (salida) pide datos de operación/pedimento — coincide exactamente con el diseño acordado en M0b.
  - Dato adicional de contexto (SIGAC 2, mencionado por Angel dentro de esta misma sesión, ~02:00:42): "en Sigac 2 generan la operación y ya que quieren mandan un evento de orden de carga a almacén y almacén hace la orden de carga" — o sea, SIGAC 2 ya generaba la operación antes que la salida física, aunque con un mecanismo distinto (evento de "orden de carga" en vez de un movimiento de salida formal).
- **SIGAC 3 hoy (D1): ya implementado y coincide con el diseño de M0.** Tab "Movimientos" en el detalle de referencia con sub-tabs **Entradas (INBOUND)** y **Subdivisiones**; balances vía `/references/{id}/shipments/available-for-operation`. Componentes reales:
  - `ShipmentCreationModal` — crea entrada o salida (tabs: Facturas/BL → Transporte → Guías → Bultos → Contenedores; OUTBOUND añade operación/pedimento).
  - `SubdivisionCreationModal`/`SubdivisionForm` — divide inventario de un movimiento padre; **regla: no se puede subdividir el 100%**, algo debe quedar en el origen.
  - `OSDReportForm` — Over/Short/Damaged (Sobrante/Faltante/Daño), con responsable CARRIER/WAREHOUSE/SUPPLIER/CUSTOMER.
  - `TransferItemsModal` — transfiere ítems entre movimientos de la misma referencia.
  - Fuente: D1, §3–4.
- Esto **confirma y enriquece** la definición de v1; la relación Movimiento↔Operación ya no es un punto abierto — quedó resuelta con M0b y coincide con lo implementado en D1.

### Expediente aduanero / "Documentos"
- **SIGAC 2 (contexto):** "expediente de comercio" se genera al pagar el pedimento e integra todo lo digitalizado. *(M3)*
- **SIGAC 3 hoy (D1):** tab **"Documentos"** (`ReferenceDocuments.tsx`) — grid de cards con badge de categoría (Facturas/Packing List/BL/Otros), badge de estado (Pendiente/Subido/Verificado/Rechazado), **toggle E-Document VUCEM por documento**, badge rojo si hay error VUCEM. No existe todavía checklist obligatorio ni estado de glosa verde/rojo.
- **Propuesta M9 (no decidida):** renombrar a "Expediente aduanero", con checklist obligatorio por perfil cliente+operación+número de parte, estado de glosa (verde/rojo), historial a nivel expediente con documento "primario". **Nada de esto está en D1 todavía** — es una brecha completa entre lo propuesto y lo construido.
- ✅ **Decidido (reunión con German, ver Punto Abierto #5 resuelto):** cada vez que se sube un documento al expediente, **se deben glosar todos de nuevo** — re-glosado completo del conjunto, no solo el documento nuevo contra los demás. El motor es **Zeus**, y valida 3 cosas: (a) el documento es válido por sí solo ante la ley aduanera; (b) es consistente con el resto de documentos y los DGO; (c) es consistente con el **previo** (verificación física de la mercancía).
- ✅ **Confirmado — recibe documentos del Portal de Documentos del Cliente:** los documentos que el cliente sube directamente (ver sección d) caen aquí, marcados como "subido por el cliente". El resultado del glosado se refleja tanto en este Expediente (para el ejecutivo) como en el portal (feedback al cliente).

### Datos Glosados por Operación (DGO) / "Glosa digital"
- ✅ **Encaje conceptual (ver "Referencia" arriba):** el DGO es una de las **3 funciones de la Referencia**, no un concepto aislado — se construye a partir de los documentos comerciales/facturas del Expediente (función 1), una vez glosados, y con eso se genera la Operación asegurando el blindaje legal.
- **SIGAC 2 (contexto):** no existía.
- **SIGAC 3 hoy (D1): no existe un tab "Datos Glosados" ni el concepto DGO.** Lo más cercano son dos cosas separadas y ya construidas:
  1. **Tab "Mercancía"** dentro de la referencia (facturas + partidas, edición inline, verificación de totales) — es donde vive la captura hoy.
  2. El paso **GLOSA** dentro del despacho de 8 pasos (`StepPedimento.tsx`): compara **Capturado vs Pedimento** campo por campo, con botones "Regenerar/Actualizar Glosa" — pero corre **después** de generar el pedimento, no es una fuente de verdad desde el arranque de la referencia como propone M9.
- ✅ **Confirmado — DGO absorbe 3 tabs que hoy son de la referencia:** "Mercancía" (facturas/partidas), "Costos Globales" e "Identificadores" (Configuración Legal) **dejan de ser tabs propios de la referencia** — su contenido pasa a vivir **por DGO**. Se pueden seleccionar uno o varios DGOs para asignarles los mismos identificadores/incrementables-decrementables, o dejarlos distintos por DGO (y para incrementables, prorratear un monto entre varios DGOs o asignar cantidades distintas a cada uno).
- ✅ **Decidido:** el DGO **reemplaza** el step GLOSA actual del despacho — no conviven. "Vamos a dejar solo DGO" — fuente única de verdad a nivel partida, comparación JSON vs JSON contra la factura real, firma obligatoria antes de avanzar. Falta definir cómo se retira/absorbe `StepPedimento.tsx` (el step GLOSA de D1) dentro del stepper de despacho de 8 pasos, y si el paso ahí se elimina o solo cambia de fuente de datos.
- ✅ **Estructura interna de un DGO (aclarada por ti, con esquema):** una referencia trae **por default 1 DGO**. Jerarquía de cada DGO:
  - **Datos Globales** — por defecto se heredan de la referencia, pero cada DGO puede tener los suyos propios (identificadores globales e incrementables/decrementables iguales o distintos a otro DGO de la misma referencia). Un gasto global se puede prorratear entre varios DGOs, o asignarse completo a uno solo.
  - **Factura(s)** — una o más por DGO.
    - **Partida(s)** — una o más por factura.
- ✅ **Regla clave (definitiva): 1 DGO = 1 pedimento.** No son pasos distintos ni uno "genera" al otro — **son la misma unidad vista desde dos lados**. El DGO ya trae los campos de pedimento (Régimen Aduanero — IMD, ITR, Apéndice 16 del Anexo 22 —, Clave de Pedimento, aduana de despacho, destino, observaciones, identificadores globales, incrementables/decrementables); no hay una "generación" posterior de esos datos, es la misma captura. Si la referencia solo necesita un pedimento, se queda con 1 DGO. Si necesita más de uno (ej. dos claves de pedimento distintas), el DGO **se separa** en tantos DGO como pedimentos hagan falta.
- ✅ **Ejemplo completo trabajado (el caso que aclara todo):**
  1. Llega una factura con 10 partidas: 5 con clave A1 y 5 con clave B2. Todo entra en **un único movimiento de entrada**.
  2. Como hay dos claves distintas, se le pide a almacén que **subdivida** ese movimiento: queda el original con las 5 partidas A1, y un movimiento de subdivisión nuevo con las 5 partidas B2. Esto es **puramente físico/logístico**.
  3. Del lado documental, la referencia arranca con 1 DGO (la factura completa). Como se necesitan dos claves, ese DGO **se separa en 2 DGO** — uno para las partidas A1, otro para las B2.
  4. Cada DGO **ya es, directamente, su propio pedimento** (DGO clave A1 = pedimento A1; DGO clave B2 = pedimento B2). No hay un paso adicional de "generar" el pedimento.
  5. Ambos DGO/pedimentos quedan agrupados bajo **una sola Operación**, que es el contenedor que los administra juntos.
  6. El vínculo entre estos DGO y los movimientos de almacén (el original y el de subdivisión) es **flexible**: aquí mapea 1 a 1 por conveniencia, pero el sistema no lo obliga — podría haber un DGO cubriendo varios movimientos, o viceversa.
- ✅ **Operación (definitivo):** agrupa uno o más DGO/pedimentos — **no los crea ni los transforma**, solo los consolida bajo un mismo expediente para trabajarlos juntos. Una operación puede llegar a agrupar **varias referencias**.
- ✅ **Movimientos, plano aparte:** la gestión física en almacén (qué llegó, cuándo, cómo se reparte físicamente la mercancía) vive en un plano distinto al documental-legal. Se vincula al DGO de forma flexible, **no 1 a 1 fija** — un solo DGO puede cubrir varios movimientos (la entrada original + N subdivisiones), o se puede vincular un DGO por movimiento individual, según convenga.
- ✅ **Resuelta la tensión con M0b — actualizado:** M0b estableció que "la Operación se crea a partir de Movimientos de Entrada/Subdivisión". Esto se refinó después: **el Step 0 del wizard pasa a ser "Seleccionar DGO(s)" directamente**, no movimientos — la Operación se arma eligiendo DGOs. El vínculo con los movimientos (de dónde salió cada DGO) queda como dato de trazabilidad dentro del DGO/tab Movimientos, no como mecanismo para crear la Operación. Con esto, la pregunta de "¿qué es el equivalente de Entrada en aéreo/marítimo para poder crear la Operación?" ya no aplica — no hace falta un equivalente, porque no se parte de movimientos.
- **Idea central a retener:** Referencia y DGO/pedimento = plano **documental-legal**. Movimientos = plano **físico-logístico** (almacén). Operación = nivel que **agrupa** los DGO/pedimentos. Los tres se relacionan, pero cada uno tiene su propia responsabilidad — no se deben mezclar como si fueran el mismo concepto.

### Operación
- ✅ **Definitivo (ver "DGO" arriba):** la Operación **agrupa** uno o más DGO/pedimentos ya existentes — no los crea ni transforma. El número de DGOs agrupados determina el número de pedimentos. **El Step 0 del wizard pasa a ser "Seleccionar DGO(s)" directamente**, no "Seleccionar Movimientos" — la Operación se arma eligiendo DGOs. Como el DGO ya trae los campos de pedimento, los Step 3/4 de abajo (Config Fiscal, Encabezado del Pedimento) llegarían prellenados.
- **SIGAC 2 (contexto):** se genera con importador, fechas, transporte, patente, aduana; puede generarse antes de terminar clasificación pero sin poder calcular impuestos hasta completarla. *(M1)*
- **SIGAC 3 hoy (D1): wizard de 6 pasos ya construido** (`CreateOperationModal.tsx`), se abre desde selección múltiple de referencias, desde el detalle de referencia, o desde el tab Movimientos:
  - **Step 0** — Seleccionar Movimientos (cards con checkbox, disponibilidad). **⚠️ Este paso cambia a "Seleccionar DGO(s)" con el modelo definitivo.**
  - **Step 1** — Origen/Agrupar Pedimentos (un embarque, o multi-embarque con selector de agrupación: **Un pedimento / Por factura / Personalizado** con drag&drop entre grupos).
  - **Step 2** — Configuración de Operación (Tipo de Cambio: TC Banxico del día vs manual; fechas; Tipo de Operación, Agente Aduanal, Empresa que Factura, Mandatario).
  - **Step 3** — Config Fiscal por pedimento (Identificadores Globales, **Gastos Globales** con prorrateo sugerido, bloqueo si la clave no permite Valor Agregado y hay VA capturado).
  - **Step 4** — Encabezado del Pedimento (Régimen, Clave de Pedimento, Aduana de Despacho, alertas de VA por clave T2/T3, **cálculo de impuestos en tiempo real** con debounce de 500ms).
  - **Step 5** — Formas de Pago (solo lectura, "Calculado por Sistema": IGI/DTA/IVA/PRV/CNT vía TaxEngine).
  - Multi-pedimento: cada grupo repite los pasos 3→5; al final `POST /operations` genera la Operación y su(s) pedimento(s).

### Pedimento
- ✅ **Nuevo encaje (ver "DGO" arriba):** el número de pedimentos que tendrá una operación se determina por cuántos DGOs se seleccionen al crearla (1 DGO = 1 pedimento). Los campos de encabezado del pedimento (régimen, clave, aduana, destino, observaciones, identificadores, incrementables/decrementables) se anticipan desde el DGO correspondiente, en vez de capturarse solo hasta crear la Operación.
- Generado desde la Operación (patente + aduana). D1 confirma: **soporta varios pedimentos por operación** (consolidados, vía agrupación en Step 1). Los "Gastos Globales" de Step 3 son lo que M9 propone renombrar a "incrementables y decrementables" — **hoy en D1 se siguen llamando Gastos Globales**, la propuesta de renombrar no está aplicada.
- **Reglas de Valor Agregado (nuevo, de D1):** claves T3/RF-6 código A1 **prohíben** VA; claves T2/RF-5 códigos RT/RTX **requieren** VA o leyenda. Prorrateo del VA por **método Hamilton (mayor residuo en centavos)**.

### Despacho — flujo final (antes "happy path")
- **SIGAC 2 (contexto, M1):** documentos → COVE → glosa → manifestación de valor → transmisión → validación → pago → DODA → chiper (si >2,500 USD).
- **SIGAC 3 hoy (D1): ya implementado como stepper de 8 pasos** orquestado por **Temporal** en `/customs-operation/{id}` (única entrada real: tab Operaciones de la referencia → "Inicio de Operación"):
  **EDOCUMENTS → COVE → MANIFESTACION → GLOSA → VALIDACION → PAGO → DODA → MODULACION**
  - 🔄 **Nota de orden (reformulada, ver Brecha #2):** en M1 (SIGAC 2) la glosa se mencionó *antes* de la manifestación de valor; en D1 (SIGAC 3 actual) el orden es MANIFESTACION antes de GLOSA. Con la decisión de dejar solo DGO, este paso GLOSA desaparece del stepper, así que la pregunta original ya no aplica — se convirtió en una tarea más amplia de validar los 8 pasos completos del despacho contra los meets de discovery, no solo el orden de dos pasos.
  - **MODULACION** (semáforo verde/rojo) es un paso nuevo que no aparece explícitamente en los meets de SIGAC 2 revisados (ahí se hablaba de "liberación"). Podría ser el equivalente, a confirmar.
  - No aparece un paso explícito de **Shipper/Chiper** (>2,500 USD) en los 8 pasos. Ver Brecha #3.
  - Botón **"Iniciar Operación"** → `POST /operations/{id}/dispatch/start` `{mode:'AUTO'}`. Solo cancelar (todo) o reintentar/forzar por paso — **no existe "pausar"**.
  - ✅ **Decidido (ver Brecha #1 resuelta):** el paso **GLOSA** de este stepper se reemplaza por el **DGO** — deja de convivir con él. Falta diseñar cómo queda representado este paso en el stepper una vez que el DGO asume esa función (¿desaparece, o se muestra en solo lectura?).

### CAAREM (antes referido como "Karem" en los meets)
- En M1 y M4 se menciona un "prevalidador... Karem" al que se envían los archivos antes del pago. D1 usa el nombre **CAAREM** (Confederación de Asociaciones de Agentes Aduanales) en el paso VALIDACION: botón "Enviar a validación CAAREM" → `POST /operations/{id}/dispatch/validacion/caarem`.
- **Hipótesis con alta probabilidad:** "Karem" es un error de transcripción automática de Gemini por "CAAREM" (suenan parecido). Doy por resuelto este término salvo que me digas lo contrario.

### VUCEM (antes referido como "Busem" en los meets)
- En M2 y M4 aparece "Busem" como el sistema ante el que se generan e-documents. D1 no usa "Busem" en ningún lado; usa **VUCEM** consistentemente (toggle "E-Document VUCEM", "Error VUCEM").
- **Hipótesis con alta probabilidad:** "Busem" es un error de transcripción de "VUCEM" (B/V se pronuncian igual en español, "usem"≈"ucem"). Doy por resuelto este término salvo que me digas lo contrario.

### Solicitud de Fondos
- **SIGAC 3 hoy (D1): implementación mínima.** Un botón "Solicitar Fondos" en el tab Operaciones de la referencia → `POST /operations/{id}/solicitar-fondos` (**POST vacío**, solo usa el TC de la operación) con feedback vía `alert()` nativo (deuda UX señalada en el propio D1).
- **Propuesta M10 (no decidida, mismo estatus que M9):** 3 categorías — gastos comprobados, impuestos de pedimento, honorarios — que deberían integrarse en una sola solicitud coherente, con cálculo automático de honorarios/impuestos. **No implementada** — ver Brecha #4.

### Modalidades TER / MAR / AER
- Confirmado por D1: campo real **"Tipo de Tráfico" con opciones MAR/AER/TER** en el Paso 3 (Datos Básicos) del wizard de referencia. Separado de **"Tipo de Operación"** (IMP/EXP/TRA — import/export/tránsito, Campo 2 Anexo 22), que es otro eje de clasificación distinto.
- **B1 (virtual) no aparece en la lista de "Tipo de Tráfico" de D1.** No queda claro cómo se marca hoy una operación virtual en el modelo de datos actual — **nuevo punto abierto**, ver h).
- ✅ **Aclarado — Modalidad "anticipada":** no es una modalidad ni un valor de campo. Le llaman "anticipado" a ir llenando los pasos/campos poco a poco con los datos que van llegando, para ir adelantando trabajo antes de tener todo completo (coincide con lo visto en M0b: "operación anticipada" como descripción de comportamiento, no como campo del sistema).

### Previo (marítimo / aéreo)
- ✅ **Aclarado:** en marítimo y aéreo le llaman "previo" a la **revisión de la mercancía cuando llega el barco o el avión**. Con esto se entiende mejor lo visto en M11 (el módulo de "reconocimiento previo" y los dos tipos de previo planeados en app móvil) — es la verificación física de arribo, no un concepto documental.
- ✅ **El ejecutivo puede solicitar el previo, aunque no lo ejecute:** la solicitud se dirige a **almacén** (terrestre) o a **trámite y despacho** (marítimo/aéreo) — quien sí lo ejecuta físicamente. Es una acción real del ejecutivo, no solo un dato de consulta.
- ✅ **Formalizado como tipo de Movimiento en marítimo:** dentro de la taxonomía de Movimiento (ver esa entrada), "Previo" es uno de los movimientos especializados de marítimo — la etapa de inspección de la carga antes del despacho. Falta confirmar si aéreo lo maneja igual o de otra forma.
- ✅ **Nueva conexión (aunque "previo" en sí es actividad de almacén, fuera de alcance):** su resultado **sí entra al alcance del ejecutivo** — el proceso de glosado digital del Expediente (con Zeus) valida que cada documento sea consistente contra los datos del previo. El ejecutivo necesita, entonces, visibilidad de ese resultado dentro del Expediente/DGO, aunque no gestione el previo mismo.

### MOMP
- ✅ **Aclarado:** MOMP es el **apartado de configuración de reglas de negocio de un cliente** — de ahí que determine, por ejemplo, si un cliente exige proforma y cartas firmadas (M1, M2) o cualquier otro requisito particular por cliente.
- ✅ **Vive en el perfil de cliente, se consume en el flujo del ejecutivo:** la configuración de MOMP en sí no es parte del flujo operativo del ejecutivo (es onboarding/perfil de cliente), pero sus reglas **sí se consumen** (solo lectura) dentro del flujo — sobre todo en el **Expediente aduanero**, cuyo checklist depende del perfil de cliente.

### Identificadores de operación virtual (MS / IM / B1 / IC)
- ✅ **Aclarado — encontré la transcripción completa de M6** (venía un segundo archivo del mismo meet que no había leído, con más detalle que las notas resumidas). Ahí Angel y Maria Maythe repasan, campo por campo, los identificadores de una operación virtual (~00:38–00:41):
  - **MS** = **Modalidad de Servicios**. Complemento siempre **1**.
  - **IM** = lleva el número de **IMMEX del propio cliente** (importador o exportador, según el rol en esa operación). Complemento **1**.
  - **B1** = identifica a la **contraparte** (proveedor o comprador): complemento **1** = número de IMMEX de la contraparte; complemento **2** = literal **"IM"**, para indicar que ese complemento es un número de IMMEX.
  - **IC** = identificador de **empresa certificada OEA**; solo aparece si el cliente está certificado. Complemento = letra **"O"** (no el número cero).
  - Angel propone automatizar MS/IM/B1 a nivel de la relación cliente-proveedor/cliente-comprador (para que se autocompleten por default al abrir una virtual), e IC a nivel cliente (según si está certificado OEA) — coincide con lo ya anotado como propuesta en la sección c).

### Nomenclatura OSND vs OS&D/OSD
- Los meets de SIGAC 2 (M2) usan la sigla **OSND** (Over, Short, Damaged). D1 usa **OS&D / OSD** para el mismo concepto (`OSDReportForm.tsx`). Es la misma idea con dos siglas distintas — unificar nomenclatura en el refactor.

### Términos técnicos nuevos, solo en D1 (vocabulario de la implementación actual)
| Término | Qué es |
|---|---|
| **Zeus** | Motor de IA que extrae datos de documentos subidos (factura, BL/AWB, packing list) vía `POST /api/workflow/v3/process`. **También corre el proceso de glosado digital** cada vez que se sube un documento al expediente (validez legal, consistencia contra otros documentos/DGO, consistencia contra el previo). |
| **Signed URL (GCP)** | URL firmada de Google Cloud Storage para previsualizar/descargar documentos; expira ~10 min. |
| **StepperModal** | Componente de armazón reutilizado en todos los wizards (header con pills de paso, barra de progreso, footer con Anterior/Cancelar/Siguiente). |
| **Bifrost** | Servicio al que se envían los e-documents para su transmisión (`POST /operations/{id}/edocuments/send`). |
| **Temporal** | Motor de orquestación de workflows que corre el despacho automático en backend. |
| **TaxEngine** | Motor que calcula impuestos (IGI/DTA/IVA/PRV/CNT) y determina forma de pago. |
| **Control Tower** | Pantalla del dashboard `/operations/` (ruta de frontend, no confundir con los endpoints de API `/operations/{id}/...` que sí son de SIGAC 3) — **es de SIGAC 2**, usa datos mock y tiene botones sin handler. **Fuera de alcance del refactor**; la única área real de SIGAC 3 a considerar es `/customs-operation/{id}`. |

---

## b) Modelo mental — dos vistas

### b.1 Flujo actual real (D1 — lo que existe hoy en SIGAC 3)
```
Referencia (wizard 4 pasos: Documentos → Doc. Aceptados → Datos Básicos → Doc. Comerciales)
   │
   ├─ Tab Mercancía (facturas/partidas, edición inline)
   ├─ Tab Movimientos (Entradas/Subdivisiones → ShipmentCreationModal, SubdivisionModal, OSD, Transfer)
   ├─ Tab Documentos (grid, badges de categoría/estado, toggle VUCEM)
   ├─ Tab Costos Globales / Config Legal
   │
   └─ Tab Operaciones → "Crear Operación" (wizard 6 pasos, desde Movimientos seleccionados)
             → genera Operación + Pedimento(s)
             │
             └─ "Inicio de Operación" → /customs-operation/{id}
                  Stepper 8 pasos (Temporal): EDOCUMENTS → COVE → MANIFESTACION → GLOSA →
                  VALIDACION → PAGO → DODA → MODULACION
```

### b.2 Dirección propuesta por M9 (no decidida — a validar contra D1)
```
Referencia (3 pasos, solo datos básicos)
   │
   detalle de referencia:
   ├─ Expediente aduanero (checklist obligatorio + estado de glosa)
   ├─ Datos Glosados por Operación — DGO (fuente única de verdad, nivel partida)
   ├─ Movimiento (sin cambios respecto a D1 — M9 no lo menciona explícitamente)
   │
   └─ Operación (usa el DGO consolidado para generar COVE, MV, pedimento)
```
**M9 no dice nada sobre el wizard de Operación de 6 pasos ni sobre el stepper de despacho de 8 pasos** — esas partes de D1 quedarían sin tocar salvo que decidamos lo contrario.

---

## c) Aprendizajes de SIGAC 2 a preservar o resolver (contexto/requisitos, de M1-M7 y M11)

Reglas de negocio y dolores que vienen de cómo trabajan hoy los ejecutivos en SIGAC 2, y que el refactor de SIGAC 3 debería preservar (si son reglas legales/de negocio) o resolver (si son dolores):

- Checklist de documentos no es fijo: depende de perfil de cliente (MOMP) + número de parte + aduana + tratados. *(M1)*
- Revisión física de almacén bloquea validación de pedimento; se pidió permitir excepción bajo autorización gerencial/cliente. *(M1, M2)*
- Documentos que aplican VUCEM se transmiten a ventanilla; los que no aplican quedan solo en sistema. *(M3)* — **esto ya está en D1** (toggle E-Document VUCEM).
- Un COVE por cada factura comercial; una MV por cada pedimento. *(M3, M4)*
- Shipper obligatorio si valor > 2,500 USD, hoy generado manualmente con acceso remoto a una PC en Texas. *(M1, M4)* — **no veo dónde vive esto en D1**, ver Brecha #3.
- Tres esquemas de pago: Cuenta de Orden, Financiado, Anticipo. *(M4)*
- Documentación mínima marítima: factura, BL revalidado, certificado, aviso automático. *(M5)*
- Rectificaciones requieren duplicar factura para generar COVE sin referencia. *(M5)*
- Aéreo: identificación por longitud de guía (12 FedEx / 11 UPS / 10 DHL); distinción Master/House; certificación de peso llega después, de la báscula del almacén. *(M7, M11)*
- NOM obligatoria solo en regímenes definitivos, exenta en temporales; Regla Octava para temporales con permiso SE. *(M7)*
- Virtual/B1: folios mensuales, cierre en 10 días hábiles (20 si OEA), identificadores MS/IM/B1/IC, descargos (pedimento contraparte pagado antes de exportar). *(M6)*

---

## d) Propuesta de refactor de SIGAC 3 — M8, M9, M10 (NO decidida — pendiente de validar contra D1)

Recordatorio: esto es lo que German (M8, M9) y Miguel/Angel (M10) propusieron mostrar, en el caso de M9 con ayuda de un plan armado con una IA. La mayoría sigue sin decidirse ni implementarse; el wizard de 3 pasos ya se confirmó como decisión (ver columna Estatus), aunque todavía falta implementarlo. Antes de diseñar pantallas hay que resolver, punto por punto, el resto de la tabla a la luz de lo que ya existe en D1.

| Propuesta | Fuente | Estado en D1 | Estatus | Decisión / pendiente |
|---|---|---|---|---|
| Wizard de referencia de 3 pasos, sin forzar documentos comerciales | M9 | D1 tiene 4 pasos, documento-primero | ✅ **Decidido** | Se hace así, como lo propuso German — falta implementarlo |
| Tab "Expediente aduanero" con checklist + estado de glosa | M9 | D1 tiene tab "Documentos" sin checklist ni estado de glosa | Pendiente | ¿Se extiende el tab Documentos actual o se construye uno nuevo? |
| "Factura glosada" / DGO como fuente única de verdad a nivel partida, con comparación JSON vs JSON | M8 → M9 | D1 tiene tab "Mercancía" + step GLOSA (post-pedimento) | ✅ **Decidido** | El DGO reemplaza el step GLOSA actual (no conviven) — falta definir cómo se retira/absorbe `StepPedimento.tsx` del stepper de despacho |
| Portal de documentos con URL pública para carga del cliente, expediente electrónico con historial de versiones | M8 | No existe en D1 | ✅ **Decidido** | Confirmado con mecánica concreta: los documentos que sube el cliente caen **directo al Expediente Aduanero**, marcados como "subido por el cliente"; el resultado del glosado (Zeus) se refleja tanto en el Expediente (ejecutivo) como en el portal (feedback al cliente) |
| Renombrar "costos globales" → "incrementables y decrementables" | M9 | D1 los llama "Gastos Globales" (Step 3 del wizard de Operación) | Pendiente | Simple renombre si se acepta — bajo riesgo |
| Renombrar "configuración legal" → "identificadores" | M9 | No verificado en D1 con ese nombre | Pendiente | Confirmar nombre actual del tab antes de renombrar |
| Menú lateral vs. tabs para navegación | M9 | D1 usa tabs (primarios + dropdown de secundarios) | Pendiente | Sin decisión — evaluar con el volumen real de tabs de D1 |
| Happy Path primero, casos complejos después | M9 | D1 ya soporta consolidados/multi-pedimento con agrupación | Pendiente | La propuesta de "empezar simple" ya está superada por lo construido — ajustar expectativa |
| Solicitud de fondos con 3 categorías (gastos comprobados, impuestos, honorarios) integradas y calculadas automáticamente, con validación de tesorería contra banco antes de autorizar el pago | M10 | D1 solo tiene un botón que dispara un `POST` vacío + `alert()` | Pendiente | Es prácticamente construir la lógica desde cero — ver Brecha #4 |
| Notificación al cliente vía URL pública para subir comprobante de depósito (solicitud de fondos) | M10 | No existe en D1 | Pendiente | Propuesta de Angel dentro de M10, sin validar con el resto del equipo |

---

## e) Brechas y preguntas nuevas (surgidas al comparar D1 con M9 y con los meets de SIGAC 2)

1. ✅ **Resuelto — DGO vs. step GLOSA actual:** decidiste dejar **solo DGO**. El DGO reemplaza el step GLOSA actual del despacho (`StepPedimento.tsx`, comparación Capturado vs Pedimento, regenerar/actualizar) en vez de convivir con él. Queda pendiente de diseño **cómo** se retira ese paso del stepper de 8 pasos: ¿el paso GLOSA desaparece del despacho porque ya se resolvió antes (en el DGO, a nivel referencia), o se queda como un paso de solo lectura que muestra el DGO ya validado?
2. 🔄 **Reformulado — validar el stepper de despacho completo (ya no es solo "orden GLOSA vs. MANIFESTACION"):** la pregunta original quedó obsoleta porque, al dejar solo DGO, **ya no habrá step GLOSA** como tal en el stepper de 8 pasos. Esto abre una tarea más amplia: revisar los 8 pasos del despacho actual (EDOCUMENTS → COVE → MANIFESTACION → GLOSA → VALIDACION → PAGO → DODA → MODULACION) contra lo discutido en los meets de discovery (cómo trabajan hoy los ejecutivos) para decidir si el stepper necesita cambiar de orden, fusionar pasos, o quitar/agregar alguno como parte de este refactor — no solo mover GLOSA. Pendiente de hacer ese ejercicio de validación completo antes de tocar el diseño de esta pantalla.
3. **Shipper/Chiper (>2,500 USD):** no aparece en los 8 pasos de despacho de D1. Hoy en SIGAC 2 se genera manualmente con acceso remoto a una PC en Texas (M1, M4) — falta validar dónde debería vivir dentro del refactor de SIGAC 3: ¿un paso propio del stepper, una acción dentro de DODA, o algo aparte?
4. **Solicitud de Fondos:** brecha grande entre lo conceptual (M10, platicado con Miguel: 3 categorías — gastos comprobados, impuestos, honorarios — con cálculo automático) y lo implementado (D1: un botón que dispara un POST vacío + `alert()`). La ubicación (dentro de Operación) ya coincide con lo propuesto en M10, pero falta **planear en el refactor cómo funcionaría esta pantalla en base a lo platicado con Miguel**, mejorar el flujo completo (incluyendo la validación de tesorería contra banco antes de autorizar el pago) y asegurar que quede bien implementado — no es solo mover la ubicación, es construir la lógica.
5. **Marca de operación virtual (B1):** D1 muestra "Tipo de Tráfico" con MAR/AER/TER pero no B1. Falta **analizar el esquema de base de datos actual** para ver cómo (o si) se identifica hoy una operación virtual, y a partir de ahí decidir cómo implementarlo — no es solo agregar un valor a un enum si el modelo de datos no lo contempla.
6. **Deuda técnica que el refactor debería considerar (autoreportada en D1 §8):**
   - ✅ **Aclarado:** `/operations/` (dashboard/Control Tower) es **de SIGAC 2**, no una segunda área de SIGAC 3 en competencia. Para el refactor, la única área a tomar en cuenta es `/customs-operation/{id}` — `/operations/` queda fuera de alcance, no hay que resolver su desconexión ni sus botones sin handler.
   - 🔄 **Por validar:** si de verdad se necesita un stepper manual de despacho como el actual (cancelar/reintentar por paso, sin "pausar"), o si al analizar los meets y cómo trabajan hoy en SIGAC 2 conviene un flujo distinto. Se decide cómo debería quedar una vez que mejoremos el flujo completo, no se asume que el stepper actual es el punto de partida fijo.
   - Duplicidad de componentes: dos `DispatchMonitor`, dos versiones de "Glosa" (`StepGlosa` sin usar vs. `StepPedimento` real) — con la decisión de dejar solo DGO, esto probablemente se resuelve solo al retirar el step GLOSA.
   - Tres mecanismos de polling que pueden solaparse (1s + 2s + 3s).
   - Código muerto: familia `PedimentoProformaDocument`, `PedimentoHbsDocument`, tab `ReferencePedimento.tsx` no montada.
   - Feedback inconsistente: "Solicitar Fondos" usa `alert()` nativo en vez de toast.

---

## f) Puntos abiertos generales (consolidado)

1. ✅ **Resuelto — "Previo":** en marítimo y aéreo es el nombre que le dan a la revisión de la mercancía cuando llega el barco o el avión. Ver glosario.
2. ✅ **Resuelto — MOMP:** es el apartado de configuración de reglas de negocio de un cliente. Ver glosario.
3. ✅ **Resuelto — "Anticipado":** no es una modalidad. Es la práctica de ir llenando pasos/campos poco a poco con los datos que van llegando, para adelantar trabajo. Ver glosario.
4. ✅ **Resuelto — Nivel de carga del expediente/glosa:** como planteó German, los documentos comerciales se sacan del formulario/wizard de creación de referencia y se quedan en el **detalle de referencia**. Con esto la pregunta de "¿referencia u operación/pedimento?" se resuelve del lado de referencia — el detalle de referencia es donde vive el expediente.
5. ✅ **Resuelto — Automatización de detección de documentos a glosar:** nueva decisión (reunión con German, no cubierta en los meets ya leídos): **cada vez que se sube un documento, se deben glosar todos de nuevo** con **Zeus**, validando que (a) el documento es válido por sí solo ante la ley aduanera, (b) es consistente con el resto de documentos y los DGO, y (c) es consistente con el **previo** (verificación física de la mercancía).
6. 🔄 **Aclarado, pendiente de diseño — Suma automática de fletes en consolidados aéreos:** confirmado en M11 (~00:28:44): "el sistema no suma automáticamente los totales de flete en consolidados, el personal realiza estos cálculos manualmente en hojas de apoyo." Tu explicación agrega el por qué: cada guía tiene su propio incrementable a nivel factura, pero en SIGAC 2 **solo se puede crear un incrementable por factura a nivel referencia** (no por guía) — por eso usan Excel para sumar y declarar un incrementable global de la referencia. Para el refactor falta decidir: ¿SIGAC 3 permite incrementable por guía y los suma automáticamente a nivel referencia, o se mantiene el esquema de un incrementable global con mejor ayuda de captura?
7. 🔄 **Por definir en el refactor — Menú lateral vs. tabs:** sin decisión tomada; se debe establecer el diseño de la mejor manera como parte del trabajo de refactor (Paso 2), evaluando el volumen real de tabs de D1.
8. 🔄 **Por validar en el refactor — Mapeo de documentos a pedimentos específicos en operaciones complejas:** pospuesto explícitamente para después del Happy Path (M9). Se debe validar al diseñar el refactor y decidir cómo se hará siguiendo las mejores prácticas.
9. ✅ **A implementar en el refactor — Alertas automáticas de vencimiento de folios B1:** propuesto en M6, no implementado en D1. Confirmado: hay que implementarlo como parte del refactor.
10. ✅ **Resuelto — Significado de MS/IM/IC:** encontrado en la transcripción completa de M6. Ver glosario ("Identificadores de operación virtual").
11. ⚠️ **Actualizado — Movimientos especializados por tráfico:** German enriqueció la taxonomía completa (ver glosario "Movimiento"): terrestre, aéreo y marítimo ya tienen definidos sus movimientos de **importación**; marítimo también tiene los de **exportación** completos. Quedan sin definir: **Exportación Terrestre** y **Exportación Aéreo** — falta esa sesión con los ejecutivos correspondientes. También quedó pendiente confirmar la tensión con M0b sobre si "Operación nace de Entrada/Subdivisión" aplica igual a los movimientos de importación de aéreo/marítimo (ver nota en el glosario).

---

## Siguiente paso

**Estado real de las 5 Brechas de la sección e), a hoy:**

| # | Brecha | Estado |
|---|---|---|
| 1 | DGO vs. step GLOSA | ✅ Resuelta (queda solo un detalle de implementación: cómo se retira el paso del stepper — se puede resolver cuando diseñemos esa pantalla puntual) |
| 2 | Orden GLOSA vs. MANIFESTACION | 🔄 Reformulada, **sigue pendiente**: falta hacer el ejercicio de validar los 8 pasos del despacho contra los meets |
| 3 | Shipper/Chiper | ⏳ **Pendiente**: falta validar dónde vive en el refactor |
| 4 | Solicitud de Fondos | ⏳ **Pendiente**: falta planear la pantalla con base en lo platicado con Miguel |
| 5 | Marca de operación virtual (B1) | ⏳ **Pendiente**: falta analizar el esquema de base de datos actual |

Y de los 10 puntos abiertos generales de la sección f), quedaron **9 resueltos** (1 al 7, 9 y 10) y **1 con tarea clara para el refactor, no bloqueante** (8, mapeo de documentos en operaciones complejas).

**Mi lectura:** nada de esto bloquea seguir. Las brechas #2, #3, #4 y #5 son específicas de pantallas puntuales (despacho, solicitud de fondos, apertura de operación virtual) — se pueden resolver cuando lleguemos a diseñar cada una de esas pantallas en el Paso 2, en vez de resolverlas todas por escrito ahora. La única que sí convendría cerrar antes de armar el inventario completo de pantallas es la **#4 (Solicitud de Fondos)**, porque su tamaño real (construir la lógica de 3 categorías desde cero) puede cambiar cuántas pantallas/pasos necesita esa parte del flujo.

¿Damos el visto bueno al documento así y pasamos al Paso 2 (inventario de pantallas), dejando #2, #3 y #5 para resolverse pantalla por pantalla — o prefieres primero una sesión rápida para cerrar la #4?
