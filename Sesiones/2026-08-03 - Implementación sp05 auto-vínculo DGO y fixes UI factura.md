# Sesión 2026-08-03 — Sub-plan 05 (auto-vínculo referenceId/dgoId) + fixes de UI en factura

Relacionado: [[2026-08-02-facturas-en-guias-sp05-auto-vinculo-dgo]], [[2026-08-01-facturas-en-guias]], [[2026-08-02-documento-oficial-factura-pdf]]

## Qué se trabajó

Implementación completa del sub-plan 05 (hijo del paraguas
`facturas-en-guias`): cuando se crea una factura desde el módulo de Guías
sobre una guía que **ya tiene** Referencia, la factura ahora se auto-vincula
a esa Referencia y su DGO (antes nacía huérfana). Las 6 tareas del plan
quedaron implementadas y verificadas con `/verify` (lint, typecheck, suite
completa de backend en verde, flujo E2E validado con Playwright):

1. `Guia.dgoId` (schema + migración por Prisma CLI).
2. `GuiasService.linkGuiasToReference` puebla `Guia.dgoId` por grupo de
   pedimento.
3. `InvoicesService.create()` auto-vincula `referenceId`/`dgoId` desde
   `guia.dgoId` cuando la factura viene con `guiaId` sin `referenceId`
   explícito.
4. `DgoService.listByReference`/`findOne` incluyen la guía de origen de
   cada factura.
5. Badge "Guía: {guiaMaster}" en `ReferenceDGOTab` (movido, tras feedback,
   al listado real `DgoComparisonPanel` — se había agregado primero como
   bloque duplicado).
6. Badge de guía en el acordeón de `ReferenceMerchandiseTab`
   (`InvoiceMerchandiseAccordion`, solo `context="dgo"`).

Después de cerrar el plan, sesión continuó con varios fixes de UI pedidos
por el usuario sobre el mismo flujo de facturas (documento oficial PDF):

- Rediseño de `InvoiceDocumentUpload.tsx`: el botón ya no cambia de tamaño
  entre "sin documento" (botón chico) y "con documento" (antes una `Card`
  grande) — ahora es siempre el mismo `Button` outline/sm, con las acciones
  (Ver, Ver histórico, Reemplazar) en un `DropdownMenu`. El nombre mostrado
  ya no es el `fileName` real: siempre "Factura Comercial · vN" (versión
  derivada de `entries.length` de `/invoices/:id/document-history`, no hay
  campo de versión en ningún DTO).
- Mismo criterio de nombrado aplicado en `InvoiceDocumentHistoryDialog.tsx`
  (cada versión del histórico y su vista previa).
- Vista previa inline del histórico (ícono de ojo) en vez de `window.open`
  a otra pestaña.

## Commits relevantes

Ninguno — todo el trabajo quedó sin commitear (staged en el repo back,
sin stagear en el front), a la espera de revisión humana. Regla del
proyecto: `/implementa` nunca commitea.

- Back (`carmi-odin-api-v2`, rama `feat/facturas-en-guias-sp01-backend-guiaid-invoice`):
  `prisma/schema.prisma`, migración `20260803041040_add_dgo_id_to_guia`,
  `guias.service.ts`, `invoices.service.ts`, `dgo.service.ts` + specs.
- Front (`carmi-digital`, rama `feat/facturas-en-guias`):
  `ReferenceDGOTab.tsx`, `ReferenceMerchandiseTab.tsx`,
  `InvoiceMerchandiseAccordion.tsx`, `InvoiceDocumentUpload.tsx`,
  `InvoiceDocumentHistoryDialog.tsx`.

## Decisiones (con su porqué)

- **Vinculación real vía `Guia.dgoId`** en vez de *match* por
  `clavePedimento`: se descartó el match porque `Dgo.clavePedimento` es
  `VarChar(2)` y `Guia.pedimentoCode` es `VarChar(5)` — cruce frágil sin
  normalización. Se prefirió un campo FK real, poblado en el mismo momento
  en que `linkGuiasToReference` resuelve el DGO por grupo.
- **Solo hacia adelante, sin backfill**: guías vinculadas *antes* de este
  cambio quedan con `dgoId: null` para siempre (decisión explícita del
  usuario, documentada en el plan). Una factura creada sobre una de esas
  guías "viejas" queda con `referenceId` pero `dgoId: null`.
- **Badge de guía solo en `context="dgo"`**: en `context="guia"` (módulo de
  Guías) sería redundante, el usuario ya sabe de qué guía son todas las
  facturas que ve ahí.
- **Nombre de documento fijo "Factura Comercial · vN"**: pedido explícito
  del usuario — el nombre real del archivo subido no debe mostrarse en la
  UI, solo un label estable + versión.

## Aprendizajes / errores a no repetir

- **[[ValidationPipe global convierte query param number ausente en NaN]]**
  no aplica aquí — ver la nota nueva de bug preexistente abajo, distinta.
- **Bug preexistente en `linkGuiasToReference`** (no introducido en esta
  sesión, verificado por aislamiento: se reprodujo con la adición de
  `Guia.dgoId` removida y el error persistió igual): el `$transaction([...])`
  en forma de arreglo incluía `invoice.updateMany(...)` — `Invoice` tiene
  trigger de auditoría, así que `AuditedPrismaService` envuelve esa llamada
  en `runAudited(...)` cuando se construye fuera de un tx activo, devolviendo
  una `Promise` normal en vez del `PrismaPromise` perezoso que Prisma exige
  para la forma de arreglo → rompía **todo** el arreglo con "All elements of
  the array need to be Prisma Client promises". **Se corrigió** convirtiendo
  ese `$transaction([...])` a la forma callback `runAudited(this.prisma,
  async tx => {...})` (mismo patrón que `remove()`/`restore()` en el mismo
  archivo). Cualquier código nuevo en este repo que arme un
  `$transaction([...])` como arreglo de promesas debe evitar mezclar
  llamadas a modelos auditados (`Invoice`, `Reference`, `Operation`, etc. —
  ver `AUDITED_MODELS` vía `pg_trigger`) fuera de un `runAudited` callback.
- **Diálogos/Sheets anidados y `pointer-events` pegado en `body`**: patrón
  ya conocido en el repo (`InvoiceFormModal.tsx`, `ItemDetailModal.tsx`,
  `OperationProformaDrawer.tsx`) mordió de nuevo al agregar un tercer nivel
  de `Dialog` anidado (Sheet no-modal → Dialog Histórico → Dialog Vista
  previa): Radix deja `pointer-events: none` en el body tras cerrar el
  último modal, bloqueando toda interacción. Fix: en el `onOpenChange` de
  cada Dialog anidado, al cerrar, `setTimeout(() => { document.body.style
  .pointerEvents = ''; }, 100)`.
- **Doble overlay = fondo casi negro**: mismo mecanismo — dos `Dialog`
  apilados sin `hideOverlay` en el hijo suman su `bg-black/80`. Ya hay un
  prop `hideOverlay` en `components/ui/dialog.tsx`/`sheet.tsx` para esto,
  documentado en su JSDoc.
- **`<button>` anidado dentro de `AccordionTrigger`**: el `AccordionTrigger`
  compartido (`components/ui/accordion.tsx`) envuelve TODO su `children` en
  un `<button>` de Radix — pasarle botones de acción como children genera
  HTML inválido (error de hidratación de Next). Si un trigger de acordeón
  necesita botones de acción además del label, hay que usar
  `AccordionPrimitive.Header`/`Trigger` directo (no el wrapper compartido) y
  poner los botones como hermanos del `Trigger`, no como sus hijos.
- **Backend corriendo como build compilado (`node dist/src/main`), no
  `nest start --watch`**: durante la verificación, reconstruir con `npm run
  build` no bastaba — había que reiniciar el proceso para que los cambios
  de código tomaran efecto. El proceso se auto-reinicia solo tras `npm run
  build` (hay un watcher externo), así que basta reconstruir y esperar,
  **no matar el proceso manualmente** (eso rompió el puerto una vez por
  conflicto con otro proceso del entorno en :47821, ajeno a este repo).
- **DB local de desarrollo se resetea entre sesiones**: los IDs de
  referencia/guía de prueba usados para verificar (REF-00136, REF-00137,
  etc.) cambiaron de una sesión a otra — no asumir que un ID de prueba
  sigue existiendo; volver a consultar por `referenceNumber`.

## Pendientes

- El diff completo (back + front) sigue sin commitear en ambos repos,
  pendiente de revisión humana antes de commit/PR.
- Sub-plan 05 queda con sus 6 tareas marcadas `[x]` en el plan — el
  paraguas `facturas-en-guias` puede continuar con el siguiente sub-plan
  pendiente (si lo hay) en una sesión limpia nueva.
- No se hizo backfill de `Guia.dgoId` para guías vinculadas antes de este
  cambio (decisión explícita, ver plan) — sigue como riesgo aceptado.
