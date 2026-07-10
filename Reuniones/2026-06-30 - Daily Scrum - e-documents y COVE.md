# Reunión 2026-06-30 — Daily Scrum (e-documents y COVE)

Capacitación de Perla sobre el flujo digital de documentos: digitalización /
e-documents, catálogo de conceptos, expediente de comercio y generación de COVE.
Continúa la revisión del proceso operativo iniciada en
[[2026-06-26 - Prueba operación real - flujo ejecutivo y consolidación de documentos]].

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo)
- Perla Lopez (operación — presenta/capacita)
- Cesar Aguirre, Enrique Lopez, Enrique Garza, Claudia Andrade, Daniel Peña,
  German Castro

## Lógica de negocio
- **Digitalización por concepto, no genérica.** Se busca por número de operación
  y los documentos se categorizan según conceptos específicos (lista de empaque,
  T-MEC…). La **factura comercial bajo el anexo 10** debe marcarse con conceptos
  específicos para que aparezca en el expediente final del cliente.
- **Catálogo de conceptos (claves del SAT).** Distingue documentos que **aplican
  para VUCEM** de los que **no aplican**: los que aplican se transmiten a la
  ventanilla; los que "no aplican" quedan solo en el sistema de referencia,
  aunque pueden subirse si el cliente los requiere para su archivo personal.
- **Expediente de comercio.** Todos los documentos digitalizados —incluidos los
  que no se envían a VUCEM— se integran **automáticamente** en el expediente de
  comercio que se genera **al pagar el pedimento**. Cesar confirma que el
  expediente captura todo lo escaneado, cumpliendo la entrega de información al
  cliente.
- **Documentos no obligatorios (p. ej. órdenes de compra).** Se pueden subir como
  **"anexos"** según preferencia del usuario/cliente, sin afectar
  obligatoriamente el flujo de e-documents.
- **e-documents múltiples por operación.** Se generan según los conceptos
  involucrados. Estrategia acordada: para facturas de glosa que **no** requieren
  e-document, marcar el concepto como **"sí aplica"** para que aparezca en el
  expediente pero **omitir la generación del e-document**, evitando duplicidad o
  errores con el anexo 10.
- **COVE (la nota lo abrevia "COV").** Se genera desde el módulo de control
  buscando por operación → arroja el número de COVE y el acuse necesario para el
  pedimento. Se genera **un COVE por cada factura comercial**: si la operación
  incluye varias facturas, la generación es individual por cada una.
- **Relevancia del COVE en el reconocimiento aduanero (Cesar).** El número de
  COVE permite a las autoridades visualizar la información capturada (valor,
  descripción de la mercancía) por sistemas digitales, validando la
  documentación sin revisar documentos físicos.

## Tareas
> Detectadas, no creadas.
- (Ángel, Perla) Retomar la revisión del proceso de **glosa** en la reunión de
  la tarde y completar la explicación de los requerimientos de los archivos de
  pedimento. *(Compromiso/logística de reunión.)*

## Temas descartados por irrelevantes
- Disculpa de Ángel por ausencia previa y arranque de agenda.
- Cierre de la sesión (Perla debe atender a un cliente; se pospone la glosa).
- Pie de página y encuesta de calidad generados por Gemini.
