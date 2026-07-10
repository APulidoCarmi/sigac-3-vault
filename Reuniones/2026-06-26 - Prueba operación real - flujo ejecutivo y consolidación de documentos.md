# Reunión 2026-06-26 — Prueba operación real (flujo ejecutivo y consolidación de documentos)

Continuación de la revisión de principio a fin de una operación real. Foco:
simplificar las pantallas del ejecutivo (quitar lo que es de almacén) y decidir
cómo consolidar documentos hacia VUCEM. Se recorre un caso real de captura
compleja (~19 referencias). Continúa [[2026-04-28 - Prueba operación real desde ticket a facturación]].

## Asistentes
Lista de convocados (Gemini "Invitado"); participaron activamente Ángel,
Perla Lopez, Enrique Lopez, German Castro, Fernando Angel Lopez Soto y
Enrique Garza:
- Angel Huberto Pulido Burgos (desarrollo), German Castro (producto/dirección),
  Perla Lopez (operación), Enrique Lopez (operación), Enrique Garza (operación),
  Fernando Angel Lopez Soto (ingeniería), Carlos Alexis Galaviz Rosas
  (implementación/almacén), Elian Shair Armendariz Puch (desarrollo), Miguel
  Gomez, Cesar Aguirre, Daniel Peña (almacén), Jose Antonio Limon, Salvador
  Cervantes Franco (clasificador), Yolanda Resendez Ortiz, Claudia Andrade,
  Brenda Rentería, Adriana Margarita Eguia Guerrero, Juana Maria Gonzalez,
  Jesus Vega Portillo.

## Decisiones (con su porqué)
- **Simplificar las pantallas del flujo del ejecutivo.** Quitar información y
  acciones que no le corresponden al ejecutivo (son de almacén u otras áreas)
  — porqué: las pantallas actuales incluyen acciones que el ejecutivo no
  realiza; la visualización debe acorde a sus tareas reales.
- **Autorización especial para operaciones urgentes sin revisión física
  completa.** Permitir pagar el pedimento antes de que almacén confirme la
  recepción completa de la mercancía, **asumiendo el cliente la
  responsabilidad** por posibles discrepancias. (Coincide con el override
  gerencial de [[2026-06-25 - Daily Scrum - flujo de importación y checklist dinámico]].)
- **Analizar primero una operación "compleja para capturar" (~19
  referencias)**, no una "delicada" (permisos/revisiones exhaustivas) — porqué:
  un caso de captura compleja da mejor material para el rediseño funcional.
- **Módulo "plan de carga" queda para el futuro, no la versión inmediata.**
  Idea: el cliente sube su propio plan de carga (basado en su inventario) y el
  sistema **genera referencias automáticamente**. Porqué: evitar la carga
  manual y aprovechar los datos del inventario; se registra como historia de
  usuario en el backlog para después.
- **La consolidación manual de PDFs se mantiene por ahora**, pero se integra
  una **pantalla intermedia de verificación** en el rediseño para que el
  ejecutivo confirme que los documentos seleccionados son los correctos antes
  de enviarlos a VUCEM/glosa. Porqué: mitigar la "ansiedad operativa" de
  transmitir sin revisión final. Propuesta de Ángel (marcar en cada referencia
  los documentos que van a VUCEM y que el sistema los consolide
  automáticamente al generar el pedimento) — Perla la reconoce funcional.
- **Crear ticket para configurar en MOMP la exigencia de proforma y cartas
  firmadas.** Hoy esa configuración **no existe** en el sistema (confirmado por
  Fernando); German solicita el ticket por ser esencial para la operación.

## Lógica de negocio
- **Flujo estándar del ejecutivo:** solicitud de cita → creación de la
  referencia en SIGAC 2 → captura y digitalización de documentos (factura +
  packing list) → validación de partidas → clasificación → revisión física por
  almacén.
- **Expediente completo:** por cada referencia se verifica factura sellada +
  revisión física al momento de la llegada. Una vez completo, no se requiere una
  segunda revisión salvo que haya que comprobar la carga física.
- **HASMAT (materiales peligrosos):** se identifica mediante el BOL. Se puede
  transportar HASMAT junto con mercancía regular en un mismo camión si se
  cumplen las normativas de la caja. Los lineamientos sugieren no mezclar, pero
  el cliente puede decidir asumir el riesgo (German).
- **Vinculación de referencias → pedimento:** se vinculan una a una para
  consolidarlas bajo un mismo pedimento, siempre que tengan la información
  necesaria y sin errores de clasificación.
- **Tabs críticos para el ejecutivo:** Referencia, Pedimento, Inventario, Datos
  de Transporte, Seguimiento y Notificación de Estatus. Otras (valor, ubicación)
  no se usan actualmente.
- **Clasificación pendiente:** si una partida no está clasificada, el sistema
  impide actualizar el cálculo de impuestos; hay que consultar al clasificador y
  al cliente antes de proceder.
- **"Cambios" y OSND (Over, Short, Damaged):** los "cambios" ocurren cuando la
  mercancía recibida difiere de la factura original (de más o dañada),
  obligando a modificar la captura original o a ajustar la factura según decida
  el cliente (Enrique Garza).
- **Proforma y etiquetado (Norma 050):** en la etapa final Perla revisa la
  proforma para verificar peso y cumplimiento (p. ej. Norma 050 de etiquetado).
  El clasificador indica desde el inicio si la mercancía requiere etiqueta, que
  se gestiona y comparte con almacén; la verificación se hace por fotografías.
- **MOMP por cliente:** ciertos clientes, según su modelo (MOMP), requieren
  obligatoriamente proforma y cartas firmadas por el representante legal; la
  configuración no es uniforme para todos.
- **Pre-glosa:** revisión obligatoria antes de enviar cartas — se verifican
  Incoterms, fechas y proveedores; los errores se corrigen manualmente en la
  referencia antes de la gestión legal.
- **Glosa y documentos:** tras revisar y enviar la proforma se solicitan las
  cartas al cliente; recibida la documentación firmada y la confirmación de
  fecha de despacho / número de caja, se procede con la glosa, la digitalización
  y la generación de documentos.
- **COVE y manifestación de valor:** el COVE se hace tras la glosa inicial y la
  recepción de documentos; la manifestación de valor se elabora **después de la
  tercera glosa** (una vez garantizado que no habrá cambios en el COVE), para
  transmitirla y luego pagar.
- **Pago:** en este caso el cliente usa cuenta de orden / cuenta PC, por lo que
  **no** requiere solicitud de fondos. El "chiper" (shipper) se elabora
  separando las partidas por proveedor una vez pagado el pedimento.
- **Digitalización:** se realiza en la referencia **más antigua** del pedimento,
  usando la extensión de SIGAC para cumplir los estándares de DPI de VUCEM. Con
  múltiples referencias, Perla recopila facturas y packing lists en una carpeta
  local y las une en un solo PDF antes de subirlas (busca eficiencia).
- **Caso GKN:** implica convertir facturas de proveedores externos a facturas
  internas (entre empresas) para importaciones temporales; ese proceso
  específico justifica el manejo manual de documentos por los ejecutivos.
- **Logística de unidades (cajas de 53 pies):** la colocación de unidades se
  coordina internamente; no se asignan cajas innecesariamente para evitar
  retrabajos si el plan de carga cambia. Hay reuniones diarias con los clientes
  para definir planes de carga y fechas de salida con la antelación debida.

## Tareas
> Detectadas; su alta en `Tareas.md` se confirma en el flujo de la skill.

**Dev / sistema:**
- (Ángel) Validar con usuarios si la interfaz de selección/revisión de
  documentos es suficiente o requiere una **pantalla intermedia** de
  verificación antes del envío a VUCEM.
- (dev) Configurar en **MOMP** la exigencia de proforma y cartas firmadas por
  cliente (ticket solicitado por German; hoy no existe).
- (dev) **Consolidación automática de documentos a VUCEM:** marcar por
  referencia los documentos que van a VUCEM y consolidarlos al generar el
  pedimento (propuesta de Ángel, validada como funcional por Perla).
- (dev, futuro) **Módulo plan de carga:** historia de usuario en el backlog para
  que el cliente suba su plan de carga y el sistema genere referencias
  automáticamente.

**Compromisos / no-dev:**
- (Enrique Lopez, Perla) Presentar la operación compleja (segunda opción) en el
  siguiente daily.
- (Fernando) Validar si proformas y cartas de firma están contempladas en el
  documento de Modelo Operativo de Procesos.
- (Grupo) Continuar la sesión el lunes para completar la revisión del proceso
  operativo (cierre de mes).

## Temas descartados por irrelevantes
- Saludos, despedidas y small talk.
- Problemas técnicos del meet.
- Pie de página y avisos generados por Gemini (encuesta de calidad, "revisa las
  notas").
