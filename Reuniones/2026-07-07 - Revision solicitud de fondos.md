# Reunión 2026-07-07 — Revisión solicitud de fondos

Ángel y Miguel revisan el flujo de solicitud de fondos para SIGAC 3: sus tres
categorías, la automatización de conceptos y su centralización en el módulo de
operación. Complementa el esquema de pago de
[[2026-06-30 - Prueba operación real - glosa, pago y DODA]].

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo)
- Miguel Gomez (pagos / solicitud de fondos — presenta)

## Decisiones (con su porqué)
- **La solicitud de fondos se estructura en 3 categorías:** (1) **gastos
  comprobados**, (2) **impuestos de pedimento**, (3) **honorarios**. Las tres
  deben integrarse para una solicitud completa y coherente.
- **Centralizar la solicitud de fondos dentro del módulo de operación/referencia**
  en lugar de una pantalla separada. Porqué: gestionar las tres categorías desde
  un punto central y mejorar la usabilidad para el ejecutivo. (→ Ángel valida la
  ubicación técnica.)
- **Los honorarios se generan automáticamente** según la configuración de
  tarifas (por peso, porcentaje o criterio configurado). Porqué: eliminar la
  entrada manual de datos por cada prefactura.
- **Prevenir el cobro duplicado de servicios en operaciones con múltiples
  pedimentos.** Algunos servicios son **por operación** y no por pedimento; hoy
  el ejecutivo debe eliminarlos manualmente para no cobrar dos veces el mismo
  concepto. Se decide desarrollar una funcionalidad que lo automatice cruzando
  referencias entre procesos.
- **Notificación directa al cliente vía URL** (propuesta de Ángel). Al generar la
  solicitud, se notifica al cliente por una URL; al depositar, el cliente accede
  al sistema para **adjuntar su comprobante**, lo que notifica automáticamente a
  ejecutivo y tesorería. Porqué: agilizar la recepción del dinero. **Tesorería
  hace una validación final en el banco** para confirmar recepción y monto
  correcto antes de autorizar el pago del pedimento.

## Lógica de negocio
- **Gastos comprobados.** El ejecutivo identifica si la operación los requiere
  (p. ej. lavado de contenedores, maniobras) y registra la solicitud en una
  pantalla específica compartida con tesorería, para transferir fondos al
  proveedor.
- **Rol de tesorería.** La pantalla de solicitud es **informativa / de control
  interno**; **tesorería autoriza y ejecuta** las transferencias bancarias en
  línea tras revisar la solicitud del ejecutivo. La comunicación sobre la
  ejecución de pagos suele ocurrir por canales externos (correo o chat).
- **Impuestos de pedimento.** Los montos se basan en la sección de **"efectivo"**
  del cuadro de liquidación del pedimento; la información se extrae directamente
  de ahí.
- **Honorarios.** Se generan por la **configuración de tarifas** (Sigac 2 y
  próximamente Sigac 3), que determina montos por peso, porcentaje o criterio.
- **Consolidación.** Los tres elementos convergen en la pantalla de solicitud de
  fondos; en SIGAC 2 la solicitud se vincula directamente al pedimento y
  **recalcula y totaliza automáticamente** los montos requeridos, asegurando que
  todos los cargos estén reflejados antes de enviar.
- **Excepciones y pagos parciales.** Ante cambios de montos o pagos parciales, el
  ejecutivo puede hacer una **solicitud complementaria** o ajustar los montos; es
  su responsabilidad mantener la precisión financiera.
- **Automatización al recalcular.** El sistema trae automáticamente impuestos,
  gastos comprobados y servicios, manteniendo la información consistente con las
  otras secciones para generar el documento de solicitud con datos precisos.

## Tareas
> Detectadas, no creadas.
- (Grupo) Desarrollar la funcionalidad que **previene el cobro duplicado de
  servicios** cuando una operación tiene múltiples pedimentos (cruzar
  referencias entre procesos).
- (Grupo) Implementar en SIGAC 3 la **estructura de tres conceptos** (gastos
  comprobados, impuestos y honorarios), replicando y optimizando SIGAC 2.
- (Ángel) Validar la **ubicación técnica** del módulo de solicitud de fondos
  (dentro de operaciones o de referencias).
- (Grupo) Determinar la **viabilidad de integrar** impuestos, gastos y honorarios
  en el proceso de solicitud de fondos.

## Temas descartados por irrelevantes
- Pie de página y encuesta de calidad generados por Gemini.
