# Manifiesto — SP-15: Ver Proforma / Pedimento (Anexo 22) + limpieza de código muerto

Implementado 2026-07-12. Sub-plan de bajo riesgo confirmado: reuso puro + limpieza,
sin cambios funcionales.

## Rama
`carmi-digital`: `refactor/customs-operation-sp15`, encadenada desde
`refactor/customs-operation-sp13` (diff acumulado previo confirmado presente con
`git status` antes de empezar). Nada comiteado — diff en working tree para revisión
humana. `carmi-odin-api-v2` no se tocó: todo el trabajo de este sub-plan cayó en el
front (viewer de proforma/pedimento).

## Paso 1 — Verificar que el viewer refleja los datos del DGO y renombres
Verificado por inspección, sin necesidad de cambios de código:
- El drawer real montado en `tabs/ReferenceOperations.tsx` es
  `drawers/OperationProformaDrawer.tsx` (único importador confirmado por grep). Este
  archivo NO fue tocado por sesiones previas de este paraguas (su último commit real es
  `be4798da feat: tipo de cambio automatico`) y ya consume correctamente:
  - `identifiers` (SPEC-009, sección "Identificadores"), poblado por
    `operations.service.ts#getProforma` (línea ~2750, incluye `identifiers` con
    `code/complement1-3/isInherited/sourceGlobalIdentifierId`).
  - Pedimento(s) vinculados vía `allPedimentos`/`pedimento` (multi-pedimento, alineado
    con el modelo `OperationPedimento` post-DGO).
  - Tipo de cambio capturado vs. efectivo (SPEC-TC-001).
  - El tab "Documento Anexo 22" usa `PedimentoHtmlViewer.tsx`, que renderiza el HTML
    generado por el backend (`POST /reports/pedimento/html`, alimentado por
    `OperationDispatchService.buildHbsFromPedimento`) — la generación del pedimento en
    sí es expresamente **fuera de alcance** de SP-15 ("vive en la Operación/DGO"), así
    que no se tocó esa lógica.
- **Hallazgo adicional no listado en el D1 del sub-plan:** los 4 archivos
  `drawers/pedimento/ProformaHeader.tsx`, `ProformaPartidas.tsx`,
  `ProformaLiquidation.tsx`, `ProformaAnnexes.tsx` (una implementación React anterior,
  fase SPEC-008, del Anexo 22) **no tienen ningún importador** — confirmado con grep
  recursivo sobre `.tsx`/`.ts` fuera de sus propios archivos y sin `index.ts` que los
  re-exporte. Quedaron huérfanos cuando se adoptó `PedimentoHtmlViewer.tsx` (HTML
  server-rendered vía Handlebars) como implementación real. Al ser código muerto
  confirmado dentro de la misma familia de componentes que este sub-plan audita, y
  dado el estándar del repo contra código muerto, se eliminaron junto con la limpieza
  declarada — ver Paso 2.
- Conclusión: no hizo falta ningún refactor de datos/renombres en el viewer; ya está
  alineado con DGO y con los renombres vigentes (identificadores, incrementables/
  decrementables se muestran únicamente en el documento Anexo 22 generado en backend,
  no en el resumen).

## Paso 2 — Eliminar código muerto
- `drawers/pedimento/PedimentoProformaDocument.tsx`,
  `drawers/pedimento/PedimentoHbsDocument.tsx`, `tabs/ReferencePedimento.tsx`: **ya
  habían sido eliminados por SP-04** (confirmado leyendo su manifiesto antes de tocar
  nada; `git status` los muestra como `D` staged en la rama heredada). No se repitió el
  borrado, sólo se verificó.
- Eliminados en esta sesión (hallazgo propio, ver Paso 1):
  - `app/(customerPortal)/references/components/drawers/pedimento/ProformaHeader.tsx`
  - `app/(customerPortal)/references/components/drawers/pedimento/ProformaPartidas.tsx`
  - `app/(customerPortal)/references/components/drawers/pedimento/ProformaLiquidation.tsx`
  - `app/(customerPortal)/references/components/drawers/pedimento/ProformaAnnexes.tsx`

## Archivos tocados
- `carmi-digital`: los 4 borrados arriba (`git rm`). Ningún archivo modificado — el
  viewer activo (`OperationProformaDrawer.tsx`, `PedimentoHtmlViewer.tsx`) ya estaba
  correcto y no requirió cambios.
- `carmi-odin-api-v2`: ninguno.

## Desviaciones
- El D1 del sub-plan asumía que `OperationProformaDrawer.tsx` se apoyaba en
  `ProformaHeader/Partidas/Liquidation/Annexes.tsx` para el Anexo 22; en la práctica el
  Anexo 22 se resuelve vía `PedimentoHtmlViewer.tsx` + backend Handlebars, y esos 4
  archivos estaban huérfanos. Se documenta como hallazgo y se limpiaron por caer
  directamente dentro del alcance de "limpieza de código muerto" de este mismo
  sub-plan (misma carpeta `drawers/pedimento`), sin tocar nada fuera de
  `references/components`.
- No se tocó `src/operations` de odin: no hizo falta ningún cambio de backend para
  este sub-plan.

## Riesgos y side effects
- Confirmado con grep (no graphify disponible en este entorno) que los 3 archivos
  originales seguían sin importadores (ya vacíos, sólo verificación) y que los 4
  archivos adicionales tampoco tenían importadores antes de borrarlos.

## Verificación
- Frontend: `npx tsc --noEmit` → limpio, sin errores nuevos.
- Frontend: `npx eslint "app/(customerPortal)/references/components/drawers"` → 4
  errores, todos preexistentes en `BulkEditMerchandiseDrawer.tsx` (comillas sin escapar,
  archivo no tocado por este sub-plan); `OperationProformaDrawer.tsx` sólo con warnings
  preexistentes de `no-explicit-any` (mismo patrón ya presente en el archivo, no
  tocado). Sin errores en los archivos borrados (dejaron de existir) ni referencias
  rotas hacia ellos.
- Búsqueda de tests: sin specs que referencien los 4 archivos borrados.
- **Playwright no se corrió**: no hay `dev_url` configurado para este cliente y no se
  detectaron servidores dev activos de sesiones previas en este momento. Queda
  pendiente para la siguiente sesión de `/verify` con el entorno arriba: abrir la
  proforma/pedimento de una operación con pedimento generado, tab "Documento Anexo 22",
  confirmar que carga el HTML sin errores de consola.

## Estado
✅ Implementado.
