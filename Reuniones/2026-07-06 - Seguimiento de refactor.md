# Reunión 2026-07-06 — Seguimiento de refactor

Sesión de diseño Ángel/German sobre la gestión documental de SIGAC 3: una
**fuente de verdad** documental, portal de clientes para carga, y la **factura
glosada** como consolidado de validación. Enlaza con la digitalización vista en
[[2026-06-30 - Daily Scrum - e-documents y COVE]].

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo)
- German Castro (producto/dirección — presenta la interfaz)

## Decisiones (con su porqué)
- **La sección de documentos comerciales será la "fuente de verdad"**
  documental, donde se integran todos los documentos subidos (packing list,
  facturas…), con un **expediente electrónico** que incluye identificación
  legal, fiscal y opinión de cumplimiento del SAT, y **mantiene historial** de
  documentos cargados y reemplazados. Porqué: garantizar trazabilidad.
- **Portal de clientes por URL pública.** El cliente accede por una dirección
  web para cargar los documentos de sus operaciones; la interfaz muestra el
  total de documentos **requeridos, validados, pendientes y rechazados**, y la
  información se valida y extrae automáticamente después. Porqué: facilitar y
  descentralizar la carga documental.
- **Mapeo obligatorio de documentos ↔ perfil operativo del cliente.** Se mapean
  los documentos obligatorios a nivel general y **por perfil de cliente** para
  vincular con precisión los archivos subidos con la referencia. Porqué:
  eliminar la falta de conexión archivo↔referencia que era un problema en
  **SIGAC 2**.
- **"Factura glosada" como fuente de verdad definitiva.** Documento consolidado
  que compara los datos extraídos contra el packing list, los resultados de
  inspecciones previas y otros documentos, **marcando discrepancias** hasta que
  toda la información —incluyendo incrementables y configuraciones legales— esté
  validada y alineada. Porqué: consolidar y validar datos automáticamente
  durante la operación. (→ German la presenta para aprobación.)
- **Centro de comando de la referencia.** La pantalla de detalle de la
  referencia es el **punto pivote central** de la operación: integra
  movimientos, mercancía, costos globales y configuración legal, combinando la
  factura glosada con el estado de la documentación pendiente para una visión
  completa.

## Lógica de negocio
- **Documentación indispensable para el alta de clientes / habilitación de
  operaciones:** Constancia de Situación Fiscal, Opinión de Cumplimiento, Acta
  Constitutiva, Poder Notarial, identificación oficial, comprobantes de
  domicilio, Certificado de Sello Digital, documentos de representación legal y
  padrón de importadores.
- **Carga integrada en el flujo de nueva referencia.** El usuario puede cargar
  archivos directamente en su referencia o completar los espacios faltantes,
  viendo un **resumen de progreso porcentual** de la documentación requerida.
- **Validación y extracción de datos.** El objetivo no es solo la extracción
  individual sino un **consolidado**: asegurar que cantidades, números de parte
  y pesos coincidan entre documentos, señalando cualquier discrepancia detectada.
- **Adopción / resistencia esperada (Ángel).** Se anticipa resistencia del
  equipo porque cada persona trabaja distinto y por los "vicios" de SIGAC 2. Las
  áreas de **marítimo y aéreo** han mostrado mayor disposición a los cambios de
  flujo que **operación terrestre**.
- **Pendiente de desarrollo:** el listado de operaciones lo está desarrollando
  Carlos.

## Tareas
> Detectadas, no creadas.
- (German) Presentar la propuesta de **factura glosada** y el flujo de documentos
  para su aprobación.

## Temas descartados por irrelevantes
- Cierre con comentarios informales entre los participantes.
- Pie de página y encuesta de calidad generados por Gemini.
