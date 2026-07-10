# Reunión 2026-07-01 — Marítimo Sigac 3.0

Demostración (Rocio) del flujo operativo de **importación marítima** en el
sistema, el proceso de **rectificaciones** y una comparación marítimo ↔
terrestre para adaptar SIGAC 3.

## Asistentes
- Angel Huberto Pulido Burgos (desarrollo)
- Rocio Teresa Cortes Salazar (operación marítima — presenta)
- Lilia Candelaria Jimenez Serrano, Carlos Alexis Galaviz Rosas
  (implementación), German Castro, Elian Shair Armendariz Puch, Diana Laura
  Figueroa Jimenez, Mariana Lagunes Moreno, Juan Ramon Martinez Moran, Teresita
  De Jesus Mata Guadarrama

## Decisiones (con su porqué)
- **Adaptar SIGAC 3 al flujo marítimo.** SIGAC 2 está diseñado principalmente
  para operaciones **terrestres**; se busca ajustar SIGAC 3 al flujo marítimo.
  Porqué/cómo: para saber qué campos sobran o faltan hay que **comparar el flujo
  marítimo con el terrestre** (Rocio), de ahí una de las tareas.

## Lógica de negocio
- **Flujo marítimo (Manzanillo).** Apertura de referencia → registro de fecha de
  arribo y estimado de salida. Hay una espera de **4–5 días** para
  desconsolidación y entrega de la **tarja** en el puerto. La operación es de
  **entrada y salida inmediata** para evitar costos de resguardo, salvo que se
  requiera subdivisión de la factura o que la mercancía vaya a otra planta (se
  solicita desconsolidación en terminal).
- **Documentación que envía el cliente por correo:** número de referencia,
  instrucciones, prealerta/manifestación, **BL House**, factura comercial
  (**factura A24**) y un **archivo TXT** con la información detallada de la
  factura.
- **Documentos indispensables** para concluir la operación: factura comercial
  (**la más crítica**), **BL revalidado**, el certificado que aplique y el
  **aviso automático**.
- **Sucursales en SIGAC 3.0.** El sistema gestiona y visualiza operaciones por
  sucursal (Altamira, Veracruz…), con acceso por áreas de responsabilidad.
- **Captura de la operación:** fechas, datos del contenedor, peso, número de BL
  House; los campos no requeridos por el cliente se omiten. Hoy **no manejan
  materiales peligrosos** (esa sección no se usa) y la información del **recinto
  fiscalizado queda pendiente** hasta obtener el registro.
- **Vinculación por TXT.** Cargar la factura mediante el archivo TXT exige un
  **paso adicional de vinculación manual** a la referencia; a diferencia de la
  carga manual, el sistema no vincula automáticamente.
- **Edición de factura:** proveedor (GKN), forma de facturación, moneda y país
  de origen; algunos datos se pre-cargan, otros (como el proveedor) se ingresan
  a mano. Para autopartes GKN se registra la **marca "GKN"** sin modelos ni
  números de serie (marca registrada que se refleja en la partida del pedimento).
- **Incrementables** (flete, seguro): se capturan manualmente si la factura los
  incluye, ajustando montos para que la factura cuadre.
- **Rectificaciones y duplicidad de factura.** Al rectificar hay que **duplicar
  la factura** en el sistema para visualizarla y **generar el COVE sin
  referencia**, porque el pedimento original **pierde la referencia** al
  rectificarse. La pestaña **"Validar CFDI"** se habilita para consultar/generar
  COVEs sin referencia durante la rectificación (queda en blanco en operación
  normal). Caso típico: un contenedor **retenido por la aduana** debe separarse
  de la operación original y despacharse con un **nuevo pedimento**, y duplicar
  la factura evita el conflicto de "factura ya capturada". Carlos: la
  duplicación **sirve como evidencia documental histórica** — no se modifica el
  pedimento original, se crea una nueva ruta documental. Advertencia (Rocio):
  las rectificaciones tienen un **costo significativo**, sobre todo cuando
  derivan de errores de captura.
- **Módulo de tráfico:** aplicación del régimen, decrementables y búsqueda de
  **reglas octavas** (p. ej. la **7476** para dampers) para la valoración
  aduanera y el pago de impuestos.
- **Tipo de cambio y arribo.** Hay que confirmar la **fecha real de arribo** del
  buque (vía **Asipona** o el forwarder) para aplicar el TC del día de entrada
  al país / día de fondeo y **no dejar el tipo de cambio en cero**.
- **Monitoreo de buques.** Se usa SIGAC + el módulo **Asipona** para arribos y
  zarpes; la sección de gestión de buques muestra programación, fechas de arribo
  y estatus de la carga, que provienen de las **manifestaciones de las navieras
  a la aduana** (ej. buque *Manzanillo Express*).
- **Manifestación de valor ≠ manifestación de la naviera.** La naviera notifica
  a la autoridad la llegada del buque y el estatus de la carga (comunicación
  naviera–aduana); la MV es un proceso distinto del ejecutivo.
- **Documentos y COVE** se pueden generar teniendo la guía y la digitalización;
  **no** requiere obligatoriamente la fecha del tipo de cambio, lo que permite
  avanzar en lo administrativo. Tras la llegada y verificación el flujo sigue con
  MV → glosa → validación → pago.
- **Post-pago:** liberación → generación del **DODA** → espera a que el
  transporte recoja el contenedor en la terminal → cruce en la garita.
- **"Shipper" (ambigüedad de término).** Para GKN, "shipper" = un **número de
  referencia** de la operación. Ángel aclara que en logística terrestre
  "shipper" suele ser el **remitente/exportador**, o la **declaración
  electrónica obligatoria (Electronic Export Information / EEI)** que exige EE.UU.
  para exportaciones.
- **Marítimo vs terrestre.** El flujo de importación marítima es muy similar al
  terrestre; la principal diferencia es **cómo se recibe y monitorea la llegada**
  (arribo del buque según la **ETA**). Requieren que el buque llegue según la ETA
  notificada para poder despachar.

## Tareas
> Detectadas, no creadas.
- (Rocio) **Comparar campos y requerimientos** de la operación terrestre vs la
  marítima para determinar las funcionalidades necesarias en SIGAC 3.
- (Rocio) Contactar a Ángel para **detallar el procedimiento de rectificación**
  cuando surja un caso real.
- (Ángel) **Validar el flujo marítimo** con la información obtenida y aclarar
  dudas sobre rectificaciones.

## Temas descartados por irrelevantes
- Pie de página y encuesta de calidad generados por Gemini.
