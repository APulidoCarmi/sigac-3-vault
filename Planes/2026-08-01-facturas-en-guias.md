# Plan paraguas: Formulario de facturas en el módulo de Guías (vinculación directa a guía House) — SIGAC 3

> Plan paraguas: este archivo lleva el contexto y las decisiones de toda la
> iniciativa y el índice de sub-planes hijos (cada uno es un plan normal,
> apto para una sesión limpia de `/implementa`, con sus propios criterios de
> verificación). El paraguas **no se implementa**: se actualiza al cerrar
> cada hijo.

## Contexto

Origen: tarea 1 de [[2026-07-31 - Revisión de Movimientos - Guía]] —
"Guías (front): agregar en el módulo de Guías el formulario de facturas
para poder subir factura (documento + datos) vinculada directamente a la
guía House." Motivo de negocio (de la nota de reunión): en Querétaro no
vinculan factura↔guía al recibir el manifiesto; el tramitador tiene que
buscar manualmente la factura para poder hacer el previo, generando doble
trabajo entre ejecutivo y tramitador.

Investigación previa (sesión del 2026-08-01, agente explorador sobre
`carmi-digital`): el módulo de Guías (`app/(customerPortal)/guias/`) es
tabla + modales, sin pantalla de detalle. El formulario de factura a
reutilizar es
`app/(customerPortal)/references/components/invoices/InvoiceFormModal.tsx`
(1668 líneas), que hoy **exige `referenceId` siempre** y solo admite
`dgoId` opcional — **no existe ningún campo `guiaId`** en el tipo `Factura`
ni en el DTO de invoice del backend (`carmi-odin-api-v2`). El componente de
documento (`InvoiceImageDrawer.tsx`, sube el archivo como base64 embebido
en el DTO) es reutilizable sin cambios.

Punto de diseño central que obligó a partir esto en paraguas: el tablero de
Guías es **pre-referencia** ([[2026-07-22-flujo-queretaro-aereo]]) — una
guía House puede existir antes de que exista cualquier Reference/DGO. Por
tanto la factura subida desde Guías no puede depender de `referenceId`
obligatorio como hoy.

Área de código: back `carmi-odin-api-v2/src/invoices/**` (o el módulo que
corresponda al modelo `Invoice`) + `prisma/schema.prisma`; front
`carmi-digital/app/(customerPortal)/guias/**` y
`app/(customerPortal)/references/components/invoices/InvoiceFormModal.tsx`.

## Decisiones tomadas (entrevista de este plan)

1. **La factura puede crearse sin `referenceId`** (huérfana, ligada solo a
   `guiaId`) cuando se sube desde el módulo de Guías, ya que la guía puede
   no tener Referencia todavía. Requiere que `referenceId` deje de ser
   obligatorio en backend cuando hay `guiaId`.
2. **El cambio de backend (agregar `guiaId` a `Invoice`, ajustar regla de
   `referenceId`) se incluye en esta misma iniciativa** — no se asume que
   ya existe. Va en el sub-plan 01, antes del front.
3. **Migración automática al crear Referencia**: cuando se cree la
   Referencia/DGO desde guías seleccionadas (flujo ya existente de
   `CreateReferencesFromGuiasModal` → `linkGuiasToReference`), el backend
   debe enlazar automáticamente las facturas huérfanas de esas guías al/los
   DGO(s) resultantes, sin intervención manual del ejecutivo.
4. **Soporta múltiples facturas por guía desde v1** (lista + agregar), no
   solo una — cubre llegadas parciales (una misma guía House puede llegar
   en distintos vuelos y con varias facturas, según la nota de la
   reunión).
5. **Se reutiliza el formulario de factura completo** (mismos campos que
   hoy: partidas, montos, incoterm, vinculación, valoración, etc.), solo
   generalizando la asociación para aceptar `guiaId` en vez de exigir
   `dgoId`/`referenceId`. No se recorta a un subconjunto simplificado.
6. **Ubicación en UI**: acción nueva por fila en la tabla de Guías (no
   dentro de `GuiaFormModal`), que abre un modal "Facturas de la guía" con
   lista + botón agregar — mismo patrón que `GuiaStatusModal` /
   `BulkAssignGuiasModal`.
7. **Sin restricción de rol nueva**: cualquiera con acceso al módulo de
   Guías puede subir/vincular facturas (aunque la nota de la reunión decía
   "el ejecutivo", se decide no bloquearlo por rol en esta iteración).

## Fuera de alcance (toda la iniciativa, todos los sub-planes)

- Tarea 2 de la misma reunión: generar movimiento de entrada automático por
  guía House (backend, oculto en UI para Querétaro). Se planifica aparte;
  este paraguas **depende** de que exista un expediente/movimiento
  asociado a la guía para que la tarea 3 tenga sentido, pero no lo
  implementa.
- Tarea 3: integrar la factura subida desde Guías/DGO al mismo objeto de
  expediente del movimiento/referencia. Depende de la tarea 2 (no
  implementada aquí) — riesgo documentado abajo.
- Tarea 4: cuando se sube factura desde DGO/Referencia sin guía asociada,
  preguntar a qué guía pertenece (flujo inverso al de este paraguas).
- Tarea 5: subir documento genérico adicional en el formulario de factura
  de DGO, o ligar documento ya existente del expediente.
- Matching automático documento↔guía vía Zeus (mencionado como "a futuro,
  falta Zeus" en la nota de la reunión).
- Equivalente marítimo (esta iniciativa cubre el modelo de guía ya
  existente sin distinción de tráfico, pero no se valida explícitamente
  para marítimo).

## Sub-planes hijos (índice, en orden de dependencia)

### Sub-plan 01 — Backend: `guiaId` en Invoice + migración automática a DGO — 🟡 diseñado, listo para `/implementa`
Archivo: [[2026-08-01-facturas-en-guias-sp01-backend-guiaid-invoice]]
Cubre: agregar `guiaId` (nullable, FK a `Guia`) al modelo `Invoice` vía
`npx prisma migrate dev` (regla global: nunca escribir la migración a
mano); relajar la validación de `referenceId` obligatorio cuando hay
`guiaId`; endpoint(s) para listar facturas por guía (`GET
/guias/:id/invoices` o equivalente, ya que `guiasService` no tiene
`getById` hoy); lógica de migración automática de facturas huérfanas al
crear Referencia desde guías seleccionadas.

### Sub-plan 02 — Front: modal "Facturas de la guía" + generalización de InvoiceFormModal — ✅ implementado (2026-08-02), pendiente de revisión/commit humano
Archivo: [[2026-08-01-facturas-en-guias-sp02-front-formulario-facturas]]
Depende de: sub-plan 01 (necesita el endpoint y el campo `guiaId` reales
para poder probarse; no se mockea). Cubre: acción nueva en la tabla de
Guías, modal de lista + agregar, y los cambios en
`InvoiceFormModal`/`buildInvoiceDto` para aceptar `guiaId` y no exigir
`referenceId` en ese modo. Sus 6 tareas quedaron completadas y verificadas
con Playwright (crear factura + documento desde Guías, editar, regresión
DGO/Referencia). De paso se encontró y corrigió un bug preexistente de
paginación en `invoices.service.ts` (`GET /invoices` sin `page`/`limit`
explícitos tronaba por `NaN` — el `ValidationPipe` global convierte un
query param ausente en `NaN`, no `undefined`; se copió el patrón defensivo
que ya usaba `GuiasService.findAll`).

### Sub-plan 03 — Documento oficial de la factura (PDF/imagen) — ✅ implementado (2026-08-03), pendiente de revisión/commit humano
Archivo: [[2026-08-02-documento-oficial-factura-pdf]]
Surgió al revisar el sub-plan 02: el usuario notó que los botones de
"Imágenes" existentes (a nivel factura y a nivel producto) son para fotos
de producto, no para el documento/comprobante oficial de la factura
(PDF o imagen). No depende de los sub-planes 01/02 más que en compartir el
mismo `InvoiceFormModal` ya generalizado — aplica a los 3 flujos (Guías,
DGO, Referencia) por igual. Sus 9 pasos quedaron completados (backend:
migración + servicio compartido `InvoiceImagesService` + soporte
`documentId` + endpoint de histórico; frontend: `InvoiceDocumentUpload` +
wireado en `InvoiceFormModal` + histórico + preview). En la prueba manual
desde Guías se encontraron y corrigieron 2 bugs no relacionados al PDF en
sí: timeout de transacción Prisma (5s por defecto, insuficiente con
subidas de imagen dentro de la transacción — se amplió a 30s) y el
`documentId` apuntando al backend equivocado (`storageService.uploadFile()`
genérico del front subía a `carm-mimir-api`, no a `carmi-odin-api-v2` —
ahora usa `apiOdinClient` directo). De paso se agregó un indicador rápido
de "tiene facturas" en la tabla de Guías (badge con conteo sobre el botón
existente, vía `_count` en el include de `GuiasService.findAll`).

**Ramas de trabajo (decisión 2026-08-02, todo el paraguas)**: los 3
sub-planes se trabajan sobre las mismas 2 ramas por repo, sin commitear
todavía — no se crea una rama nueva por sub-plan-03:
- `carmi-digital`: `feat/facturas-en-guias`
- `carmi-odin-api-v2`: `feat/facturas-en-guias-sp01-backend-guiaid-invoice`
  (el nombre quedó con el sufijo `-sp01-` de cuando se creó, pero es la
  rama compartida de todo el paraguas — no renombrar, solo seguir
  trabajando ahí).

### Sub-plan 04 — Gestión de partidas para facturas creadas desde Guías — 🟡 diseñado, listo para `/implementa`
Archivo: [[2026-08-03-facturas-en-guias-sp04-partidas-en-guias]]
Surgió al probar el sub-plan 03 desde Guías: el usuario notó que no hay
forma de agregar partidas a una factura creada ahí. Investigación
confirmó que **no es una regresión** — `InvoiceFormModal` nunca tuvo
sección de partidas, ni en el flujo original de Referencias/DGO (se
agregan aparte, en `ReferenceMerchandiseTab.tsx` vía `AddPartidaDrawer`,
ambos ya agnósticos de `referenceId`/`guiaId`, solo necesitan
`invoiceId`). Decisión: extraer esa lógica a un componente compartido
(`InvoiceMerchandiseAccordion`) en vez de duplicarla, y que
`GuiaInvoicesModal` adopte el mismo acordeón que ya usa DGO, sin
auto-abrir el formulario de partida tras crear la factura (a diferencia
de DGO). Depende del sub-plan 02 (ya completo).

### Sub-plan 05 — Auto-vincular facturas de Guías al DGO cuando la guía ya tiene Referencia + mostrar la guía de origen — 🟡 diseñado, listo para `/implementa`
Archivo: [[2026-08-02-facturas-en-guias-sp05-auto-vinculo-dgo]]
Surgió al probar el sub-plan 04 end-to-end: la guía 1402027826 ya tenía
Referencia/DGO, pero una factura creada desde Guías después de esa
vinculación quedó huérfana (`referenceId`/`dgoId` null) porque
`InvoicesService.create()` solo auto-resuelve `referenceId` desde
`shipmentId`, nunca desde `guiaId` — y la migración automática de la
decisión 3 solo corre una vez, al vincular guías→Referencia, no rescata
facturas creadas después. Cubre: auto-vincular `referenceId`+`dgoId` en
`create()` (resolviendo el DGO correcto por *match* de `clavePedimento`,
ya que una Referencia puede tener varios DGOs) y mostrar un badge "Guía:
X" en `ReferenceDGOTab` y en `InvoiceMerchandiseAccordion` (modo DGO). Sin
backfill de facturas ya huérfanas existentes (decisión explícita).
Depende de los sub-planes 01 y 04 (ambos completos).

## Riesgos y side effects transversales

- `InvoiceFormModal` (1668 líneas) está fuertemente acoplado a la
  suposición de `referenceId` siempre presente (queries, invalidación de
  caché, keys de react-query). Hacerlo opcional exige auditar con cuidado
  antes de tocar, para no romper el flujo actual de DGO/Referencia.
- Hacer `referenceId` nullable en el backend puede afectar reportes o
  lógica existente que asuma su presencia (p. ej. métricas de turnaround de
  facturación mencionadas en `Tareas.md`, sección "Referencia, DGO y
  Operaciones") — revisar antes de relajar la constraint.
- `carmi-odin-api-v2` tiene un pre-commit que exige que **todo** método
  público de un `.service.ts`/`.controller.ts` staged aparezca referenciado
  en su `.spec.ts` (no solo los nuevos) — tocar `invoices.service.ts`
  puede exponer deuda de cobertura preexistente ajena a esta iniciativa.
- Sin movimiento/expediente por guía (tarea 2 fuera de alcance), la factura
  subida desde Guías quedará visible en la lista del nuevo modal pero
  **no** aparecerá todavía en el Expediente Aduanero del movimiento — dejar
  esto explícito al usuario/cliente para evitar expectativa equivocada
  mientras no se implemente la tarea 2/3.

## Criterios de verificación (globales)

- Cada sub-plan cierra con su propio `/verify` antes de darse por
  terminado.
- Al cerrar el sub-plan 02: crear una guía sin Referencia, subir una
  factura (documento + datos) desde el nuevo modal, confirmar que queda
  huérfana con `guiaId` y sin `referenceId`; después crear la Referencia
  desde esa guía y confirmar que la factura aparece ya vinculada al DGO
  resultante sin intervención manual.

## Estado del paraguas

**2026-08-01**: creado a partir de la tarea 1 de
[[2026-07-31 - Revisión de Movimientos - Guía]]. Sub-planes 01 (backend) y
02 (front) diseñados en la misma sesión de entrevista y listos para
`/implementa`, cada uno en su propia sesión limpia (sp01 primero, sp02
depende de él).

**2026-08-03**: sub-plan 03 (documento oficial) implementado y verificado
manualmente desde Guías (con 2 bugs preexistentes corregidos de paso, ver
su entrada arriba). De esa misma prueba salió el sub-plan 04 (partidas en
Guías), diseñado y listo para `/implementa` en sesión limpia.
