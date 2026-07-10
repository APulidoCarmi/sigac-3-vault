# Reunión 2026-06-25 — Daily Scrum (flujo de importación y checklist dinámico)

Sesión enfocada en **simplificar la UI con IA** y establecer un **flujo de datos
estructurado**. Perla recorre una operación terrestre real (cliente **NALCO**)
de principio a fin para entender los procesos actuales (marítimo, aéreo,
terrestre) y alimentar el rediseño. Se cierra definiendo el "happy path".

## Asistentes
- Angel Huberto Pulido Burgos (convoca — desarrollo)
- German Castro (producto/dirección)
- Perla Lopez (operación — presenta el ejemplo NALCO)
- Enrique Lopez (operación)
- Cesar Aguirre, Claudia Andrade, Enrique Garza, Daniel Peña

## Decisiones (con su porqué)
- **Checklist dinámico de documentos.** Los requisitos documentales **no son
  estáticos**: resultan de combinar perfil del cliente (**MOMP**), número de
  parte, aduana y tratados aplicables. El sistema debe gestionar un checklist que
  **crece/ajusta según el contexto**. Ej.: acero y aluminio activan requisitos
  específicos (certificado de molino, avisos automáticos).
- **Validar documentos obligatorios desde la creación de la referencia.** Factura
  comercial y packing list son los básicos obligatorios; el sistema debe
  **advertir automáticamente** de documentos faltantes o requisitos adicionales
  derivados de la mercancía, para asegurar cumplimiento normativo.
- **Integrar la verificación física en la app** (sustituir el PDF externo del
  "expediente completo") para **comparar datos reales vs factura** y detectar
  discrepancias automáticamente, mejorando la coherencia del expediente digital.
- **Permitir excepción bajo autorización gerencial.** El bloqueo por falta de
  revisión física **no debe ser total**: hay casos de **despacho inmediato** bajo
  responsabilidad del cliente. Se propone que el sistema permita continuar
  (documents, COVE, manifestación de valor) bajo autorización de un **gerente o
  del propio cliente**.
- **Esquema gráfico del flujo ideal ("happy path")** para estandarizar la
  ejecución del sistema. Ángel lo elabora y lo presenta en el daily del día
  siguiente.

## Lógica de negocio
- **Inicio de la operación (terrestre, NALCO):** la línea de transporte solicita
  **cita al almacén** con un número de **BOL** y otros datos — paso clave para
  evitar costos de almacenaje (el almacén requiere documentación previa para
  recibir la mercancía).
- **Documentos:** factura comercial + packing list obligatorios en la mayoría de
  importaciones; COA, certificado de origen, etc. dependen del cliente o de la
  naturaleza de la mercancía. Si solo hay packing list, se registra para
  **anticipar la referencia** y se convierte a factura cuando esta llega. La
  digitalización de documentos adicionales depende de cada cliente.
- **Clasificación:** al dar de alta partidas, si el número de parte ya está
  clasificado el sistema despliega su descripción; si no, se deriva al
  clasificador (que recopila fichas técnicas, fotos, uso). La **operación puede
  generarse antes de terminar la clasificación**, pero **no calcula impuestos ni
  actualiza datos** correctamente hasta que la clasificación esté completa.
- **Revisión física (expediente completo):** contiene el resultado de la revisión
  física (daños, discrepancias). Es **obligatoria** en importaciones; su ausencia
  **bloquea** la orden de carga, la validación del pedimento y el envío al
  prevalidado (**Karem**). Perla identifica un **error/bloqueo actual** que impide
  validar pedimentos por falta de revisión física.
- **Cálculo de impuestos (automático):** IVA, prevalidación e IVA de la
  prevalidación; sin certificado de origen el sistema calcula IGI y DTA con el
  **tipo de cambio del día siguiente** (publicado) para el pago. El personal debe
  conocer el detalle para explicar los cobros al cliente.
- **Solicitud de fondos (ejemplo NALCO, contado):** ~44,000 MXN (cuenta mexicana)
  y 425 USD (cuenta americana) cubriendo embarque consolidado, almacén, chiper y
  transfer; impuestos por 25,257 MXN + honorarios.
- **Referencias y pedimentos:** un tab vincula múltiples pedimentos a una sola
  operación (referencias consolidadas); el **régimen se ingresa manualmente en
  tráfico**. Otras secciones: transporte/vehículos, eventos de seguimiento (lluvia
  de información al cliente), gastos comprobados, características vinculares (las
  usa bodega), bitácora de notificación de estatus (registro histórico de por qué
  la mercancía no ha salido y cuándo se notificó al cliente).
- **Flujo final definido (happy path):** documents → COVE → glosa → manifestación
  de valor → transmisión → validación → pago del pedimento → generación de DODA
  → y, si aplica por monto (2,500 USD), chiper.

## Tareas
> Detectadas, **no creadas** (a petición del usuario).

**Próximos pasos explícitos del daily:**
- (Ángel) **Documentar el flujo operativo** actual para optimizar el diseño de la
  UI con IA.
- (Ángel) Crear un **esquema del flujo de trabajo actual (happy path)** y
  presentarlo en el daily del día siguiente.
- (German) **Verificación cruzada con Charlie** cuando el sistema de verificación
  saque los documentos, para asegurar que caigan en la referencia correctamente.

**Requerimientos de sistema derivados:**
- Checklist dinámico de documentos por perfil de cliente (MOMP) / número de parte
  / aduana / tratados.
- Validación de documentos obligatorios (factura + packing list) desde la
  creación de la referencia, con advertencias automáticas.
- Integrar el reporte de verificación física en la app (sustituyendo el PDF),
  comparando datos reales vs factura para detectar discrepancias.
- Override/excepción bajo autorización gerencial o del cliente para continuar
  (documents/COVE/manifestación de valor) pese a falta de revisión física.
- Corregir el error/bloqueo que impide validar pedimentos por falta de revisión
  física.

Relacionado: [[2026-04-28 - Prueba operación real desde ticket a facturación]].

## Temas descartados por irrelevantes
- Enlaces y pie de página generados por Gemini (encuesta de calidad, adjunto de
  calendario, aviso de "revisa las notas").
