# Reunión 2026-03-19 — Análisis de Referencia y Movimientos

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo SIGAC 3.0)
- Enrique Lopez (ejecutivo de cliente / operación)
- German Castro (producto/dirección)
- Carlos Alexis Galaviz Rosas (implementación / ingeniería)
- Perla Lopez (operación)
- Sala de Juntas Laredo TX (equipo de almacén Laredo)
- Jeniffer Castro

## Decisiones (con su porqué)
- **Separación conceptual Referencia vs Movimientos.** La *Referencia* es el
  expediente **documental**: número que no cambia, da seguimiento desde el
  anticipo. Los *Movimientos* concentran todo lo **físico**: entradas/salidas
  de almacén, inventario, funciones tipo WMS. Cada concepto tiene una
  responsabilidad distinta y **deben mantenerse separados** — porqué: mezclar
  los conceptos es lo que hoy cuesta y causa problemas; al separarlos, el
  trabajo de almacén cae en movimientos y el de ejecutivo/documental en
  referencia, pudiéndose interconectar (ventaja de que Carmi hace ambas cosas
  en un mismo sistema, a diferencia de comprar un WMS externo).
- **No se disminuye capacidad, se aumenta.** German recalca que el nuevo
  esquema da mayor granularidad, control y veracidad de inventario respecto al
  esquema anterior basado solo en referencia.
- **Numeración de movimientos desde 1 (consecutivo), no desde 17,000.**
  Enrique lo pidió (por legibilidad del consecutivo de salidas/entradas).
  Carlos indica que **ya lo agregó** (se lo había pedido Eli) y que también
  **ya agregó la columna** que muestra el dato en la vista. Queda pendiente
  únicamente revisarlo juntos tras la junta.

## Lógica de negocio
- **Reglas de movimientos:**
  - Un movimiento de entrada da de alta mercancía (aparece en inventario); un
    movimiento de salida la da de baja (evento de despacho = ya no está
    físicamente en locación).
  - Regla adaptable **por aduana/cliente**: en **Laredo**, todo movimiento de
    entrada debe estar asociado a una referencia; en **Bronzeville**, no (el
    WMS solo gestiona almacén; todo por movimiento de entrada/salida, sin
    necesidad de casarse con referencia).
  - Se pueden generar **movimientos de entrada sin referencia**, pero **todo
    movimiento de salida requiere referencia** y requiere un movimiento de
    entrada previo.
  - Una factura puede **repartirse en dos regímenes / dos pedimentos**
    diferentes.
- **Granularidad de salida:** a una referencia se le pueden dar N entradas
  (varias facturas, cada una en el/los movimientos que se quiera). Al salir, el
  movimiento de salida permite elegir **qué facturas, qué partidas y qué items**
  sacar, en una o N salidas. La mercancía disponible se muestra por lo que hay
  en inventario en ese momento; se vincula a la **referencia (padre)**, no hay
  vínculo rígido entrada↔salida. El movimiento de entrada solo indica al almacén
  **qué** llega y **cuándo/cómo** (analogía dada: cajas que llegan en días
  distintos = entradas; el inventario total es lo que puedes sacar después).
- **Retorno / reexpedición (caso Perla):** mercancía dañada que el cliente
  retorna se maneja con **dos movimientos de salida**: uno hacia el pedimento y
  otro de retorno. Reexpedición **no maneja pedimento pero sí inventario**; el
  movimiento de salida de retorno es más sencillo: no llena campos de pedimento,
  solo instrucción de salida (transportista, fecha) y una **orden de carga al
  final del día** que almacén comprueba contra su etiqueta antes de dar de baja.
- **Trazabilidad:** manteniendo referencia como expediente + movimientos, se
  obtiene el historial completo por número de partida (cuándo se dio de alta,
  cuándo de baja, si salió junto o no, si hubo SND, si se regresó), todo
  englobado en la referencia. Cada bulto se etiqueta con un identificador y la
  orden/instrucción de carga debe salir con **la misma identificación** →
  debe hacer *match* físico contra la etiqueta.
- **Candado en divisiones (requerimiento de almacén Laredo):** el ~90% de las
  divisiones ocurre *después* de la verificación física. Cuando el ejecutivo
  manipula/divide una referencia ya verificada, el sistema debe **quitar el
  candado** de la verificación original y **notificar a almacén**; el ejecutivo
  **no puede validar ni pagar** hasta que almacén extraiga físicamente el
  material, valide y **vuelva a candadear** la división correcta. Se gestionará
  desde la torre de control.
- **Catálogos requeridos para avanzar a pedimentos** (hoy las capturas no salen
  clasificadas): números de parte (migrar del SIGAC 2; ya hay perfil por número
  de parte/producto), proveedores por cuenta (Fernando ya trabajó cómo se ligan;
  falta darlos de alta y validar con ejecutivos) y transportistas. Sistema 100%
  administrable desde web; se levantará con Fer en el daily.

## Tareas
- (Ángel) Permitir sacar **dos o más pedimentos en una sola salida/operación**
  (hoy es un pedimento por salida) — cambio ya identificado como pendiente.
- (Ángel) Desarrollar el **módulo de Movimientos** (seguimiento y búsqueda por
  número de movimiento, además del número de referencia) — para la siguiente
  entrega.
- (dev) Implementar el **candado/notificación en divisiones**: al dividir una
  referencia verificada, notificar a almacén y bloquear validación/pago del
  ejecutivo hasta re-candadeo por almacén (torre de control).
- (Carlos/Fer) Cargar **catálogo de números de parte** migrando del SIGAC 2 y
  lograr que las capturas salgan clasificadas; notificar a ejecutivos para
  validar.
- (Fernando/Carlos) Dar de alta **catálogo de proveedores por cuenta** y
  notificar a ejecutivos para revisión.
- (Carlos) Cargar **catálogo de transportistas**.
- (Enrique + almacén Laredo) Correr **pruebas PAP end-to-end en staging**
  (equipo Enrique → almacén → regreso hasta generación de pedimento), en doble
  función con el flujo convencional, sin atrasarlo. Clientes propuestos: GKN,
  troquelados y Ecolab (pueden ser todos, no limitarse a tres).

## Temas descartados por irrelevantes
- Saludos, despedidas y agradecimientos de cierre.
- Problemas técnicos del meet (comprobaciones de audio "¿me escuchas?").
- Muletillas y repeticiones propias de la transcripción automática.
