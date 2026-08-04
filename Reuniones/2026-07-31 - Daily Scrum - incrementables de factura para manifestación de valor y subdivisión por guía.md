# Reunión 2026-07-31 — Daily Scrum: incrementables de factura para manifestación de valor y subdivisión por guía

## Asistentes
- Enrique Lopez
- Angel Huberto Pulido Burgos
- Perla Lopez

## Decisiones (con su porqué)

- El campo "incrementable" de factura debe dividirse en los conceptos que exige la manifestación de valor (COVE), porque en factura hoy solo existen 3-4 conceptos (flete, embalaje, seguro y otros) mientras que la manifestación de valor tiene 6-7 conceptos distintos — un solo campo genérico no basta para mapear 1:1 a lo que pide la manifestación.
- La moneda a tomar es la de la factura, pero la manifestación de valor debe reflejar esos montos convertidos a pesos en el campo de la factura / de incrementables.
- Angel confirma que el sistema **ya tiene** por factura los campos de vinculación y método de valoración (no hace falta agregarlos).
- Sigac 3 confirma: **una sola referencia se genera por arribo**, sin importar cuántas facturas traiga ese arribo (una guía puede tener muchas o pocas facturas, no hay tema).
- Preferencia operativa de Enrique ante mercancía incompleta: pedir al cliente que corrija/reemita la factura con solo lo que sí se va a importar, en vez de subdividir. Razón: evita dejar referencias y valores "vivos" pendientes de importar, y evita fricción contable con el proveedor (pago parcial de una factura).
- Legalmente no hay obligación de declarar/notificar nada adicional (ni pedimento de cierre) si lo subdividido y dejado pendiente nunca llega a importarse — lo ya importado pagó sus impuestos y lo pendiente simplemente queda sin importar. Aun así, lo ideal sigue siendo pedir la factura corregida, porque en auditorías años después el cliente puede no tener ya los contactos y se cuestiona por qué se importaron menos piezas de las facturadas.

## Lógica de negocio

### Incrementables de factura → manifestación de valor
- Agregar a factura: fechas, precio pagado/por pagar, forma de pago (transferencia, carta de crédito, cheque, etc.), dos campos libres (uno de ellos para señalar "pagar al final del financiamiento de la factura" y similares).
- El incrementable capturado en factura debe poder dividirse/mapearse a los conceptos específicos que pide la manifestación de valor (la manifestación exige que se señale por COVE a qué concepto va cada incrementable).
- Referencia de implementación: Carlos (de Veracruz), Brenda y Miguel ya están viendo/desarrollando este tema en Sigac 2.0 para actualizarlo pronto — Angel puede guiarse de ahí para definir qué datos agregar exactamente.
- Urgencia: el proceso de manifestación de valor arranca el **2026-08-01** ("mañana"), por lo que estos cambios deben quedar listos para esa fecha.
- Enrique adelanta que la **próxima semana** (semana del 2026-08-03) viene un cambio importante adicional en manifestación de valor.

### Guías y arribo (Sigac 3)
- Las guías **no se declaran** en el pedimento de importación/exportación; se usan para controlar el arribo de mercancía a bodega, anticipar la llegada y evitar que se duplique cuando llega la mercancía.
- Una referencia se genera por arribo, independientemente de cuántas facturas traiga ese arribo.

### Subdivisión de factura por mercancía incompleta o dañada
- **Dos facturas, llega solo una:** no se subdivide, simplemente se quita del arribo la factura que no llegó.
- **Una sola factura, llega incompleta:**
  - Si aún no se ha capturado nada, se espera a que llegue el resto de la mercancía antes de capturar (no se subdivide).
  - Si ya se capturó toda la mercancía de esa factura y falta que llegue una parte, y se quiere importar ya lo que sí llegó, **sí hay que subdividir** la referencia.
- **Camino preferido (mejor operativamente):** pedir al cliente que corrija la factura y mande solo lo que se va a importar.
- **Camino alterno si el cliente no corrige la factura:** subdividir. Ejemplo dado: de 100 piezas facturadas llegan 50 → se declara "subdivisión 1" con 50 piezas y su valor proporcional, dejando pendiente de importar las otras 50 con su valor.
- **Caso real de mercancía dañada:** llegó mercancía que no se va a poder importar completa; se le pidió al proveedor/cliente actualizar la factura con el valor de las piezas que sí se van a importar (las dañadas quedan fuera), como alternativa a subdividir la factura.
- En Sigac 2.0, al subdividir se pueden agregar notas/observaciones de soporte, por ejemplo:
  - Que se importa primero X porque no llegó la mercancía restante y no tiene evento de arribo, respaldado por el reporte de verificación (ej. solo se revisaron 10 piezas de las 20 esperadas).
  - Que la mercancía pendiente de importar se debe a que el cliente no cuenta con el permiso/regulación aplicable para esa mercancía.
- Angel propone dejar evidencia en el sistema (para fines de auditoría) de que se solicitó al cliente soltar la mercancía y que la subdivisión se hizo por esa razón — punto abierto, no se definió aún cómo se registraría ese dato en Sigac 3 (no hay tarea formal, es una idea planteada en la charla).

## Tareas

- [ ] Implementar en factura los conceptos de incrementables desglosados según la manifestación de valor (COVE): dividir el campo genérico de incrementable en los 6-7 conceptos que exige la manifestación (vs. los 3-4 actuales de factura), y agregar a factura fechas, precio pagado/por pagar, forma de pago, dos campos libres y el manejo de moneda (factura en su moneda original, manifestación en pesos). Urgente: el proceso de manifestación de valor arranca 2026-08-01. Guiarse del desarrollo que ya están haciendo Carlos/Brenda/Miguel en Sigac 2.0.

## Temas descartados por irrelevantes
- Saludos, despedidas y problemas de audio/mute durante la llamada.
- Comentario de coordinación sobre no asistir Angel a todas las reuniones de manifestación de valor (delega en Germán y Carlos el seguimiento) — es logística interna de reuniones, no una decisión de producto ni una tarea de sistema.
