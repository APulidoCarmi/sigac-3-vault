# Manifiesto — SP-08: Recinto (solo lectura) por tráfico

**Estado: ✅ IMPLEMENTADO (2026-07-11).** Rama `refactor/customs-operation-sp08`
en `carmi-digital`, encadenada desde `refactor/customs-operation-sp07`. Diff
sin commitear, listo para revisión humana. `carmi-odin-api-v2` **no requirió
cambios** (se creó la rama por convención del paraguas, pero queda sin diff
propio de este sub-plan).

## Resumen de lo implementado

Solo frontend. El sub-plan pedía "Crea: nada estructural (renombre +
variación por tráfico)" — se respetó ese alcance mínimo, a diferencia de
SP-07 (Movimientos) que sí dividió en archivos por tráfico porque ahí
AEREO/MARITIMO quedan diferidos a sub-planes futuros (SP-10/SP-11). Aquí no
hay sub-plan futuro que vaya a rehacer Recinto por tráfico, así que se adaptó
el contenido **dentro del mismo componente** en vez de crear un router +
archivos `warehouse/ReferenceWarehouseTerrestre.tsx` /
`ReferenceWarehouseTrafficPending.tsx` (patrón de SP-07) que habría dejado
AEREO/MARITIMO como placeholders sin contenido real — no pedido por el
sub-plan (pide "contenido adaptado por tráfico", no "placeholder").

### `app/(customerPortal)/references/components/ReferenceDetailShell.tsx`

- Único cambio: `label: "Almacén"` → `label: "Recinto"` en el array
  `sections` (la entrada `value: "warehouse"` se dejó igual a propósito para
  no romper el ruteo existente por `?tab=warehouse`).

### `app/(customerPortal)/references/components/tabs/ReferenceWarehouse.tsx`

- Se agregó `getRecintoCopy(trafficCode)`, que lee
  `reference.trafficType?.code` (mismo campo expuesto por SP-07 en
  `useReferenceDetail.ts`, catálogo `TransportMode` compartido con
  Movimientos — no es un enum propio) y devuelve nombre/copy por tráfico:
  - `TERRESTRE` (o `trafficType` ausente, mismo fallback D1 que SP-07) →
    "Almacén".
  - `AEREO` → "Almacén Fiscalizado".
  - `MARITIMO` → "Terminal".
- El nombre de tráfico se usa en: el texto de carga ("Cargando información de
  {recinto.name}..."), el estado vacío (título/descripción), y un badge junto
  al nombre del registro en el header ("Recinto: {recinto.name}").
- **Solo lectura (paso 3 del sub-plan):** se removió toda la superficie de
  "gestión del recinto" que el componente D1 exponía (fuera de alcance según
  el sub-plan — es rol de almacén):
  - `handleAction` (Confirmar Arribo / Marcar Inspeccionado / Listo para
    Despacho / Despachar) y sus botones — eliminados.
  - `handleRequest` (crear solicitud de Desconsolidación/Notificación/
    Despacho) y sus botones en el tab "Solicitudes" — eliminados. El
    historial de solicitudes existentes se sigue mostrando (consulta), solo
    se quitó la capacidad de **crear** nuevas desde el ejecutivo.
  - Botón "Subir Evidencia" / "Subir Nueva" y el estado `showUploadModal` —
    eliminados junto con el modal `WarehouseEvidenceUpload` que dejaron de
    usar (ver abajo).
  - Botón "Crear Registro de Almacén" en el estado vacío — eliminado (era una
    acción de creación, no de consulta).
  - Prop `onUpdate` dejó de usarse dentro del componente (ya no hay
    mutaciones que requieran refrescar al padre), pero se conserva en la
    interfaz de props para no tocar la firma que `ReferenceDetailShell` ya
    invoca (`<ReferenceWarehouse reference={reference} onUpdate={onUpdate} />`,
    sin cambios en el shell más allá del label).
- Import `AlertCircle` (no usado ya en el archivo original) removido junto
  con el resto de la limpieza, por el estándar de no dejar código muerto al
  intervenir el archivo.

### `app/(customerPortal)/references/components/cards/WarehouseConditionsCard.tsx`

- Se removió el botón `Edit2` (ghost, sin `onClick` cableado en el original —
  affordance de edición sin funcionalidad real, incompatible con "solo
  lectura") y la prop `onUpdate` (no usada tras quitar el botón).

### `app/(customerPortal)/references/components/modals/WarehouseEvidenceUpload.tsx`

- **Eliminado.** Único consumidor era `ReferenceWarehouse.tsx`; al quitar la
  capacidad de subir evidencia (fuera de alcance, rol de almacén) el modal
  quedó huérfano — se confirmó con grep que ningún otro archivo lo importaba
  antes de borrarlo.

## Riesgos verificados

- El sub-plan pedía "confirmar qué datos expone hoy `/references/:id/warehouse/*`
  por tráfico". Hallazgo: **ese no es el endpoint real que consume el
  componente.** `ReferenceWarehouse.tsx` llama directo (fetch, sin pasar por
  `lib/api/modules`) a `GET /warehouse/reference/:id`, `POST /warehouse/:id/action`
  y `POST /warehouse/:id/request` — una superficie de API distinta a los
  endpoints `references.controller.ts` (`:id/warehouse/evidence`,
  `:id/warehouse/evidence/:shipmentId`, `:id/warehouse/status`, backed por
  `WarehouseApiService`, actualmente **mock data**, sin diferenciación por
  tráfico en ninguno de los dos casos). No se encontró un módulo `warehouse`
  standalone en `carmi-odin-api-v2/src` que sirva `/warehouse/reference/:id`
  (posible servicio externo o gap D1 no cubierto por este sub-plan). Como
  ninguno de los dos caminos diferencia por tráfico hoy, la adaptación de
  SP-08 es puramente de presentación (front); no había datos backend que
  cambiaran el contenido por tráfico que "confirmar" más allá de esto.
  **No se tocó backend** — está fuera del alcance declarado ("Crea: nada
  estructural").

## Archivos tocados

**Frontend (`carmi-digital`):**
- `app/(customerPortal)/references/components/ReferenceDetailShell.tsx` (label "Almacén" → "Recinto")
- `app/(customerPortal)/references/components/tabs/ReferenceWarehouse.tsx` (adaptado por tráfico, solo lectura)
- `app/(customerPortal)/references/components/cards/WarehouseConditionsCard.tsx` (removido botón de edición no funcional)
- `app/(customerPortal)/references/components/modals/WarehouseEvidenceUpload.tsx` (eliminado — huérfano)

**Backend (`carmi-odin-api-v2`):** sin cambios (rama creada por convención, sin diff propio).

## Verificación

- **Frontend**: `npx tsc --noEmit -p tsconfig.json` → 0 errores relacionados
  con los archivos tocados. `npx eslint` sobre los 3 archivos modificados →
  0 errores, 1 warning preexistente (`no-explicit-any` en el cast
  `STATUS_COLORS[...] as any`, heredado sin cambios del componente original,
  mismo patrón usado en otros tabs del dominio Referencia).
- **Playwright**: **pendiente de sesión humana** (mismo bloqueo transversal
  documentado desde SP-01: no hay `dev_url` configurado para este cliente).
  Flujo a validar cuando haya entorno: abrir el tab "Recinto" en una
  referencia de cada tráfico (terrestre/aéreo/marítimo) y confirmar que el
  nombre/copy se adapta (Almacén / Almacén Fiscalizado / Terminal), que no
  aparece ninguna acción de escritura (sin botones de acción/solicitud/subida),
  y que no hay errores de consola.

## Bloqueos

Ninguno propio de este sub-plan. Hereda el bloqueo transversal ya
documentado (falta `dev_url` para validación Playwright end-to-end).
