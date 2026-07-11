# Manifiesto — SP-03: Shell del detalle de Referencia (menú lateral) + Resumen

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp03-detalle-shell-menu-lateral]]
Paraguas: [[2026-07-10-refactor-flujo-ejecutivo]]
Fecha: 2026-07-11. Implementación **rehecha desde cero** tras el descarte
(2026-07-11) de la rama previa `refactor/customs-operation-sp03` (ya eliminada).

## Repo y rama

- `carmi-digital`: rama `refactor/customs-operation-sp03`, creada desde `test`
  (rama base del repo al momento de este reinicio de Fase 1). Sin commits —
  el diff queda en el working tree para revisión humana.
- `carmi-odin-api-v2`: NO tocado — el sub-plan es front-only (shell de UI del
  detalle de Referencia), no requiere cambios de API/back. No se creó rama en
  este repo.

## Auditoría previa (por qué se confirmó que no había residuo)

Antes de tocar código se verificó que ambos repos estaban en su rama base
(`test` / `staging`) con solo cambios ajenos preexistentes sin commitear
(`public/firebase-messaging-sw.js` en digital, `src/companies/companies.module.ts`
en odin — no tocados). Se buscó `ReferenceDetailShell`/`*Shell*` en
`references/**` y no existía: el código seguía siendo el D1 original con
`ReferenceTabs.tsx` (Tabs de shadcn), confirmando que la rama descartada no
dejó residuo y que se partía limpio del as-is documentado en el paraguas.

## Archivos tocados (`carmi-digital`)

- **`app/(customerPortal)/references/components/ReferenceDetailShell.tsx`**
  (nuevo — `git mv` de `ReferenceTabs.tsx`, mismo componente reescrito). Antes:
  `Tabs`/`TabsList`/`TabsTrigger` de shadcn con split "primary tabs" (grid de 5)
  + dropdown "Más" para las 6 secundarias (11 tabs totales, incluida
  `timeline`). Ahora: menú lateral de **una sola columna** (`<nav>` vertical de
  botones, `w-56` en desktop / fila horizontal scrolleable en mobile) con las
  **10 secciones** restantes (`overview`, `inventory`, `shipments`,
  `globalExpenses`, `legalConfig`, `instructions`, `documents`, `operations`,
  `appointments`, `warehouse`) — ya no hace falta el split primary/dropdown
  porque el lateral tiene espacio de sobra. Se preservó:
  - El ruteo por `?tab=` querystring (mismo patrón `useSearchParams` +
    `router.replace`, sin cambiar la API de navegación para no romper enlaces
    externos existentes que ya usan `?tab=warehouse`, etc. — ver
    `ReferenceHeroHeader.tsx:96`).
  - Fallback a `overview` si la URL trae un valor no navegable (p.ej.
    `?tab=timeline`, absorbida ahora en Resumen) o desconocido.
  - Los badges de notificación por sección (documentos/operaciones/movimientos)
    y los `Suspense` + skeletons por sección, igual que en el original.
  - La transición `framer-motion` (`AnimatePresence`) entre secciones.
  - Se tipó explícitamente el arreglo `sections: Section[]` (en vez de
    `as const`) porque el badge es opcional en unas secciones y no en otras;
    con `as const` TS infería tipos-literal heterogéneos y fallaba al leer
    `.badge` genéricamente (error TS2339, corregido).

- **`app/(customerPortal)/references/[id]/page.tsx`**: import y uso de
  `ReferenceTabs` → `ReferenceDetailShell`; comentarios actualizados
  ("Tabs (Executive Work Area)" → "Shell de menú lateral"; "Sticky Tabs
  Container" → "Sticky Header Container"); `Card` ahora con `overflow-hidden`
  para que el borde del menú lateral no se salga del contenedor redondeado.

- **`app/(customerPortal)/references/components/tabs/ReferenceOverview.tsx`**:
  se agregó una 3ª columna (`lg:col-span-1`, la que en D1 quedaba vacía en el
  grid `lg:grid-cols-3`) que renderiza `<ReferenceTimeline referenceId={...} />`
  reutilizado tal cual de `ReferenceTimeline.tsx` (consume
  `GET /references/:id/timeline`, sin cambios en ese componente ni en el back).
  Decisión tomada (punto abierto del sub-plan): se sigue **M9** (línea de
  tiempo dentro de Resumen) sobre el Inventario #13 (tab propia) por ser M9 la
  fuente de diseño más reciente — mismo criterio que ya estaba anotado en el
  sub-plan.

- **`app/(customerPortal)/references/components/ReferenceHeroHeader.tsx`**:
  breadcrumb "Referencias › {clientCompany.tradeName} › {referenceNumber}" →
  "Referencias › {referenceNumber}". El segmento intermedio enlazaba a la
  misma ruta `/references` que "Referencias" (redundante, no navegaba a una
  vista filtrada por cliente), así que se elimina en vez de arreglarle el
  destino (no hay una ruta de detalle de cliente en scope de este sub-plan).

## Fuera de alcance (no tocado, verificado)

- Contenido/rediseño de cada sección individual (Expediente, DGO, Movimientos,
  etc.) — sus propios sub-planes (SP-01, SP-04 a SP-18).
- Módulo `operations` (`customerPortal/operations`, `components/operations`,
  controllers `operations` del back) — no se tocó nada con ese nombre.
- `carmi-odin-api-v2` — sin cambios, sub-plan front-only.
- `public/firebase-messaging-sw.js` (digital) y `src/companies/companies.module.ts`
  (odin) — cambios ajenos preexistentes sin commitear, no tocados.

## Verificación

- **`npx tsc --noEmit -p tsconfig.json`**: sin errores nuevos en los archivos
  tocados. Persisten 4 errores preexistentes y ajenos (`Trailer3D.tsx`: módulo
  `three` no instalado; `DocxRenderer.tsx`: módulo `mammoth` no instalado) —
  no relacionados con este sub-plan.
- **`npx eslint` sobre los 4 archivos tocados**: 0 errores. 5 warnings, todos
  en `ReferenceHeroHeader.tsx` (imports/variables no usadas) — verificado con
  `git stash` que son preexistentes al diff de este sub-plan, no introducidos
  por el cambio de breadcrumb.
- **Playwright**: NO ejecutado. El entorno de este cliente usa credenciales
  reales (`.env`) contra infraestructura viva y esta ejecución corre
  desatendida (sin supervisión humana en vivo) — no se navegó el flujo de
  detalle de referencia en el navegador. Queda pendiente que un humano lo
  valide (menú lateral navegando las 10 secciones sin errores de consola,
  timeline visible en Resumen, breadcrumb correcto) antes de mergear.

## Bloqueos / desviaciones

Ninguno. Único ajuste respecto a la redacción previa del sub-plan: el conteo
de secciones del lateral es **10** (no 9) — D1 tenía 11 tabs totales
(`overview, inventory, shipments, globalExpenses, legalConfig, instructions,
documents, appointments, warehouse, operations, timeline`), y al quitar
`timeline` (absorbida en Resumen) quedan 10, no 9. Se corrigió la aritmética
en el propio sub-plan al cerrarlo; no cambia ninguna decisión de diseño, solo
la nota de la cuenta.
