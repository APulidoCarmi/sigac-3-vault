# Plan: Ingesta de manifiesto de guías línea por línea (captura manual + bulk)

## Contexto

Tarea 14 del flujo Querétaro (aéreo), registrada en `Tareas.md` bajo
"Tablero de Guías — ingesta y matching", con origen citado en
[[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]. **Nota:
ese archivo de reunión no existe todavía en `Reuniones/`** — la
transcripción sigue pendiente de depurar vía `/reunion` (ver memoria
`reunion-transcripciones-grandes-pendientes`); el wikilink se mantiene por
convención de `Tareas.md`, pero hoy es un enlace roto. Contexto adicional
confirmado en [[2026-07-20 - Recap visita Querétaro]], sección "Campos
reales del manifiesto de carga (ejemplo: DH Express México)".

La tarea 13 (tablero de guías pre-referencia) ya está implementada y creó
un modelo Prisma propio `Guia` (`carmi-odin-api-v2/prisma/schema.prisma`),
**distinto** de `UnidentifiedWaybill` (que sirve al flujo de reconciliación
de guías llegadas por correo, SP-17, no al de captura manual de Querétaro).
`Guia` hoy tiene: `id`, `companyId` (FK a `Company`, obligatorio),
`destinatario` (texto libre, obligatorio), `guiaMaster`, `guiaHouse`,
`regimen`, `status` (`GuiaStatus`), `referenceId`, `registeredBy`, `notes`.

Front relacionado: `carmi-digital/app/(customerPortal)/guias/page.tsx`
(tablero), `.../guias/components/GuiaFormModal.tsx` (alta manual
individual), `lib/api/modules/guias.ts` (tipos + `guiasService`).

Esta tarea es sobre **capturar el manifiesto de carga real** (que hoy llega
por correo en Excel) con sus campos propios, línea por línea, ampliando el
modelo `Guia` — no crear una tabla nueva ni tocar `UnidentifiedWaybill`.

## Decisiones tomadas

- **Captura únicamente manual.** La automatización de la ingesta del correo
  (mencionada como posible en la reunión del 20-jul) se descarta por ahora;
  no se implementa en este plan. Motivo: decisión explícita del usuario en
  la entrevista, priorizando tener el flujo manual funcionando primero.
- **Se reutiliza y amplía el modelo `Guia` existente** (no se crea tabla
  nueva, no se reusa `UnidentifiedWaybill`). Motivo: `Guia` ya es la tabla
  construida específicamente para el tablero pre-referencia de Querétaro
  (tarea 13); es la responsabilidad correcta.
- **Ningún campo nuevo es obligatorio.** Los campos ya obligatorios hoy
  (`companyId`, `destinatario`) no cambian. Motivo: decisión explícita del
  usuario — el manifiesto real trae datos incompletos/inconsistentes (ver
  riesgo de "Origen" abajo) y no se quiere bloquear la captura.
- **"Destinatario" y "Cliente" son el mismo concepto.** No se introduce un
  campo nuevo para destinatario: el texto crudo capturado (columna
  "Destinatario" del manifiesto) llena el campo `destinatario` que ya
  existe en `Guia`, y se resuelve contra `companyId` (FK a `Company`) en el
  mismo flujo de captura — no se difiere a un estado "sin identificar".
  Motivo: aclaración explícita del usuario; evitar duplicar el concepto de
  cliente.
- **Peso con unidad seleccionable, reusando el patrón visual de
  `WeightSection.tsx`** (`features/classification/components/form/`): un
  `Select` fijo con las opciones `KG` / `LB` / `G` / `TON` (no es un
  catálogo Prisma, es una lista fija hardcodeada en el front). Se replica
  el mismo patrón para `Guia.pesoUnidad`, no se crea una tabla de catálogo
  nueva. Motivo: consistencia con el formulario de números de parte que el
  usuario señaló como referencia.
- **"Número de Vuelo" es un string libre**, sin catálogo de vuelos.
- **Bulk import agrupado por Destinatario (Opción B).** El formulario de
  importación masiva (pegar texto o subir CSV/Excel) agrupa
  automáticamente las filas por texto de `Destinatario` idéntico, y el
  usuario asigna el `companyId` **una vez por grupo** (no una vez por todo
  el bloque, ni una vez por línea). Motivo: en el manifiesto real de
  ejemplo (17 líneas), la mayoría de filas comparten destinatario pero hay
  varios distintos en el mismo bloque — asignar cliente por grupo reduce
  clics sin asumir que todo el bloque es de un solo cliente.
- **Mismo formulario/campos para alta individual y alta masiva.** El modal
  de alta manual (`GuiaFormModal`, ya existente de la tarea 13) se amplía
  con los mismos campos nuevos que usa el bulk import — no son dos
  esquemas distintos.
- **Copiar los datos capturados hacia `AirManifest` al crear la
  Referencia queda fuera de este plan** (es la tarea 21, "Generar/vincular
  AirManifest al crear Referencia", separada en `Tareas.md`). El usuario
  confirmó que sí deben copiarse eventualmente, pero dejó explícita la duda
  de si `AirManifest` es la tabla correcta para recibirlos, dado que hoy
  conviven tres entidades con responsabilidades parecidas (`Guia`,
  `AirManifest`, `UnidentifiedWaybill`). **Antes de implementar esa tarea
  21, detenerse a analizar y decidir la relación definitiva entre las tres
  tablas** — no asumir el mapeo campo-a-campo sin esa revisión.

## Fuera de alcance

- Automatización de la ingesta del correo del manifiesto de carga (queda
  para una iteración futura, no definida).
- Matching automático de `destinatario` contra el catálogo de `Company` vía
  LLM/fuzzy (tarea 15, separada — aquí la asignación de `companyId` es
  manual, por grupo).
- Entidad `Team` y filtrado del tablero por equipo de ejecutivos (tarea 16).
- Copiar los campos capturados hacia `AirManifest` al crear la Referencia
  (tarea 21) — incluyendo el análisis pendiente de la relación entre
  `Guia`, `AirManifest` y `UnidentifiedWaybill`.
- Definir los estatus finitos del tablero de guías (tarea aparte, mencionada
  en la reunión del 20-jul; hoy `GuiaStatus` es un set inicial hipotético).
- Validación/bloqueo de guías duplicadas (mismo `guiaMaster`/`guiaHouse` ya
  cargado) — no se cubre en este plan.
- Moneda del campo `valor`: se asume USD para el parseo inicial (ver
  riesgos); definir soporte multi-moneda es tema aparte si llega a hacer
  falta.

## Pasos

- [ ] **Migración Prisma** (vía CLI, `npx prisma migrate dev --name
  add-manifest-fields-to-guia`) agregando a `Guia` los campos nuevos, todos
  opcionales: `fecha` (DateTime?), `origen` (String?), `descripcionMercancia`
  (String? @db.Text), `peso` (Decimal? @db.Decimal(12,3)), `pesoUnidad`
  (String? @db.VarChar(5)), `piezas` (Int?), `bultos` (Int?), `valor`
  (Decimal? @db.Decimal(14,2)), `domicilioDestinatario` (String? @db.Text),
  `remitente` (String? @db.VarChar(255)), `domicilioRemitente` (String?
  @db.Text), `numeroVuelo` (String? @db.VarChar(20)).
- [ ] **Backend — DTOs y servicio de `Guia`**: actualizar
  `CreateGuiaDto`/`UpdateGuiaDto` y el servicio correspondiente para
  aceptar y persistir los campos nuevos (todos opcionales).
- [ ] **Backend — endpoint de creación masiva**: nuevo endpoint (ej. `POST
  /guias/bulk`) que reciba un arreglo de filas ya resueltas (con
  `companyId` asignado por grupo desde el front) y cree N registros `Guia`
  en una sola transacción Prisma.
- [ ] **Frontend — tipos y servicio**: actualizar `Guia`,
  `CreateGuiaDto`/`UpdateGuiaDto` y `guiasService` en
  `lib/api/modules/guias.ts` con los campos nuevos y el método de creación
  masiva.
- [ ] **Frontend — ampliar `GuiaFormModal`** (alta individual) con los
  campos nuevos: fecha, origen, descripción de mercancía, peso + selector
  de unidad (KG/LB/G/TON, mismo patrón visual de `WeightSection.tsx`),
  piezas, bultos, valor, domicilio destinatario, remitente, domicilio
  remitente, número de vuelo — todos opcionales.
- [ ] **Frontend — parser de bulk import**: función que reciba texto
  pegado (TSV, tal como Excel copia filas) o un archivo CSV/Excel, detecte
  el header esperado (Fecha, Guía Master, Guía House, Origen, Descripción
  de la Mercancía, Peso, Piezas, Bulto, Valor, Destinatario, Domicilio
  Destinatario, Remitente, Domicilio Remitente, Número de Vuelo) y mapee
  cada columna a su campo del modelo, parseando `valor` (quitar `$` y
  comas) y `peso`/`piezas`/`bultos` como numéricos.
- [ ] **Frontend — UI de importación masiva**: modal/pantalla nueva con
  textarea para pegar el bloque (o input file), tabla de previsualización
  editable de las filas parseadas, agrupación automática por texto de
  `Destinatario` idéntico, y un selector de `Company` por cada grupo
  distinto antes de confirmar.
- [ ] **Frontend — conectar el botón "Importar guías"** en
  `app/(customerPortal)/guias/page.tsx`, junto al botón ya existente
  "Nueva guía", que abre el modal de importación masiva.

## Riesgos y side effects a vigilar

- **Migración Prisma obligatoria por CLI** (regla global del usuario): no
  escribir SQL de migración a mano bajo ninguna circunstancia.
- **Moneda de `valor` no viene explícita** en el manifiesto de ejemplo
  (solo símbolo `$`); se asume USD para el parseo inicial — confirmar con
  Querétaro si en algún caso real viene en otra moneda antes de dar el
  campo por cerrado.
- **Campo "Origen" con datos dudosos**: en el manifiesto de ejemplo (DH
  Express México, 17 líneas) todas las filas traen literalmente `"0"` como
  origen — probable columna mal poblada en la fuente real. No bloquear la
  captura por esto: se guarda tal cual venga, sin validar formato.
- **Volumen del bulk**: "podrían ser muchas" líneas por manifiesto (sin
  número promedio confirmado) — la tabla de previsualización debe
  soportar scroll sin trabarse; no hay límite de filas por importación
  definido todavía, revisar si hace falta uno al implementar.
- **Pendiente arquitectónico marcado explícitamente por el usuario**: antes
  de tocar la tarea 21 (copiar `Guia` → `AirManifest` al crear Referencia),
  detenerse a analizar la relación real entre `Guia`, `AirManifest` y
  `UnidentifiedWaybill` — hay riesgo de responsabilidades solapadas entre
  las tres tablas.
- **Trazabilidad de origen incompleta**: la nota de reunión del 21-jul que
  Tareas.md cita como origen de esta iniciativa no existe todavía en el
  baúl (transcripción pendiente); si aparece después con detalles
  adicionales, revisar que no contradiga las decisiones de este plan.
- **Duplicados no validados**: cargar el mismo manifiesto dos veces (bulk o
  manual) no se detecta ni bloquea en este plan.

## Criterios de verificación

- La migración corre limpio con `npx prisma migrate dev` (sin edición
  manual de SQL) y el cliente Prisma se regenera sin errores de tipos en
  el resto del backend.
- **Alta manual**: crear una guía desde `GuiaFormModal` con todos los
  campos nuevos poblados (incluyendo peso + unidad) y confirmar que
  persisten y se reflejan correctamente en la tabla de `/guias`.
- **Alta manual con campos vacíos**: confirmar que el submit no se bloquea
  si se dejan sin llenar los campos nuevos (ningún campo nuevo es
  obligatorio).
- **Bulk import**: pegar el bloque de ejemplo real (17 líneas, 3
  destinatarios distintos — Tyco Electronics México, Emuge-Franken, Agco
  México) y confirmar que:
  - las filas se parsean correctamente (peso, piezas, bultos, valor sin
    símbolos, domicilios, remitente, vuelo);
  - se agrupan automáticamente por texto de `Destinatario` idéntico (3
    grupos en el ejemplo);
  - se puede asignar un `companyId` distinto por cada grupo;
  - al confirmar, se crean las 17 guías en el tablero con todos los campos
    correctos y el cliente correcto según su grupo.
- **Playwright** (cambio de front): flujo completo en `/guias` — abrir
  modal de importación masiva, pegar el bloque de ejemplo, verificar
  agrupación visual, asignar cliente a cada grupo, confirmar creación, y
  verificar que las 17 filas aparecen en la tabla del tablero sin errores
  en consola del navegador.
