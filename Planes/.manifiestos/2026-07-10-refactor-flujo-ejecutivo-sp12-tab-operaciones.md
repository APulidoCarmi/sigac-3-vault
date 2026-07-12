# Manifiesto — SP-12: Tab Operaciones como agrupador vía DGO

Implementado 2026-07-12. Ver sub-plan
[[2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]].

## Ramas
- `carmi-odin-api-v2`: `refactor/customs-operation-sp12` (encadenada desde
  `refactor/customs-operation-sp10`). Nada comiteado.
- `carmi-digital`: rama `refactor/customs-operation-sp12` creada (encadenada desde
  `sp10`) por si un paso futuro la necesitara, pero **no se tocó ningún archivo del
  front en este sub-plan** — ver sección "Por qué el front queda intacto".

## Resumen del cambio (solo backend)
El tab reusa `GET /references/:id/operations` → `ReferencesService.getOperations`
(`src/references/services/references.service.ts`). Antes, el `where` solo consideraba
dos vínculos "estáticos" fijados al crear la Operación:
- `referenceId` (la referencia primaria).
- `operationMetadata.additionalReferenceIds` (JSON snapshot poblado una sola vez en
  `OperationsService.create()`, ver `src/operations/services/operations.service.ts`
  líneas ~1008 y ~1180 — nunca se vuelve a actualizar si después se agregan pedimentos
  vinculados a otras referencias vía DGO).

Con el modelo DGO de SP-05/SP-06, el vínculo *real* de un pedimento con su referencia
vive en `OperationPedimento.dgoId → Dgo.referenceId` (ver `prisma/schema.prisma`,
modelos `Dgo` ~línea 3469, `OperationPedimento` ~línea 2364). Una Operación puede
agrupar pedimentos cuyos DGO pertenecen a referencias distintas de la primaria, y ese
vínculo puede crearse **después** de que la Operación ya existe — el snapshot JSON no
lo refleja. SP-12 agrega una tercera rama al `OR`:

```ts
{
  pedimentos: {
    some: {
      dgo: { referenceId: id, deletedAt: null },
    },
  },
},
```

Además, el `select` de `pedimentos` ahora trae `dgo: { select: { referenceId: true } }`
por cada `OperationPedimento`, y tanto `allLinkedIds` (para resolver `referenceNumber`)
como `linkedReferences` (payload que consume el front) incorporan esas referencias
derivadas de DGO junto a `referenceId`/`additionalReferenceIds`.

## Archivos tocados (odin)
- `src/references/services/references.service.ts`:
  - `getOperations()`, dentro del `where` de `this.prisma.operation.findMany` (rama
    `OR` nueva, comentario `// SP-12: ...`).
  - `select.pedimentos.select`: agregado `dgo: { select: { referenceId: true } }`
    (sibling de `pedimentoId`/`pedimentoType`/`pedimento`).
  - Cómputo de `allLinkedIds` (antes de resolver `linkedRefMap`): agregado loop sobre
    `op.pedimentos` sumando `opPed.dgo?.referenceId`.
  - Cómputo de `linkedReferences` dentro del `.map(op => ...)` de `enriched`: mismo
    loop agregado antes del `[...new Set(allIds)].map(...)`.
- `src/references/services/references.service.spec.ts`:
  - Agregados mocks `mockPrisma.operation.findMany`, `mockPrisma.pedimento.findMany`,
    `mockPrisma.customsOffice.findMany` (no existían — `getOperations` nunca tuvo
    tests antes de este sub-plan).
  - Nuevo `describe('getOperations', ...)` con 3 tests: 404 si la referencia no
    existe, el `where.OR` incluye las 3 ramas, y un caso end-to-end donde una
    Operación con `referenceId: 'r-2'` pero un pedimento cuyo `dgo.referenceId` es
    `'r-1'` aparece correctamente en `linkedReferences` al consultar `r-1`.

## Por qué el front queda intacto (importante para SP-14, que toca el mismo archivo)
`references/components/tabs/ReferenceOperations.tsx` **no se modificó — cero diff**.
El D1 del sub-plan decía explícitamente "Crea: nada estructural (ajuste de
fuente de datos/consulta)". Al revisar el componente (líneas 150-166: el `useQuery`
que llama `GET /references/:id/operations`; líneas 269-333: el render de
`operation.linkedReferences` con badge "N referencias" + tooltip listando cada una)
ya soporta multi-referencia de punta a punta desde SPEC-007 (SP-07) — solo faltaba
que el backend devolviera el conjunto correcto y completo de operaciones/referencias
vinculadas, que es justo lo que este sub-plan corrigió.

**Mapa para el subagente de SP-14** (Solicitud de Fondos, toca el mismo archivo,
líneas 196-221 en la versión actual): el handler `handleSolicitarFondos` vive ahí
(fetch a `POST /operations/:id/solicitar-fondos`), justo arriba del bloque de render
(línea 223 en adelante). El botón "Solicitar Fondos" del `DropdownMenuItem` está en
líneas 468-478. Ningún cambio de SP-12 tocó esas líneas ni las de arriba/abajo — el
archivo completo permanece como estaba antes de este sub-plan, así que SP-14 puede
editar esa zona sin conflicto de merge con este trabajo.

## Verificación
- `npx tsc --noEmit -p tsconfig.json`: sin errores nuevos en `references.service.ts`
  (los errores preexistentes del repo están en archivos de test no relacionados:
  `twilio.service.spec.ts`, `seal-resolver.service.spec.ts`,
  `company-patent-signature-config.service.spec.ts`, `audited.spec.ts`, y 3
  `*.e2e-spec.ts` — mismos errores con o sin este cambio).
  npx eslint sobre `references.service.ts` y el `.spec.ts`: mismos hallazgos
  preexistentes (verificado con `git stash`/`git stash pop`), ningún error nuevo en
  las líneas tocadas.
- `npx jest src/references`: 3 suites, 7 tests, todos verdes (antes: 5 tests, 3
  suites — se sumaron 2 tests nuevos de `getOperations`).
- Playwright: no ejecutado (el sub-plan no toca front; su criterio de verificación
  —"una referencia cuyos DGO están repartidos en 2 operaciones muestra ambas"— quedó
  cubierto a nivel de servicio con el test "surfaces an operation linked only via a
  DGO belonging to another reference"). Pendiente de sesión humana con Playwright
  contra un entorno con datos reales si se quiere validación E2E visual.

## Desviaciones / decisiones fuera de lo literal del sub-plan
- Se agregaron tests nuevos a `references.service.spec.ts` aunque el sub-plan no lo
  pedía explícitamente — `getOperations` no tenía cobertura previa y es la pieza
  central del cambio; se consideró necesario para no dejar el ajuste de consulta sin
  verificación automatizada dado que Playwright no es viable en este entorno.
- No se tocó `prisma/schema.prisma` (no hacía falta modelo/campo nuevo — el modelo
  DGO de SP-05 ya expone todo lo necesario), por lo que no aplica la regla de
  migraciones vía CLI.

## Coordinación externa (sin acción tomada)
Nota del paraguas: "el listado de operaciones lo desarrolla Carlos (M8); coordinar
SP-01/SP-12 con él." No se detectaron señales de trabajo concurrente de Carlos en
`src/references/services/references.service.ts` ni en `ReferenceOperations.tsx`
durante esta sesión. Queda como pendiente de coordinación humana, fuera del alcance
técnico de este sub-plan.
