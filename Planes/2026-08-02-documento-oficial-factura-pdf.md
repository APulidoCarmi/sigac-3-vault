# Plan: Documento oficial de la factura (PDF/imagen) separado de las imágenes de producto

> Sub-plan 03 del paraguas [[2026-08-01-facturas-en-guias]]. **Ramas de
> trabajo (decisión 2026-08-02): no crear ramas nuevas** — se trabaja sobre
> las mismas ramas ya activas de los sub-planes 01/02 del paraguas, sin
> commitear todavía ninguno de los 3:
> - `carmi-digital`: `feat/facturas-en-guias`
> - `carmi-odin-api-v2`: `feat/facturas-en-guias-sp01-backend-guiaid-invoice`

## Contexto

Origen: pedido directo del usuario (sesión 2026-08-02), durante la revisión
del diff del sub-plan [[2026-08-01-facturas-en-guias-sp02-front-formulario-facturas]]
(paraguas [[2026-08-01-facturas-en-guias]]). Al probar el modal nuevo
"Facturas de la guía", el usuario reportó: *"el botón de imágenes es para
subir imágenes de los productos de la factura, no para subir imágenes del
documento como tal, deberíamos tener un botón diferente para subir el
documento de la factura ya sea como pdf o como imagen."*

Investigación de código (sesión 2026-08-02, agentes exploradores sobre
`carmi-digital` y `carmi-odin-api-v2`):

- `InvoiceFormModal.tsx` (`app/(customerPortal)/references/components/invoices/`)
  hoy tiene **dos** botones de imágenes, ambos para **fotos de producto**, en
  dos granularidades: a nivel factura (`category: 'ATTACHMENT'`, cuando el
  cliente manda fotos sin decir a qué producto pertenecen) y a nivel
  partida/producto (`category: 'PRODUCT'`, cuando sí lo dice). Ninguno de
  los dos representa el documento/comprobante oficial de la factura física.
- `InvoiceImageDrawer.tsx` (componente reusado por ambos botones) solo
  acepta `image/*` (`accept="image/*"`, línea 220) y comprime todo a JPEG
  vía canvas — no soporta PDF.
- El modelo `InvoiceImage` (Prisma, `carmi-odin-api-v2`) ya tiene
  `documentId` → `Document` (GCS real, no bytea — el campo legacy
  `imageData: Bytes?` no se usa en el flujo actual) y un `category` de tipo
  string libre cuyo enum documentado en el DTO **ya incluye `'MAIN'`**
  (`invoice-image.dto.ts`), sin uso activo hoy en el frontend. Es el
  candidato natural para "documento oficial de factura" — no hace falta
  migración de schema.
- El único ajuste de backend necesario es que `processInvoiceImages`/
  `processInvoiceItemImages` (`invoices.service.ts` líneas ~60 y ~115)
  decodifican el base64 con un regex que solo reconoce el prefijo
  `data:image/...;base64,` — si llega un PDF (`data:application/pdf;base64,`)
  el buffer queda corrupto (el prefijo no se limpia). El `mimeType` en sí ya
  se pasa tal cual (sin validación de formato en el DTO), así que el fix es
  puntual.
- El mecanismo de actualización ya existente (`syncInvoiceImages`,
  `invoices.service.ts` línea 162) trata `dto.images` como "estado final
  deseado": lo que no se reenvía queda con **soft-delete** (`deletedAt`,
  nunca se borra la fila ni el archivo en GCS). Esto da "reemplazar pero
  mantener histórico" gratis, sin lógica nueva de borrado — solo hay que
  construir la UI para mostrar ese histórico (no existe hoy ninguna
  consulta que devuelva las versiones soft-deleted).
- Ya existen en `carmi-digital` patrones maduros de subida de PDF+imagen sin
  compresión (`features/appointments/components/upload/DocumentUpload.tsx`,
  `BolDocumentUpload.tsx`) y de previsualización de PDF vía `<iframe>`
  (`components/movements/movement-form/tab-documentos.tsx` línea 1047,
  `components/document-reader/pdf-viewer.tsx`) — se toman como referencia de
  estilo, aunque no se reutilizan directamente (ver decisión 3).

### Ronda 2 de investigación (sesión 2026-08-02, tras revisión previa a `/implementa`)

Antes de dar el plan por listo, se investigó a fondo con agentes
exploradores 3 dudas de diseño abiertas. Correcciones y hallazgos nuevos
sobre lo anterior:

- **`InvoiceImage.uploadedBy` sí existe** (`Uuid`, ya se puebla en
  `invoices.service.ts:93`), pero **sin relación Prisma a un modelo
  `User`** — igual que `Reference.createdBy`. El patrón ya usado en el
  listado de referencias para mostrar "Creado por" con nombre legible
  (`ReferenceClassicTable.tsx`) **no es un join**: son columnas
  denormalizadas `createdByName`/`createdByEmail` en el modelo `Reference`
  (`schema.prisma:3929-3931`), resueltas **una sola vez al crear** vía
  `applyAuthenticatedCreator()` → `fetchUserProfile()`
  (`references.service.ts:308-381`, hace `fetch()` a carmi-db; si falla,
  cae a los claims del propio token sin bloquear la creación). Se replica
  el mismo patrón para `InvoiceImage` (ver decisión 8) — **esto sí requiere
  una migración de Prisma** (2 columnas nuevas), corrigiendo la decisión 5
  original que asumía cero cambios de schema.
- **`InvoiceImage.documentId → Document` con GCS real ya está en
  producción para facturas**, contradiciendo la premisa inicial de que hoy
  solo se usa `imageData: Bytes?` (legacy): `processInvoiceImages`/
  `processInvoiceItemImages` (`invoices.service.ts:47-155`, comentario
  `SPEC-INVOICE-IMAGES-GCP`) ya suben a GCS vía
  `storageService.uploadFile(...)` y crean el `Document` + `InvoiceImage`
  con `documentId` real. El base64 que manda el frontend hoy es solo el
  **transporte** DTO→backend, no el almacenamiento final.
- **Ya existe un endpoint genérico de subida directa**: `POST
  /storage/upload` (`storage.controller.ts:55-127`) sube un archivo a GCS y
  devuelve el `Document` creado (con su `id`) de forma independiente a
  cualquier entidad padre — es el mismo patrón que ya usa
  `DocumentUpload.tsx` hoy (vía `storageService.uploadFile()` en
  `lib/services/storageService.ts`). Se usa este endpoint para el documento
  de factura en vez de embeber base64 (ver decisión 9).
- **`OnboardingDocumentCardNew.tsx`** (patrón maduro de historial de
  versiones con tabs/purga, usado en `onboarding-new/[token]/page.tsx` y
  `CompanyOnboardingDocumentsTab(New).tsx`) se evaluó como base para el
  paso de "Ver histórico" y **se descarta**: está fuertemente acoplado al
  dominio de onboarding de compañías (tipo `CompanyOnboardingDocument`,
  `companyId` obligatorio, modo público/admin, catálogo, validar/rechazar/
  comentar) — adaptarlo equivaldría a reescribirlo. Se mantiene el
  componente de histórico simple ya previsto originalmente (ver decisión
  10).

### Ronda 3 de investigación (sesión 2026-08-02, verificación previa a implementar)

Antes de cerrar el plan, se verificaron con agentes exploradores todas las
afirmaciones de las rondas 1 y 2 contra el estado actual del código, y se
resolvieron dos dudas abiertas del usuario. Hallazgos:

- **Existen DOS archivos `InvoiceFormModal.tsx`** en `carmi-digital`: el que
  describe este plan (`app/(customerPortal)/references/components/invoices/
  InvoiceFormModal.tsx`, con la lógica de categorías `ATTACHMENT`/`PRODUCT`)
  y otro, no relacionado, del wizard viejo de creación de referencia
  (`.../createReference/.../step3Facturas/components/InvoiceFormModal.tsx`,
  sin esa lógica). **Se confirmó cuál es el real**: el drawer de DGO
  (`ReferenceDGOTab.tsx` línea 45, renderizado dentro de `DgoActionsDrawer`
  como `Sheet` lateral, no modal centrado) importa y usa el primero. El
  segundo archivo no se toca — no es parte de ningún flujo vigente de los
  3 (Guías/DGO/Referencia). La decisión 4 del plan (componente único
  compartido) se confirma correcta; solo se corrige la premisa visual: la
  superficie real donde vive el botón nuevo es un **Sheet/drawer lateral**
  en el caso DGO, no un modal centrado — el diseño del botón "Documento de
  factura" debe funcionar bien en ese layout más angosto.
- **`processInvoiceImages`/`processInvoiceItemImages`/`syncInvoiceImages`
  están duplicados en TRES servicios de `carmi-odin-api-v2`, no solo en
  `invoices.service.ts`**, y las tres copias están vivas (ninguna es código
  muerto), cada una disparada por una ruta distinta:
  - `invoices.service.ts` → `POST /invoices`, `PATCH /invoices/:id`.
  - `references.service.ts` → `PATCH /references/:id` cuando el body trae
    `dto.invoices` anidado (vía `createInvoiceForReference`/`updateInvoice`
    internos) — **este es el camino real que dispara el drawer de DGO**.
  - `invoice-items.service.ts` → `POST /invoice-items`,
    `PATCH /invoice-items/:id` (solo nivel partida).

  Si el fix del regex base64 y el soporte de `documentId`/`MAIN` solo se
  aplican en `invoices.service.ts` (como asumía la redacción original del
  plan), el flujo de DGO —que pasa por `references.service.ts`— quedaría
  con el bug del PDF corrupto y sin soporte del documento oficial. Se
  decide (ver decisión 11) extraer un servicio compartido en vez de
  triplicar el fix.

**Respuestas a las dudas planteadas al usuario:**
1. El drawer de DGO usa el `InvoiceFormModal.tsx` correcto (el de
   `references/components/invoices/`) — confirmado, sin cambios de alcance
   más allá de la aclaración visual (Sheet, no modal).
2. El código de `references.service.ts` no es descartable ni código
   muerto: es el camino real usado por DGO. El usuario pidió la mejor
   práctica para mantener el código limpio y escalable → se opta por
   extraer un servicio compartido en vez de duplicar el fix (ver decisión
   11 y nuevo paso de refactor).
3. El endpoint de histórico se deja sin controles de permisos adicionales,
   usando el mismo nivel de acceso ya implementado para ver la factura
   (decisión del usuario, sin cambios).

## Decisiones tomadas

1. **Solo un documento "vigente" por factura**, pero al reemplazarlo el
   anterior se conserva como histórico (no se borra). Esto ya lo da gratis
   el soft-delete existente de `syncInvoiceImages` — no requiere lógica de
   borrado nueva.
2. **Reemplazo automático, sin diálogo de confirmación extra**: subir un
   documento nuevo reemplaza el vigente de inmediato en la UI (no acumula,
   no pregunta "¿seguro?").
3. **El documento es opcional** para crear/guardar la factura (igual que
   las imágenes hoy) — no bloquea el botón Crear/Guardar Factura.
4. **Aplica a los 3 flujos que comparten `InvoiceFormModal`** (Guías, DGO,
   Referencia) de una vez, no solo a Guías — es el mismo componente
   compartido en producción.
5. **Se reutiliza el modelo existente** (`InvoiceImage.category = 'MAIN'`,
   `documentId` → `Document` en GCS, ya en producción para facturas — ver
   ronda 2). El regex de decodificación base64 se generaliza igual
   (`image/\w+` → cualquier mimetype) como fix defensivo para el resto de
   categorías (`ATTACHMENT`/`PRODUCT`), aunque el documento `MAIN` en sí no
   pasa por ese camino (ver decisión 9).
6. **Componente nuevo y separado** para la subida del documento
   (`InvoiceDocumentUpload.tsx` o nombre equivalente), en vez de extender
   `InvoiceImageDrawer.tsx` — este último queda intacto (multi-imagen,
   compresión JPEG, sin PDF) para no arriesgar el flujo de fotos de
   producto ya en producción. El componente nuevo es single-file,
   acepta `image/*` + `application/pdf`.
7. **Sí se requiere UI de histórico**: al reemplazar el documento, el
   usuario debe poder ver las versiones anteriores (fecha, quién la subió,
   cuándo se reemplazó, y poder verla/descargarla) — no basta con que quede
   solo en BD.
8. **"Quién subió" vía columnas denormalizadas, no relación Prisma**
   (decisión ronda 2): agregar `uploadedByName`/`uploadedByEmail` a
   `InvoiceImage` vía `npx prisma migrate dev --name
   add-uploaded-by-name-email-to-invoice-image`, resueltas una sola vez al
   subir el documento reusando la misma lógica de `fetchUserProfile()` que
   ya usa `references.service.ts` para `Reference.createdByName`/
   `createdByEmail` — mismo patrón, sin bloquear la subida si el fetch a
   carmi-db falla (cae a los claims del token, igual que hoy en
   referencias).
9. **Subida directa a storage (`documentId`), no base64 embebido**, solo
   para el documento oficial (`category: 'MAIN'`) (decisión ronda 2): el
   frontend sube el archivo de inmediato vía `POST /storage/upload`
   (endpoint genérico ya existente, mismo que usa hoy
   `DocumentUpload.tsx`/`storageService.uploadFile()`) y recibe un
   `documentId`; el DTO de factura acepta ese `documentId` directo para la
   entrada `MAIN` en vez de forzar `imageData` en base64 — evita embeber
   PDFs pesados en el JSON de la factura. Los botones de fotos de producto
   (`ATTACHMENT`/`PRODUCT`) **no cambian**, siguen con el flujo base64
   actual vía `InvoiceImageDrawer`.
10. **Histórico simple, sin reusar `OnboardingDocumentCardNew.tsx`**
    (decisión ronda 2, evaluado y descartado por acoplamiento fuerte al
    dominio de onboarding de compañías): lista con fecha de subida, quién
    la subió (nombre resuelto vía decisión 8), fecha de reemplazo, y
    acción ver/descargar — sin tabs, sin catálogo, sin flujo de
    validar/rechazar/comentar/purgar.
11. **Extraer un servicio compartido `InvoiceImagesService`** (decisión
    ronda 3, a pedido explícito del usuario de priorizar código limpio y
    escalable sobre un fix puntual): en vez de aplicar el fix del regex
    base64 y el soporte de `documentId`/`MAIN` por separado en
    `invoices.service.ts`, `references.service.ts` e
    `invoice-items.service.ts` (las tres copias actuales de
    `processInvoiceImages`/`processInvoiceItemImages`/`syncInvoiceImages`,
    las tres vivas y disparadas por rutas distintas — ver ronda 3), se
    extraen estos métodos a un servicio nuevo inyectable, y los tres
    servicios existentes lo consumen en vez de mantener su propia copia.
    Esto asegura que el flujo de DGO (que pasa por `references.service.ts`)
    también soporte el documento oficial y el fix de PDF, sin triplicar el
    cambio ni arriesgar que quede desincronizado a futuro.

## Fuera de alcance

- No se reclasifican imágenes `ATTACHMENT` ya existentes como "documento de
  factura" — el cambio aplica solo hacia adelante, a partir de facturas
  nuevas o ediciones donde el usuario suba el documento explícitamente por
  el botón nuevo.
- No se toca `InvoiceImageDrawer.tsx` ni el flujo de fotos de producto
  (a nivel factura ni a nivel partida) más allá del fix defensivo del mismo
  regex en `processInvoiceItemImages` (mismo bug, mismo archivo, cero
  riesgo, no cambia comportamiento de imágenes).
- No resuelve la Tarea 3 del paraguas [[2026-08-01-facturas-en-guias]]
  (que el documento aparezca en Expediente Aduanero al crear la
  Referencia) — es un problema distinto, ya documentado ahí como fuera de
  alcance, depende de la Tarea 2 (movimiento/expediente por guía).
- No agrega restricción de tamaño/tipo más allá de lo que ya use el patrón
  de referencia (`DocumentUpload.tsx`: ~10MB por archivo) — no se pide
  límite especial para PDFs multipágina grandes.
- No se pagina el historial de versiones (se asume bajo volumen — pocas
  rectificaciones por factura).

## Pasos

- [x] **Backend** (`carmi-odin-api-v2`): migración Prisma
      `npx prisma migrate dev --name add-uploaded-by-name-email-to-invoice-image`
      agregando `uploadedByName String? @db.VarChar(255)` y
      `uploadedByEmail String? @db.VarChar(255)` a `InvoiceImage`. Al subir
      un `InvoiceImage` (cualquier categoría, no solo `MAIN`), resolver y
      persistir estos campos reusando la misma lógica de
      `fetchUserProfile()`/`applyAuthenticatedCreator()` que ya usa
      `references.service.ts` para `Reference.createdByName`/
      `createdByEmail` (mismo fallback a claims del token si falla el fetch
      a carmi-db).
- [x] **Backend — refactor previo (decisión 11)**: extraer
      `processInvoiceImages`/`processInvoiceItemImages`/`syncInvoiceImages`
      a un servicio nuevo compartido (p. ej. `InvoiceImagesService`),
      inyectado en `invoices.service.ts`, `references.service.ts`
      (camino real del drawer de DGO vía `dto.invoices` anidado en
      `PATCH /references/:id`) e `invoice-items.service.ts` — las tres
      copias actuales quedan reemplazadas por el uso del servicio
      compartido. Este paso va **antes** de los dos siguientes para que el
      fix del regex y el soporte de `documentId` se apliquen una sola vez
      y cubran los 3 flujos (Guías/DGO/Referencia), no solo
      `POST /invoices`. Actualizar/agregar specs (`invoice-images.service.spec.ts`
      nuevo + ajustar los specs existentes de los tres servicios
      consumidores, dado que el pre-commit exige que todo método público
      esté referenciado en su spec).
- [x] **Backend**: en el servicio compartido, generalizar el regex de
      decodificación de `imageData` de `/^data:image\/\w+;base64,/` a
      `/^data:[^;]+;base64,/` — fix defensivo para las categorías
      `ATTACHMENT`/`PRODUCT` que siguen con el flujo base64 (no bloquea el
      documento `MAIN`, que usa `documentId` directo). Cubre automáticamente
      los 3 flujos al vivir en el servicio compartido.
- [x] **Backend**: en el servicio compartido, permitir que el DTO de
      creación/edición de `InvoiceImage` acepte `documentId` directo
      (documento ya subido vía `POST /storage/upload`) como alternativa a
      `imageData` base64, para la entrada `category: 'MAIN'`. Ajustar
      `processInvoiceImages`/`syncInvoiceImages` para no re-subir a GCS
      cuando ya llega `documentId` (solo vincular el `Document` existente
      al `InvoiceImage` nuevo). Al vivir en el servicio compartido, aplica
      igual si el documento se sube desde Guías, DGO o Referencia.
- [x] **Backend**: nuevo endpoint de solo lectura para el histórico del
      documento de factura — p. ej. `GET /invoices/:id/document-history`,
      devolviendo los `InvoiceImage` con `category: 'MAIN'`
      (incluyendo `deletedAt` no nulo) ordenados por fecha desc, con
      metadata del `Document` relacionado (fileName, mimeType, fileSize,
      signed URL) y `uploadedByName`/`uploadedByEmail`. Se consulta
      on-demand (no se agrega al `findOne` general de factura, que no debe
      cargar este peso en cada fetch). Mismo nivel de acceso/permisos ya
      implementado para ver la factura — sin controles adicionales
      (decisión del usuario, sesión 2026-08-02).
- [x] **Frontend**: crear `InvoiceDocumentUpload.tsx` (junto a
      `InvoiceImageDrawer.tsx`, en
      `references/components/invoices/`) — subida single-file,
      `accept: ['image/*', 'application/pdf']`, **subida directa**: al
      seleccionar el archivo, llama `POST /storage/upload` (vía
      `storageService.uploadFile()`, mismo patrón que
      `DocumentUpload.tsx`) y guarda el `documentId` devuelto en
      `formData.images` para la entrada `MAIN` — no usa
      `FileReader.readAsDataURL` ni compresión JPEG (eso queda solo para
      `InvoiceImageDrawer`). Siempre reemplaza la entrada local (nunca
      acumula): si ya hay un documento cargado, la UI muestra su
      nombre/tipo/preview con acciones "Reemplazar" y "Ver histórico", no
      un dropzone aditivo. Confirmado (ronda 3): el archivo a tocar es
      `references/components/invoices/InvoiceFormModal.tsx` — el drawer de
      DGO (`ReferenceDGOTab.tsx`, dentro de `DgoActionsDrawer`) lo renderiza
      como `Sheet` lateral, no como modal centrado; el diseño del nuevo
      botón/estado debe caber bien en ese layout más angosto. El otro
      archivo homónimo del wizard viejo de creación de referencia
      (`createReference/.../step3Facturas/`) no se toca — no está en el
      camino de ningún flujo vigente.
- [x] **Frontend**: en `InvoiceFormModal.tsx`, derivar
      `documentoFactura = formData.images.find(img => img.category === 'MAIN')`
      y ajustar `invoiceImages` (el pool que hoy alimenta el botón
      "Imágenes" existente) para **excluir** la entrada `MAIN`
      (`formData.images.filter(img => img.category !== 'MAIN')`) — el botón
      de fotos de producto a nivel factura sigue igual, solo deja de
      mezclarse con el documento. Agregar botón nuevo "Documento de
      factura" en el header (junto al de "Imágenes"), que abre
      `InvoiceDocumentUpload`. Guardar/reemplazar actualiza `formData.images`
      quitando la `MAIN` vieja y agregando la nueva — `buildInvoiceDto` no
      necesita cambios (ya mapea `category` tal cual por imagen).
- [x] **Frontend**: UI de histórico dentro de `InvoiceDocumentUpload` (o un
      diálogo aparte que abra desde ahí) — componente simple nuevo (no se
      reutiliza `OnboardingDocumentCardNew.tsx`, descartado por
      acoplamiento al dominio de onboarding de compañías, ver ronda 2):
      lista las versiones anteriores (fecha de subida, quién la subió,
      fecha de reemplazo) usando el endpoint nuevo, con acción para
      ver/descargar cada versión. Solo aplica en modo `edit` con
      `invoiceId` real (en modo `create` todavía no hay historial
      posible).
- [x] **Frontend**: preview del documento vigente — si `mimeType ===
      'application/pdf'`, usar el patrón `<iframe src={signedUrl}>` ya
      usado en `components/movements/movement-form/tab-documentos.tsx`
      (o extraer/reusar `components/document-reader/pdf-viewer.tsx` si
      encaja mejor); si es imagen, thumbnail simple como ya hace
      `InvoiceImageDrawer`.

## Riesgos y side effects a vigilar

- `InvoiceFormModal.tsx` es el componente de factura más compartido en
  producción (Guías, DGO vía `ReferenceDGOTab`, Referencia vía
  `ReferenceMerchandiseTab`) — cualquier cambio a `formData.images`/
  `buildInvoiceDto` debe probarse en los 3 flujos, no solo en el nuevo.
- Facturas ya existentes no tienen ninguna imagen con `category: 'MAIN'` —
  el nuevo botón/estado debe manejar bien "sin documento todavía" sin
  romper con facturas viejas (no asumir que siempre existe una).
- Antes de tocar el regex, grep por otros consumidores de
  `processInvoiceImages`/`syncInvoiceImages`/`category` a nivel factura por
  si algún reporte o vista asume `'ATTACHMENT'` como único valor posible.
- El endpoint de histórico expone `deletedAt`/metadata de documentos
  soft-deleted — confirmar que no hay reglas de permisos/rol distintas para
  ver histórico vs. ver el documento vigente (no se detectó ninguna en esta
  investigación, pero no se auditó a fondo el módulo de permisos).

## Criterios de verificación

- `/verify`: lint/typecheck del front; specs de backend existentes +
  cobertura nueva del caso PDF en `invoices.service.spec.ts`.
- Flujo manual con Playwright MCP (revisando consola), en los 3 flujos
  (Guías, DGO, Referencia):
  1. Crear una factura nueva subiendo un PDF real como "Documento de
     factura" (además de datos normales) — confirmar que se guarda y que
     al reabrir en modo edición el documento sigue ahí con preview.
  2. Reemplazar ese documento por uno nuevo (imagen esta vez) — confirmar
     que el nuevo se muestra como vigente y que el "Ver histórico" muestra
     el PDF anterior con su fecha, sin que aparezca duplicado en la vista
     principal.
  3. Regresión: el botón "Imágenes" (fotos de producto a nivel factura)
     sigue funcionando igual que antes de este cambio — no debe mostrar ni
     mezclar el documento de factura.
  4. Regresión: crear/editar factura sin documento (dejarlo vacío) — debe
     guardar sin error (es opcional).
