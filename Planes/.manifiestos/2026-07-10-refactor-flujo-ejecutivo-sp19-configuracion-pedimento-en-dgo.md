# Manifiesto — SP-19: Configuración de datos de pedimento a nivel DGO

Sub-plan: `Planes/2026-07-10-refactor-flujo-ejecutivo-sp19-configuracion-pedimento-en-dgo.md`
Paraguas: `Planes/2026-07-10-refactor-flujo-ejecutivo.md`
Rama (ambos repos): `refactor/customs-operation-sp19`, encadenada sobre
`refactor/customs-operation-sp16b-legacy-purge` (diff acumulado del refactor grande
presente en ambos repos, confirmado antes de empezar). Sin commits (working tree).

## Pieza 1 — UI de configuración de pedimento en el DGO

**Archivo tocado:** `carmi-digital/app/(customerPortal)/references/components/tabs/ReferenceDGOTab.tsx`

- Nuevo componente `DgoPedimentoForm` (dentro del mismo archivo, reusando el patrón de
  `signMutation` ya existente): formulario controlado para `aduana`/`clavePedimento`/
  `patente`/`regimen`/`destino`, con `useMutation` que llama `PATCH /dgo/:id`, invalida
  `["dgos", referenceId]` y muestra toast de éxito/error.
- Se agregó como un cuarto tab **"Pedimento"** (ahora el tab por defecto) dentro del
  `DgoActionsDrawer` ya existente, junto a Mercancía/Gastos/Identificadores — sigue el
  patrón documentado en la cabecera del archivo ("Acciones del DGO como tabs de un mismo
  Sheet").
- Si `dgo.locked` es `true`, el formulario se reemplaza por un `Alert` de solo lectura
  explicando que el DGO ya originó un pedimento y que las correcciones se hacen a nivel
  pedimento/despacho (mensaje alineado con el que ya lanza el backend en `assertNotLocked`).
- La línea de resumen de solo lectura del acordeón (antes solo mostraba
  aduana/clave/patente/régimen) ahora también muestra `destino`, y se agregó un badge
  "Bloqueado" cuando `dgo.locked`.
- Se agregó `locked?: boolean` al tipo `Dgo` local del archivo.

**Backend — soporte necesario, no existía:**
`carmi-odin-api-v2/src/dgo/services/dgo.service.ts` (`listByReference`, `findOne`):
se agregó `_count: { select: { operationPedimentos: true } }` al `include` y un helper
privado `withLocked()` que deriva `locked: boolean` (`_count.operationPedimentos > 0`) y
lo expone en el payload, reemplazando el campo `_count` crudo. Antes el frontend no tenía
ninguna forma de saber de antemano si un DGO estaba bloqueado salvo intentar el PATCH y
capturar el 400 — ahora se sabe desde el GET.
El endpoint `PATCH /dgo/:id` y su DTO (`UpdateDgoDto`) ya existían completos y funcionando
(incluidos los 5 campos), no requirieron cambios.

**Verificación (Playwright, real, contra REF-00137 / DGO #1, sin pedimento aún):**
- Abrí el tab DGO → Acciones → tab "Pedimento", llené
  Aduana=140, Clave=A1, Patente=3846, Régimen=IMD, Destino="Planta Celaya", Guardar.
- Sin errores nuevos de consola (solo los 503 preexistentes de `/api/zeus/*`, no
  relacionados).
- Recargué la página (`GET /dgo/reference/:id`) y confirmé que el resumen del acordeón
  muestra "Aduana 140 · Clave A1 · Patente 3846 · Régimen IMD · Destino Planta Celaya" —
  persistencia confirmada.
- **No pude probar el caso "DGO ya bloqueado" con datos reales**: en la BD de este
  ambiente no existe ningún DGO vinculado a un `OperationPedimento` (`operationPedimento.count
  = 0` en toda la tabla). Verifiqué la lógica por code review (`assertNotLocked` cuenta
  `OperationPedimento.dgoId`, y mi `withLocked()` deriva el mismo booleano) y por el test
  unitario ya existente `dgo.service.spec.ts` (14 tests, pasan). Documento esto como
  limitación de datos de prueba, no de la implementación.

## Pieza 2 — Selector de grupo activo (bug de "grupos congelados") + auditoría Aduanas

**Archivos tocados:**
- `carmi-digital/app/(customerPortal)/customs-operation/createOperation/components/stepCustomsInfo.tsx`
- `carmi-digital/app/(customerPortal)/customs-operation/createOperation/page.tsx`

**Fix del bug real** (`stepCustomsInfo.tsx`):
- El `useEffect` que siembra el formulario (react-hook-form) desde `state.customsData`
  tenía `deps: []` (corría solo al montar) — por eso cambiar de grupo actualizaba el store
  pero el formulario seguía mostrando los valores del primer grupo. Se cambió a
  `deps: [activePedimentoGroupIndex]`, dejando `customsData`/`setValue` deliberadamente
  fuera (con comentario explicando por qué: incluir `customsData` reiniciaría el cursor
  del usuario en cada tecleo, porque el efecto de `watch` de abajo reescribe
  `customsData` en cada cambio).
- Se agregó el selector de grupo activo (nunca antes construido en la UI, aunque el store
  ya tenía `setActivePedimentoGroupIndex`/`loadGroupFormData`/`saveCurrentFormDataToGroup`
  sin consumidor): botones por cada `pedimentoGroups[i].label`, visibles cuando
  `pedimentoGroups.length > 1`, arriba de los tabs internos del step (visible sin importar
  en qué sub-tab del step esté el usuario, porque `customsData` abarca fechas/aduanas/
  transporte/etc. del grupo activo). Al cambiar de grupo: `saveCurrentFormDataToGroup`
  (grupo viejo) → `setActivePedimentoGroupIndex` → `loadGroupFormData` (grupo nuevo).
- **Fix adicional necesario en `page.tsx`**: el submit nunca llamaba
  `saveCurrentFormDataToGroup` antes de construir el payload — los edits del ÚLTIMO grupo
  activo vivían solo en `state.customsData` (compartido) y no se reflejaban en
  `pedimentoGroups[i].formData` hasta que el usuario cambiaba de grupo. Se agregó el flush
  (`saveCurrentFormDataToGroup(activePedimentoGroupIndex)`) justo antes de construir
  `pedimentos`, leyendo el estado fresco vía `useCreateOperationStore.getState()` (la
  variable `pedimentoGroups` capturada en el render sigue apuntando al array pre-flush).
  Se eliminó la variable `pedimentoGroups` del hook en `page.tsx` por quedar sin uso tras
  este cambio (evita código muerto).

**Decisión documentada — auditoría "Aduanas" del wizard** (`customsOfficeId`/`customsPatentId`):
- Confirmé en el DTO backend (`create-operation.dto.ts`) que `customsOfficeId` SÍ existe a
  nivel de grupo (`PedimentoGroupDto.customsOfficeId`, línea 182) — soporta que cada
  pedimento tenga su propia aduana de despacho. Dado que el DGO ya es la fuente de verdad
  configurada por el usuario (pieza 1) y el patrón ya establecido para `clavePedimento`/
  `regimen` (ya removidos como inputs, correctamente derivados del DGO), apliqué el mismo
  criterio: `customsOfficeId` ("Aduana de Despacho") ahora se muestra de **solo lectura**
  por grupo activo (sourced de `selectedDgos.find(d => d.id === group.dgoId)?.aduana`)
  cuando el grupo activo tiene `dgoId`; si no (flujo legacy sin DGO seleccionado), se
  mantiene el `ComboBox` editable de siempre.
- `customsPatentId` ("Patente Aduanal"): confirmé que el DTO backend NO tiene este campo a
  nivel de grupo (`PedimentoGroupDto` no lo incluye), solo existe en
  `CreateOperationDto.customsPatentId` (top-level). Es decir, el backend ya modela la
  patente como dato de la Operación completa, no del pedimento/DGO — a diferencia de la
  aduana. Decisión: se queda como input editable (única instancia, no por grupo), con un
  texto aclaratorio agregado ("Aplica a toda la Operación, no varía por pedimento/DGO")
  para que no se confunda con un dato del DGO.
- `customsEntryExitOfficeId` ("Aduana de Entrada/Salida") y "Mandatario"/`agentId` no se
  tocaron — son conceptualmente distintos (punto de entrada física / agente que despacha),
  fuera del acotamiento explícito del sub-plan a `customsOfficeId`/`customsPatentId`.
- Los badges de DGOs seleccionados en la tab "Aduanas" ahora resaltan (`variant="default"`)
  el DGO del grupo activo.

## Pieza 3 — Backend: resolver clave de pedimento server-side desde el DGO

**Archivo tocado:** `carmi-odin-api-v2/src/operations/services/operations.service.ts`

- `validarValorAgregadoPorClave` (antes usaba `dto.pedimentos[].pedimentoCode` directo del
  payload del wizard) ahora recibe el `dgoById` map (ya construido en `create()` a partir
  de `dgoService.validateHomogeneousRegimen`) y, cuando el grupo trae `dgoId`, resuelve la
  clave desde `dgoById.get(g.dgoId)?.clavePedimento` — ignorando el valor que mande el
  cliente para ese grupo. Si el grupo no trae `dgoId` (flujo legacy), sigue usando
  `g.pedimentoCode` del payload (retrocompatibilidad).
- `dgoById` se amplió (antes solo guardaba `regimen`) para incluir `clavePedimento`,
  aprovechando que `validateHomogeneousRegimen` ya trae el objeto `Dgo` completo de Prisma
  (sin `select`).
- **Auditoría de otros puntos de `dto.pedimentos[].X`** (pedida en el sub-plan): revisé
  todos los usos restantes en `operations.service.ts`. El `create()` construye el
  `Pedimento` "mínimo" (comentario explícito en el código, línea ~1266) y NO persiste
  `pedimentoCode`/`regime`/`customsOfficeId` de los grupos en ningún campo del Pedimento en
  la creación — el único punto real que confiaba en el payload del cliente para una
  decisión de negocio (la validación de Valor Agregado) era `validarValorAgregadoPorClave`,
  ya corregido. El único otro punto que toca campos análogos (`group.pedimentoCode`/
  `group.regime` en el método `update()`, líneas ~2478-2479) es una edición POSTERIOR de un
  pedimento ya creado y desconectado de la edición del DGO (el DGO ya estaría bloqueado en
  ese punto) — coherente con "correcciones se hacen a nivel pedimento/despacho" y fuera del
  alcance de este sub-plan (no se tocó).

**Verificación:** `npx tsc --noEmit` limpio, `eslint` sin errores (2 warnings preexistentes
sin relación, líneas 2943/3564), `jest operations.service.spec.ts` (24/24) y
`dgo.service.spec.ts` (14/14) pasan.

## Desviación/hallazgo NO previsto en el sub-plan (corregido igual, por bloquear verificación)

Durante la verificación con Playwright del Step 0 del wizard encontré **dos bugs
preexistentes de doble-envoltura de respuesta** en `dgo.controller.ts`, del mismo tipo que
ya estaba documentado y corregido en 3 endpoints hermanos del mismo archivo
(`listByReference`/`listSelectable`/`comparison`, con comentario "No envolver aquí: el
TransformInterceptor global ya envuelve..."), pero que no se habían corregido en estos dos:

1. `POST /dgo/validate-selection`: el controller envolvía manualmente
   `{ success, data: { valid, regimen, dgos }, timestamp }`, y el `TransformInterceptor`
   global volvía a envolver, dejando `response.data.data` como el wrapper interno en vez
   del resultado esperado por `dgoServices.validateDgoSelection` (front). Esto bloqueaba
   **completamente** el Step 0 del wizard con el toast "No se pudo validar la selección de
   DGOs." — para CUALQUIER selección, no solo 2+ DGOs.
2. `GET /dgo/:id` (`findOne`): mismo problema — rompía `dgoServices.getDgoById` (usado por
   el wizard para resolver `invoiceIds` de cada DGO tras validar la selección).

Ambos se corrigieron con el mismo patrón ya establecido en el archivo (dejar de envolver
manualmente, un solo nivel de envoltura vía el interceptor global). Verificado con `curl`
directo tras el fix (respuesta de un solo nivel) y confirmado en Playwright: el wizard
avanzó de Step 0 ("Referencias") a Step 1 ("Datos Básicos") con el toast correcto
"Selección válida — Régimen aduanero homogéneo: IMD".

No se tocó nada más de `dgo.controller.ts` fuera de estas dos correcciones puntuales
(mismo patrón, mismo archivo, ambas bloqueaban directamente la verificación de mi propio
trabajo). `tsc --noEmit` y `dgo.service.spec.ts` siguen verdes tras el cambio.

## Bloqueos de verificación (documentados, no resueltos — fuera de alcance de SP-19)

1. **No se pudo probar el escenario "crear Operación con 2+ DGOs"**: en la BD de este
   ambiente solo hay **un** DGO firmado y sin vincular a un pedimento (el de REF-00137,
   configurado en la pieza 1). Los demás DGOs candidatos (REF-00134, REF-00069) tienen
   discrepancias factura-capturado-vs-declarado que bloquean la firma
   (`DgoService.sign` → `BadRequestException` por `hasDiscrepancies` sin
   `requiresManualGlosa`) — validación de negocio preexistente (SP-05), no relacionada con
   este sub-plan, y resolverla (vía glosa manual) está fuera de alcance. Confirmé con
   consultas Prisma directas que no hay ningún DGO ya vinculado a un `OperationPedimento`
   en toda la BD tampoco (0 registros), así que ni siquiera pude verificar el caso "DGO
   bloqueado" de la pieza 1 con datos reales (sí por code review + tests unitarios).
2. **No se pudo navegar el wizard completo hasta el step "Despacho"** (donde vive mi
   cambio de la pieza 2) durante esta sesión: los steps 2 ("Datos Básicos") y 3
   ("Transporte") — archivos `stepOperationBasicData.tsx`/`stepTransportConfig.tsx`, que
   NO toqué — presentan un problema de layout donde el contenido del `tabpanel` colapsa a
   tamaño 0×0 (confirmado con `getBoundingClientRect()`: el wrapper reporta
   `opacity:1`/`display:block`/alto correcto, pero los elementos internos —p. ej. el
   `<h3>`— tienen rect `{0,0,0,0}`), dejando la pantalla visualmente en blanco pese a que
   el DOM contiene el HTML correcto y no hay errores de consola. Es un bug de
   renderizado/layout preexistente, no introducido por mí (confirmé con `git diff` que no
   toqué ninguno de esos dos archivos), y ajeno a los 3 archivos de mi alcance
   (`stepCustomsInfo.tsx`, `page.tsx`, `ReferenceDGOTab.tsx`). No lo corregí: excede el
   alcance de SP-19 y requeriría investigar el componente compartido de layout/Typography
   del wizard.
   - Sí verifiqué en Step 0 (que carga sin este problema) que, tras el fix de los bugs de
     doble-envoltura, el DGO configurado en la pieza 1 se refleja correctamente en el
     selector ("Régimen: IMD · Clave: A1"), y que la validación de selección homogénea
     funciona end-to-end.
   - El código de la pieza 2 (selector de grupo, fix del `useEffect`, flush en submit,
     lectura read-only de aduana por DGO) quedó verificado por: lectura/auditoría directa
     del código existente y nuevo, `tsc --noEmit` limpio, `eslint` limpio, y los tests
     unitarios ya existentes de `create-operation.store.spec.ts` (11/11, incluyendo
     `saveCurrentFormDataToGroup`/`loadGroupFormData`) — pero NO con un recorrido E2E
     visual del step "Despacho" en Playwright por el bloqueo de layout arriba descrito.
3. Gap preexistente observado (no corregido, no bloqueante): `GET /operations/prepare/:referenceId`
   no existe en el backend (`operations.controller.ts` no tiene esa ruta) — el frontend
   (`stepOperationBasicData.tsx`) lo llama para precargar gastos/identificadores globales,
   maneja el 404 con `if (response.ok)` (no rompe nada), pero genera 2 errores de consola
   404 en cada paso por ese step. Ajeno a SP-19, no tocado.

## Gates estáticos — resumen final

- **Backend**: `npx tsc --noEmit -p .` limpio para todos los archivos tocados;
  `eslint` sin errores nuevos; `jest` — `dgo.service.spec.ts` (14/14),
  `operations.service.spec.ts` (24/24) pasan.
- **Frontend**: `npx tsc --noEmit -p .` (repo completo) limpio; `eslint` sin errores
  nuevos en los 3 archivos tocados; `create-operation.store.spec.ts` (11/11) pasa
  (sin cambios al store, pero confirma que las funciones que consume mi UI siguen
  correctas).

## Índice del paraguas

Pendiente: marcar SP-19 como completado (con las notas de bloqueo de verificación arriba)
en `Planes/2026-07-10-refactor-flujo-ejecutivo.md`.
