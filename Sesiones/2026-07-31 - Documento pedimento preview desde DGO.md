# Sesión 2026-07-31 — Documento pedimento preview desde DGO

Relacionado: [[2026-07-30-documento-pedimento-preview-desde-dgo]],
[[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]],
[[Wizard unificado de creación de operación (DGO)]]

## Qué se trabajó

`/implementa` completo del plan `2026-07-30-documento-pedimento-preview-desde-dgo` (6
tareas, ambos repos): permite ver el "Documento Anexo 22" en vivo desde un DGO sin
operación creada, antes solo disponible para pedimentos reales.

- `carmi-odin-api-v2`: extraje de `buildHbsFromPedimento` (`operation-dispatch.service.ts`)
  tres piezas reusables (`computeIncrementablesDecrementables`,
  `buildProveedoresFromInvoices`, `buildPartidasFromInvoices`) y una cuarta
  (`buildTasasYLiquidacion`, agregada durante la tarea 3 al notar que
  `TaxEngineService.calculateTaxes` ya devuelve `operationTaxes` con el mismo shape que
  `Operation.operationTaxes`). Limpié ~28 `console.log` de debug de esa función de paso.
  Nueva función hermana `buildHbsFromDgo(dgo)` arma el `hbsData` sintético resolviendo
  aduana/régimen por código (igual que `getProformaFromDgo`). Nuevo endpoint
  `GET /dgo/:id/pedimento-preview` (`getPedimentoPreviewFromDgo` en
  `OperationDispatchService`): delega a `getOperationPedimento` si el DGO ya está
  `locked`, si no arma el sintético e invoca `previewTaxes` (con `paymentDate` = hoy para
  forzar el TC FIX de Banxico) para poblar tasas/liquidación reales, con `try/catch` que
  degrada a vacío sin romper el documento si falla.
- `carmi-digital`: `OperationProformaDrawer.tsx` muestra el tab "Documento Anexo 22"
  también en modo DGO-only (antes oculto), la query de `pedimentoData` bifurca por
  `isDgoOnlyUnlocked` hacia `GET /dgo/:id/pedimento-preview`, banner "Vista previa — DGO
  sin operación, pendiente de validar" (patrón `Alert` amber ya usado en
  `BulkEditMerchandiseDrawer.tsx`), y gateo de los botones "Regenerar"/"Generar borrador"
  (no aplican sin `Operation` real).
- **Bug bloqueante encontrado y corregido durante la verificación (tarea 6)**: ver
  [[Sheets no-modales anidados y onInteractOutside]] — el `SheetContent` padre
  ("Acciones — DGO #N" en `ReferenceDGOTab.tsx`) no prevenía `onInteractOutside`, así que
  cualquier click dentro de un Sheet hijo no-modal (`OperationProformaDrawer`,
  `InvoiceFormModal`, incluido simplemente cambiar de tab) se detectaba como click "fuera"
  del padre y lo cerraba, desmontando también al hijo. Autorizado explícitamente por el
  usuario a investigar y corregir (el fix vive en un archivo fuera del diff que se había
  pedido no tocar en la sesión anterior, pero el bug bloqueaba la verificación de este
  plan). Verificado con Playwright: DGO sin operación (REF-00136, régimen IMD) y DGO
  `locked` (REF-00073, operación real OP-00041) — ambos flujos completos sin errores de
  consola ni cierres inesperados.

## Commits relevantes

Ninguno. Diff sin commitear en ambos repos, rama compartida
`feat/documento-pedimento-preview-desde-dgo` (`carmi-digital` y `carmi-odin-api-v2`), para
revisión humana tarea por tarea. `carmi-digital` además arrastra sin tocar el diff sin
commitear de la sesión anterior (fix original de Sheets anidados para "Ver Proforma") —
el usuario pidió explícitamente dejarlo intacto esta sesión.

## Decisiones (con su porqué)

- **`GET /dgo/:id/pedimento-preview` devuelve un array `[{ hbsData }]`**, misma forma que
  `GET /operations/:id/pedimento`, para que el front reutilice `pedimentoData[idx]?.hbsData`
  sin bifurcar el render entre modo real y preview.
- **`valor_dolares`/`valor_aduana` en modo preview**: no estaban en la lista explícita de
  "Fuera de alcance" del plan, pero dependen de cálculos a nivel `Operation`. Se resolvió
  usando `Reference.totalValue` como fallback y, cuando `previewTaxes` tiene éxito, sus
  `customsValue`/`customsValueMXN` reales (más preciso que un placeholder fijo).
- **`buildTasasYLiquidacion` extraído durante la tarea 3, no en la tarea 1**: al ir a
  cablear `previewTaxes` en `buildHbsFromDgo` noté que el bloque de tasas/liquidación de
  `buildHbsFromPedimento` era 100% reusable (mismo shape `{contributionCode, rateCode,
  rateValue, amount, paymentForm}` venga de `Operation.operationTaxes` o de
  `TaxEngineService.calculateTaxes().operationTaxes`). Se hizo como parte del scope de la
  tarea 3 en vez de re-abrir la 1 (ya revisada), por ser una extracción directamente al
  servicio de esa tarea.

## Aprendizajes / errores a no repetir

- **Sheets no-modales anidados (`modal={false}` + Portal) requieren `onInteractOutside`
  prevenido en TODOS los niveles del stack, no solo en el hijo que se abre encima.**
  Ver nota nueva [[Sheets no-modales anidados y onInteractOutside]] — el fix previo (sesión
  2026-07-28) solo cubrió el momento de *apertura* del hijo; cualquier interacción
  posterior dentro del hijo (cambiar de tab, click en un botón) seguía cerrando al padre
  en cascada porque el padre nunca dejó de tratar el contenido portado del hijo como
  "fuera" de sí mismo.
- Antes de dar por buena una verificación Playwright que solo confirma que un Sheet *abre*,
  probar también interacciones dentro de él (cambiar de tab, clicks en controles) — el
  bug de esta sesión solo se manifestaba al interactuar con contenido ya abierto, no al
  abrirlo.

## Pendientes

- Ninguno para este plan — las 6 tareas quedaron completas y verificadas.
- Diff sin commitear en ambos repos (`feat/documento-pedimento-preview-desde-dgo`),
  pendiente de revisión humana antes de mergear.
- Sigue pendiente de revisión/commit el diff de la sesión anterior en `carmi-digital`
  (fix original de Sheets anidados para "Ver Proforma") — el usuario pidió dejarlo intacto
  esta sesión también.
