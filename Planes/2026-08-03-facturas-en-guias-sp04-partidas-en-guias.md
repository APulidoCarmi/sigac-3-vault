# Plan: Front — Gestión de partidas para facturas creadas desde Guías (sp04 de facturas en Guías)

> Sub-plan hijo de [[2026-08-01-facturas-en-guias]]. Repo: `carmi-digital`
> (front) y `carmi-odin-api-v2` (back, ajuste menor de include).
> Depende de: sub-plan 02 (ya completo — modal `GuiaInvoicesModal.tsx` +
> `InvoiceFormModal` con `guiaId`).

## Contexto

Origen: pedido directo del usuario (sesión 2026-08-02/03), al probar el
flujo de crear factura desde Guías: "no veo como agregar partidas a la
factura". Investigación de código (misma sesión, agentes exploradores
sobre `carmi-digital`) encontró que **esto no es una regresión de este
sub-plan ni del sp02** — es una discrepancia entre el plan sp02 (decisión
3: "se reutiliza InvoiceFormModal completo — partidas, montos...") y el
código real: `InvoiceFormModal.tsx` **nunca tuvo** una sección de partidas,
ni siquiera en el flujo original de Referencias/DGO.

El comportamiento real hoy en Referencias/DGO: se crea la factura primero
(sin partidas) y **después**, desde una pestaña separada del detalle de la
Referencia — `ReferenceMerchandiseTab.tsx`
(`app/(customerPortal)/references/components/tabs/`) — se agregan las
partidas vía `AddPartidaDrawer.tsx` → `AddPartidaForm.tsx`
(`app/(customerPortal)/references/components/invoices/`). Ese tab está
scopeado a `referenceId` (dos queries: `GET /references/:id/invoice-items`
para partidas ya creadas + `GET /dgo/reference/:referenceId` para no dejar
huérfanas las facturas sin partidas, merge por `invoiceId`).

El módulo de Guías (`app/(customerPortal)/guias/`) es tabla + modales, sin
pantalla de detalle — no existe ningún tab equivalente ahí, así que tras
crear la factura desde `GuiaInvoicesModal.tsx` no hay dónde ir a agregar
sus partidas.

**Buena noticia confirmada por investigación**: `AddPartidaDrawer.tsx` y
`AddPartidaForm.tsx` **ya son agnósticos** de `referenceId`/`guiaId` — solo
necesitan `invoiceId` (grep de ambos archivos: cero referencias a
`referenceId`/`guiaId`). Toda la atadura a `referenceId` vive en el
componente padre `ReferenceMerchandiseTab.tsx` (fetch, merge, invalidación
de cache, el `<Accordion>` en sí). Eso es lo único que hay que
generalizar.

**Backend — dato ya disponible, con un matiz**: `GET /invoices?guiaId=`
(`invoices.service.ts`, método `findAll`) ya incluye `items` anidados con
`products` (líneas ~520-553) — no hace falta un endpoint nuevo tipo
`/guias/:id/invoice-items`. Sin embargo, el `select`/`include` que sí usa
el flujo de Referencia (`references.service.ts`, método
`getInvoiceItems`, línea 4174) trae muchísimos más campos de enriquecimiento
por partida (UMC/UMT, país de origen, identificadores, reglas 8va, etc.) y
**además incluye `batches`** (`InvoiceItemBatch[]`, necesario para editar
una partida existente vía `AddPartidaForm`'s `initialBatches`) — el include
actual de `GET /invoices?guiaId=` **no trae `batches`** y su selección de
campos de `items`/`products` es más delgada. Hay que auditar y ajustar el
include de la ruta guiaId para tener paridad de campos con lo que el
acordeón compartido necesita mostrar/editar, antes de asumir que "ya viene
todo".

**Aclaración post-investigación (2026-08-03, sesión de dudas antes de
implementar)**: no hace falta paridad total de campos entre ambos
contextos — ver decisión 8. Solo `batches` (necesario para editar, ambos
contextos) e `images` (decisión 9, ambos contextos) requieren ajuste real
del include de `findAll`. Los campos de enriquecimiento tipo
`unitConversions`/UMC-UMT/reglas 8va se quedan exclusivos de
DGO/Referencia por decisión de producto, no por limitación técnica —
aunque de cualquier forma no se pueden traer con garantía en huérfanas de
Guías (`unitConversions` depende de `referenceId`).

También se confirmó (investigación de agente explorador) que `findAll` y
`findOne` de `InvoicesService` **no comparten** un builder de include hoy
— cada uno repite su propio objeto inline. Esto reduce el riesgo original
("tocar `findAll` infla otros endpoints") pero decisión 10 aprovecha para
extraer un builder compartido como mejora de mantenibilidad.

Por último, se confirmó que el bug conocido de `GET /invoices` devolviendo
500 (comentado en `ReferenceMerchandiseTab.tsx` líneas 449-453, commit
`c6bc12e38`, 2026-07-28) **ya tiene fix aplicado** en
`InvoicesService.findAll` (guard defensivo de `page`/`limit` contra `NaN`,
mismo patrón que `GuiasService.findAll`) — pero ese fix vive sin commitear
en el mismo working tree del paraguas (junto con sp02/03). No bloquea este
sub-plan (se continúa sobre la misma rama sin commitear), pero debe ir
commiteado junto con el resto antes de cualquier deploy. El comentario en
`ReferenceMerchandiseTab.tsx` queda obsoleto una vez se commitee — no es
parte del alcance de este sub-plan actualizarlo, pero se deja anotado.

## Decisiones tomadas

1. **Replicar exactamente el comportamiento actual de DGO/Referencia**
   (crear factura primero, agregar partidas después) — no un flujo nuevo
   ni simplificado. El usuario fue explícito: "que funcione como funciona
   ahora en el detalle de referencia en el dgo".
2. **Alcance: solo Guías.** El flujo de Referencias/DGO no cambia de
   comportamiento — se generaliza el mecanismo interno para que ambos lo
   compartan, pero la experiencia en Referencias/DGO debe verse y
   comportarse igual que hoy.
3. **Extraer a un componente compartido, no duplicar.** El usuario fue
   explícito en preferir una solución limpia y reutilizable ("vamos a
   crear el flujo completo para que funcione de forma limpia y con
   mejores prácticas UX/UI") sobre construir un duplicado aislado en
   Guías. Se extrae la lógica de acordeón + handlers
   (`handleOpenAddPartida`/`handleOpenEditPartida`/`handlePartidaSaved`)
   de `ReferenceMerchandiseTab.tsx` a un componente nuevo parametrizable
   (p. ej. `InvoiceMerchandiseAccordion.tsx`, ubicación a definir en la
   fase de implementación — candidato natural:
   `app/(customerPortal)/references/components/invoices/`, ya que
   `AddPartidaDrawer`/`AddPartidaForm` viven ahí y no son
   reference-specific). `ReferenceMerchandiseTab.tsx` pasa a consumirlo
   (mantiene sus dos queries + merge, que sí son específicos de
   Referencia/DGO) en vez de tener el JSX del acordeón inline.
4. **No auto-abrir el formulario de partida tras crear la factura.** A
   diferencia del comportamiento actual de `ReferenceMerchandiseTab`
   (`handleInvoiceCreated` abre automáticamente `AddPartidaDrawer` si
   `mode === "create"`), en Guías el usuario decide cuándo agregar la
   primera partida desde la lista. Esto es una diferencia deliberada
   respecto al comportamiento "igual que DGO" de la decisión 1 — se marca
   explícitamente para que no se asuma paridad total sin querer.
5. **`GuiaInvoicesModal.tsx` reemplaza su tabla plana actual por el mismo
   acordeón que ya usa DGO** (vía el componente compartido de la decisión
   3) — no se mantiene la tabla resumen ni se agrega una fila expandible
   aparte; es consistencia total con la experiencia ya conocida en
   Referencia/DGO.
6. **Fuente de datos: reutilizar `GET /invoices?guiaId=`** (ya incluye
   `items`+`products`) en vez de crear un endpoint nuevo
   `/guias/:id/invoice-items` — solo se ajusta su `include`/`select` de
   items para tener paridad de campos con `getInvoiceItems` (ver Contexto,
   sobre todo `batches`). No hace falta el segundo query "todas las
   facturas del DGO" que usa Referencia para evitar huérfanas — el
   `GET /invoices?guiaId=` ya devuelve directamente todas las facturas de
   esa guía (con o sin partidas), sin necesitar merge con una segunda
   fuente.
7. **Edición de partidas existentes incluida**, con la misma paridad que
   hoy tiene DGO (`editingItem`/`AddPartidaEditingItem`) — no solo alta.
8. **Enriquecimiento de campos condicional por contexto, no por paridad
   total.** El acordeón en modo Guías muestra solo los datos importantes
   de alta de partida (los básicos); los campos de enriquecimiento propios
   de `getInvoiceItems` (UMC/UMT, país de origen, reglas 8va,
   `unitConversions`, etc.) **solo se muestran en modo DGO/Referencia**. Esto
   resuelve de raíz el problema de que `unitConversions` se filtra por
   `referenceId` en el backend (`getInvoiceItems`, línea ~4279-4283) — una
   factura huérfana de Guías (sin `referenceId` todavía) simplemente no
   necesita resolver ese campo porque no se renderiza en ese contexto. El
   componente compartido decide qué mostrar según una prop de
   contexto/modo (p. ej. `context: "guia" | "dgo"`), no según si el dato
   está disponible.
9. **Imágenes a nivel producto también en Guías.** A diferencia de la
   decisión 8, esto sí se generaliza: el ajuste de include del paso de
   backend agrega `images` (con URL firmada) al `include` de `findAll`
   para tener paridad completa con `getInvoiceItems` en este campo
   específico — no es exclusivo de DGO/Referencia.
10. **Include compartido entre `findAll` y `findOne`.** Investigación
    confirmó que ambos métodos de `InvoicesService` repiten hoy su propio
    objeto `include` inline (no hay builder común) — se aprovecha este
    sub-plan para extraer un builder/constante compartida de include de
    `items` (con `batches`/`images` incluidos) que ambos métodos consuman,
    en vez de mantener duplicado a mano y arriesgar que uno de los dos se
    quede desactualizado.

## Fuera de alcance

- Cambiar el comportamiento visual o de datos del flujo de
  Referencias/DGO más allá del refactor interno (mismo aspecto, mismo
  comportamiento, solo cambia qué componente renderiza el acordeón por
  dentro).
- Auto-apertura del formulario de partida tras crear factura en Guías
  (decisión 4 — explícitamente no).
- Migración de datos: facturas ya creadas desde Guías sin partidas no
  necesitan ningún fix retroactivo — en cuanto se abra `GuiaInvoicesModal`
  con el nuevo acordeón podrán agregarse partidas normalmente (el
  mecanismo es por `invoiceId`, agnóstico de cuándo se creó la factura).
- El problema, ya documentado aparte, de que la factura de Guías no
  aparezca en el Expediente Aduanero del DGO hasta que exista Referencia
  (tarea 2/3 del paraguas, fuera de alcance de este sub-plan también).
- Bultos/lotes (`batches`) más allá de exponerlos con paridad de campos —
  no se rediseña esa UI, se hereda tal cual la usa hoy `AddPartidaForm`.

## Pasos

- [x] **Backend** (`carmi-odin-api-v2`): extraer un builder/constante
      compartida de include de `items` (decisión 10) que agregue `batches`
      e `images` (con URL firmada) — los dos únicos gaps reales frente a
      `getInvoiceItems` que aplican a ambos contextos (decisiones 8 y 9) —
      y hacer que tanto `InvoicesService.findAll` (usado por
      `GET /invoices?guiaId=`) como `findOne` consuman ese builder en vez
      de repetir el `include` inline. NO portar los campos de
      enriquecimiento exclusivos de DGO/Referencia (`unitConversions`,
      UMC/UMT, reglas 8va, etc. — decisión 8, esos se quedan resueltos solo
      por `getInvoiceItems`). Sin migración de schema — solo ajuste de
      query. Actualizar/agregar specs de `invoices.service.spec.ts` para la
      nueva forma del include (incluyendo el builder compartido y su uso en
      ambos métodos).
- [x] **Frontend — extracción**: crear `InvoiceMerchandiseAccordion.tsx`
      (componente nuevo, ubicación sugerida
      `app/(customerPortal)/references/components/invoices/`) que reciba
      como prop la lista de facturas-con-items ya resuelta (sin acoplarse
      a `referenceId` ni `guiaId`), `regimenType`/`isGomasClient`, un
      callback `onPartidaSaved` (el caller decide qué invalidar), y una
      nueva prop de contexto (p. ej. `context: "guia" | "dgo"`, decisión 8)
      que controla si se muestran los campos de enriquecimiento exclusivos
      de DGO/Referencia (UMC/UMT, país de origen, reglas 8va,
      `unitConversions`) o solo los básicos de alta de partida (modo
      Guías). Encapsula: el `<Accordion type="multiple">` por factura, los
      botones "Agregar partida"/editar por partida, el estado local del
      `AddPartidaDrawer` (open/editingItem/invoiceId activo), y su
      renderizado.
- [x] **Frontend — refactor de `ReferenceMerchandiseTab.tsx`**: reemplazar
      el JSX del acordeón + sus handlers locales
      (`handleOpenAddPartida`/`handleOpenEditPartida`/`handlePartidaSaved`)
      por el consumo de `InvoiceMerchandiseAccordion`, pasándole
      `groupedInvoices` (ya calculado ahí) y conectando `onPartidaSaved` a
      la invalidación existente de `["reference-merchandise", referenceId]`.
      Mantiene intactas sus dos queries (`reference-merchandise` +
      `dgos/reference`) y el merge — eso sigue siendo específico de
      Referencia. Regresión: probar que Referencias/DGO se ve y comporta
      exactamente igual que hoy (alta, edición, expandir/colapsar).
- [x] **Frontend — tipos/hook de Guías**: extender `InvoiceSummary` en
      `lib/api/modules/invoices.ts` para incluir `items` (con la forma ya
      ajustada en el paso de backend) y confirmar que
      `hooks/use-guia-invoices.ts`/`invoicesService.getByGuiaId` los
      exponen sin cambios adicionales (el include ya viene del backend).
- [x] **Frontend — `GuiaInvoicesModal.tsx`**: reemplazar la tabla plana de
      facturas actual por `InvoiceMerchandiseAccordion`, alimentado por
      `useGuiaInvoices(guiaId)`, con `onPartidaSaved` invalidando
      `["guia-invoices", guiaId, ...]`. Conserva el botón "Agregar
      factura" existente (abre `InvoiceFormModal` con `guiaId`, sin
      cambios) — el acordeón sustituye solo la lista/visualización de
      facturas ya creadas. No se auto-abre el formulario de partida tras
      crear factura (decisión 4).

## Riesgos y side effects a vigilar

- `ReferenceMerchandiseTab.tsx` es un componente en desarrollo activo pero
  ya con lógica no trivial (dos queries + merge + acordeón grande) — el
  refactor para extraer el acordeón debe preservar el comportamiento
  exacto (edición, orden, badges de estado) para no introducir una
  regresión silenciosa en el flujo de Referencias/DGO.
- El ajuste al `include` de `GET /invoices?guiaId=` en el backend también
  afecta el payload que consume `findOne`/otros usos de `InvoicesService`
  si comparten el mismo método de construcción de include — confirmar que
  el ajuste es local a `findAll` y no infla innecesariamente otros
  endpoints.
- Sin el auto-open (decisión 4), es más fácil que una factura de Guías
  quede sin partidas por descuido del usuario — considerar un indicador
  visual (badge "sin partidas") en el acordeón o en la fila, aunque no es
  requisito de este plan.

## Criterios de verificación

- `/verify`: lint/typecheck de front y back; specs de backend ajustados.
- Playwright, flujo Guías: crear factura nueva desde `GuiaInvoicesModal`,
  confirmar que aparece en el acordeón sin partidas, agregar una partida
  (alta), confirmar que se refleja sin recargar. Editar esa partida y
  confirmar que los valores persisten.
- Playwright, regresión Referencias/DGO: confirmar que
  `ReferenceMerchandiseTab` se ve y comporta exactamente igual que antes
  del refactor (alta y edición de partida, expandir/colapsar acordeón, sin
  auto-open roto).

## Ajuste post-implementación (2026-08-03, feedback del usuario tras revisar el diff)

- **Columnas de la tabla de partidas en modo Guías**: se reemplazó el enfoque
  original de "subconjunto de columnas de DGO" (decisión 8) por una tabla
  propia y más reducida, exclusiva de `context="guia"`: No. Parte, País de
  origen, Cantidad (+UMC), Precio Unitario, Precio Total, Acciones. La tabla
  de DGO se dejó intacta (columnas sin condicionales, igual que antes del
  ajuste) — `InvoiceMerchandiseAccordion.tsx` ahora bifurca completamente el
  `<Table>` por `context` en vez de ocultar columnas parciales sobre la misma
  tabla.
- **Drawer de listado de facturas de Guías más ancho**: `GuiaInvoicesModal.tsx`
  pasó de `sm:max-w-2xl` a `sm:max-w-5xl` (mismo ancho que `AddPartidaDrawer`).
