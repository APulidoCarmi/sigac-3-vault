# Reunión 2026-06-30 — Prueba operación real (glosa, validación, pago y DODA)

Cierre del recorrido de la operación real: desde la glosa del pedimento hasta el
pago, la generación del Shipper y el DODA. Continúa
[[2026-06-26 - Prueba operación real - flujo ejecutivo y consolidación de documentos]]
y la sesión de e-documents [[2026-06-30 - Daily Scrum - e-documents y COVE]].

## Asistentes
Lista de convocados; participaron Perla Lopez, Ángel, Enrique Garza, Enrique
Lopez y Brenda Rentería:
- Angel Huberto Pulido Burgos (desarrollo), Perla Lopez (operación — presenta),
  Enrique Garza (operación), Enrique Lopez (operación), Brenda Rentería
  (pagos/tesorería), German Castro, Carlos Alexis Galaviz Rosas, Fernando Angel
  Lopez Soto, Elian Shair Armendariz Puch, Cesar Aguirre, Daniel Peña, Geovanny
  Manuel Martinez, Jose Antonio Limon, Salvador Cervantes Franco, Yolanda
  Resendez Ortiz, Claudia Andrade, Miguel Gomez, Brenda Rentería, Adriana
  Margarita Eguia Guerrero, Juana Maria Gonzalez, Jesus Vega Portillo.

## Decisiones (con su porqué)
- **Abrir un nuevo ticket "Validación de total de facturas"** vinculado a la
  historia original. El error: la diferencia de cuadre del total se muestra
  **arriba en formato pequeño y poco evidente**, pero no se marca claramente en
  el total inferior. Aunque ya se reportaba en tickets previos, Ángel decide un
  ticket nuevo para mayor claridad; Enrique Garza lo abre en Jira. (Ver Tickets.)
- **Queda a evaluar el nivel de carga de la glosa** (referencia vs
  operación/pedimento) — porqué: una operación puede consolidar hasta **16
  referencias**, y subir el documento a una referencia "aleatoria" no es lo
  ideal para información compartida. Hoy se usa la **referencia más antigua**
  porque aparece primero en el pedimento y sirve de identificador de control
  (Perla).
- **Queda a analizar optimizar la carga de documentos para glosa** eliminando la
  necesidad de consolidarlos manualmente en un solo archivo. Contrapunto
  (Enrique Garza/Enrique Lopez): la separación actual existe porque ciertos
  documentos tienen **códigos de digitalización específicos para ventanilla**
  (e-documents) y otros solo son soporte interno/historial.

## Lógica de negocio
- **Candado de glosa.** El sistema obliga a digitalizar el archivo "glosa de
  pedimento" **y** un checklist antes de habilitar el envío del trámite a
  revisión (trámite y despacho). El documento de glosa es el comprobante de que
  el ejecutivo ya revisó y corrigió el pedimento descargado; el **checklist en
  Excel** (certificados, observaciones, datos a verificar) confirma la
  integridad de la operación.
- **"Documentos para glosa" (archivo combinado).** Para operaciones complejas
  (p. ej. Nestlé con permisos de salud o inspección de bodega) se arma un
  archivo único con facturas, packing lists, certificados de origen y permisos.
  Sirve para revisar glosa contra pedimento en simultáneo **y** para que bodega
  lo imprima y entregue al transportista en los puntos de inspección de la
  autoridad.
- **Notificaciones de glosa.** Al enviar el trámite, el sistema notifica por
  correo a trámite y despacho (número de glosa, quién la envió, fecha). Trámite
  autoriza o rechaza; si autoriza, el ejecutivo recibe comentarios de revisión y
  la hora exacta de autorización, y puede proceder a validación y pago.
- **Manifestación de Valor (MV).** Alta **manual** en SIGAC: valoración,
  incrementables y precios pagados por cada pedimento. Luego se digitalizan
  documentos específicos (factura americana, póliza de seguro…), se generan los
  e-documents ante VUCEM (la nota dice "Busem") y el ejecutivo **firma** la MV
  con sellos precargados. Se genera **una MV por cada pedimento**, aunque
  pertenezcan a la misma operación.
- **MV hecha por el cliente.** El ejecutivo genera un "archivo de validación"
  (borrador manual en SIGAC) y lo comparte; el cliente genera su folio de MV, lo
  sube a una carpeta compartida en Drive, y el ejecutivo toma el folio final
  para declararlo en el pedimento.
- **Validación electrónica del pedimento.** Se hace en una pantalla según la
  **patente aduanal**; el archivo se envía al prevalidador (**Karem**) y se
  refresca hasta el estado "validado". Ante error, el sistema muestra un detalle
  técnico (p. ej. **código ST22** si falta un permiso) para corregir y
  reintentar.
- **Esquemas de pago del pedimento (3):** (1) **Cuenta de Orden** — cuenta PECE
  del cliente vinculada al banco; (2) **Financiado** — Carmi paga y se autoriza
  vía chat a tesorería; (3) **Anticipo** — requiere solicitud de fondos previa.
  En anticipo, si el depósito del cliente es insuficiente, tesorería **bloquea**
  hasta cubrir la diferencia o autorizar financiamiento por cobranza.
- **Mecanismo de envío de pago (Brenda).** Autorizados los fondos, el ejecutivo
  genera un archivo de pago que viaja por un servidor hacia el banco; contiene
  líneas con cuenta bancaria, clave e importe exacto. El banco descuenta
  automáticamente y el sistema pasa el estatus a "pagado".
- **Solicitud de fondos (contado).** Desde el módulo de tráfico se genera el
  formato de solicitud de fondos al importador con el detalle de impuestos y
  honorarios; el cliente responde con comprobantes de depósito, que tesorería
  valida y facturación usa para cerrar la operación.
- **Shipper.** Obligatorio cuando el valor de la mercancía supera **2,500 USD**
  (ejemplo mostrado: 56,437.50 USD). Se genera ingresando **remotamente a una
  computadora en Texas** con plantillas por cliente/proveedor, sustituyendo
  datos para obtener el folio/clave **ITN**; el archivo se sube a SIGAC en
  remoto para incluirlo al reportar el DODA.
- **DODA (Documento de Operación para Despacho Aduanero).** Tras el pago se envía
  correo a trámite y despacho con número de pedimento, CAT, folio, número de
  caja y cliente para que generen el DODA. Documentos de una operación normal:
  carta por T, factura y Shipper; con inspección se añaden esos tres al conjunto
  que se entrega al operador. El DODA permite la salida de la mercancía y la
  modulación.
- **Monitoreo de DODAs.** Trámite a veces envía el DODA directamente o se
  consulta en "control de generación de DODA"; búsqueda por número de pedimento,
  número de DODA o control de envíos, con filtros por rango de fecha, cliente o
  pedimento.
- **Tiempos muertos.** Durante la espera de glosa o pago, los ejecutivos
  adelantan el "shipper" o la facturación de otras operaciones del mismo cliente.

## Tareas
> Detectadas, no creadas.
- (Grupo) Evaluar si los documentos de glosa deben subirse a nivel
  **operación/pedimento** en lugar de referencia.
- (Grupo) Analizar cómo mejorar el flujo de carga de documentos para glosa
  eliminando la consolidación manual en un solo archivo (respetando los códigos
  de digitalización de ventanilla).
- (Enrique Garza) Abrir ticket Jira **"Validación de total de facturas"** por el
  error del cuadre poco visible, vinculado a la historia original.

## Tickets
- `Validación de total de facturas` — Jira, a crear por Enrique Garza (sin ID
  aún); vinculado a la historia original del error de cuadre de totales.

## Temas descartados por irrelevantes
- Cierre de la reunión y recordatorio de "revisar temas pendientes en Jira"
  (logística).
- Pie de página y encuesta de calidad generados por Gemini.
