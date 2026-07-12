# Manifiesto — SP-04: Expediente Aduanero + glosado (Zeus/CEUS)

Implementado 2026-07-12 en la misma sesión/subagente que SP-05, por la dependencia
circular declarada entre ambos sub-planes (ver sección dedicada abajo).

## Ramas
- `carmi-digital`: `refactor/customs-operation-sp05` (partió de
  `refactor/customs-operation-sp04`, que a su vez partió de
  `refactor/customs-operation-sp02`). El trabajo de SP-04 se hizo primero sobre `sp04`;
  al pasar a SP-05 se creó `sp05` encadenada y ahí quedó también la vuelta a cerrar la
  validación (b) de SP-04. Nada comiteado — diff en working tree para revisión humana.
- `carmi-odin-api-v2`: mismo patrón, `sp02` → `sp04` → `sp05`.

## Resolución de la dependencia circular SP-04 ↔ SP-05
Criterio aplicado (el sugerido por el orquestador, sin desviarme):
1. Construí primero todo lo que SP-04 puede hacer sin el DGO: repositorio documental
   con metadatos, clasificación, motor de re-glosado con las validaciones (a) y (c),
   primaria/secundaria, ligado, historial, borrado lógico, checklist, bloqueo de
   operación.
2. Implementé el modelo y los endpoints de SP-05 (DGO) completo.
3. Volví a SP-04 y cerré la validación (b) "consistencia con los demás documentos y el
   DGO": inyecté `DgoConsistencyValidatorService` (vive en `src/dgo/services/`) como la
   implementación real del contrato `DgoConsistencyValidator` que `GlosaEngineService`
   ya esperaba (token `DGO_CONSISTENCY_VALIDATOR`, inyección `@Optional`). El cableado
   final es en `DgoModule` (provee el token) + `ReferenceDocumentsModule` (importa
   `DgoModule` con `forwardRef` porque `DgoModule` también importa
   `ReferenceDocumentsModule`).

Mecanismo técnico: `GlosaEngineService.reglosa()` ejecuta 3 validaciones
(`LEGAL_VALIDITY`, `DGO_CONSISTENCY`, `PREVIO_CONSISTENCY`). Antes de que existiera
SP-05, `DGO_CONSISTENCY` siempre devolvía `applicable: false` (no bloqueaba). Con
`DgoConsistencyValidatorService` ya registrado, compara el total extraído de una
factura (JSON de Zeus/CEUS) contra el total capturado en las partidas (`InvoiceItem`)
del DGO al que esa factura terminó asignada.

## Archivos tocados (odin)
- `prisma/schema.prisma`: `ReferenceDocument` +campos (`category`, `legalBasis`,
  `requirements`, `validityDate`, `glosaStatus`, `glosaPhase`, `glosaErrorReason`,
  `extractedJson`, `isPrimary`, `linkedDocumentId` + auto-relación); modelo nuevo
  `ReferenceDocumentHistory`; enums `ReferenceDocumentGlosaStatus`,
  `ReferenceDocumentHistoryAction`.
- `prisma/migrations/20260712014901_add_dgo_and_expediente_glosado/` (generada con
  `npx prisma migrate dev`, incluye también los cambios de SP-05 — un solo `migrate dev`
  cubrió ambos sub-planes porque se diseñaron en la misma pasada de schema).
- `src/reference-documents/services/reference-documents.service.ts`: create/update con
  los nuevos campos + clasificación automática; `markPrimary`, `link`, `unlink`,
  `getReferenceHistory`, `reglosaExpediente`, `assertInvoicesGlossedOk`, `getChecklist`;
  `remove` ahora registra historial.
- `src/reference-documents/services/document-classifier.service.ts` (nuevo): asigna
  `category` a partir de `documentType`/`vucemDocumentTypeCode`.
- `src/reference-documents/services/glosa-engine.service.ts` (nuevo): motor de
  re-glosado, 3 validaciones, punto de extensión DI para SP-05.
- `src/reference-documents/dtos/create-reference-document.dto.ts`: +`category`,
  `legalBasis`, `requirements`, `validityDate`.
- `src/reference-documents/dtos/document-actions.dto.ts` (nuevo): DTOs de
  primaria/ligado/reglosado.
- `src/reference-documents/controllers/reference-documents.controller.ts`: endpoints
  nuevos (`checklist`, `history`, `reglosa`, `can-start-operation`, `primary`, `link`,
  `unlink`); `remove` acepta `performedBy` (query).
- `src/reference-documents/reference-documents.module.ts`: registra
  `DocumentClassifierService`, `GlosaEngineService`; importa `DgoModule` (forwardRef).
- Specs actualizados/creados: `reference-documents.service.spec.ts`,
  `reference-documents.controller.spec.ts` (ajustados a las nuevas dependencias/firmas),
  `glosa-engine.service.spec.ts` (nuevo).
- `src/dgo/**` (nuevo): ver manifiesto de SP-05 — incluye
  `DgoConsistencyValidatorService`, la pieza que cierra la validación (b) de este
  sub-plan.
- `src/app.module.ts`: registra `DgoModule`.

## Archivos tocados (digital)
- `app/(customerPortal)/references/components/tabs/ReferenceDocuments.tsx`: rename a
  "Expediente Aduanero"; usa `doc.category` del backend (fallback al heurístico D1);
  badges de estatus de glosa (verde/rojo con motivo vía tooltip) y primaria; botones
  "Re-glosar expediente" e "Historial" (Sheet); toggle primaria/secundaria por documento;
  borrado lógico envía `performedBy`; el upload dispara re-glosado del expediente.
- `app/(customerPortal)/references/components/ReferenceDetailShell.tsx`: sección
  `documents` renombrada a "Expediente Aduanero" en el menú lateral (icono sin cambios).
- Limpieza de código muerto confirmada por el inventario del paraguas: eliminados
  `tabs/ReferencePedimento.tsx`, `drawers/pedimento/PedimentoProformaDocument.tsx`,
  `drawers/pedimento/PedimentoHbsDocument.tsx` (verificado sin referencias activas).

## Desviaciones / reducciones de alcance
- **Checklist dinámico por perfil MOMP**: MOMP no expone hoy un catálogo estructurado de
  "documentos requeridos" (confirmado explorando `src/momp/`); el checklist usa la base
  normativa mínima (factura/packing list/BL) más una señal `mompReady` (MOMP aprobado
  por el cliente). Cuando MOMP exponga reglas documentales, sólo hay que enriquecer
  `ReferenceDocumentsService.getChecklist`.
- **Zeus/CEUS en vivo**: no se cablea ninguna llamada HTTP real a Zeus/CEUS para
  extracción/clasificación (el propio sub-plan marca como riesgo abierto "confirmar
  versión del endpoint v1 vs v3 antes de cablear"). El motor de re-glosado consume
  `extractedJson` ya resuelto (por quien suba/genere el documento) y se limita a
  glosarlo — la extracción en sí queda para cuando se confirme el endpoint.
- **Validación (c) contra el Previo**: no aplica todavía (no existe una fuente de
  resultado del Previo en el codebase — SP-09 fuera de alcance); el motor la deja
  como `applicable: false` (no bloquea), lista para activarse cuando SP-09 exista.
- **Bloqueo de inicio de operación**: expuesto como primitiva
  (`assertInvoicesGlossedOk` / `GET can-start-operation`); cablearlo dentro del wizard
  de Operación real es alcance de SP-06 (fuera de este sub-plan y del `customs-operation`
  IN-SCOPE, no se tocó `operations`).

## Verificación
- Backend: `npx tsc --noEmit` limpio (sin errores nuevos); `npx eslint
  src/reference-documents src/dgo src/app.module.ts` sin errores (sólo warnings
  preexistentes de un spec no tocado por mí); `npx jest reference-documents dgo` → 28/28
  tests verdes (incluye specs nuevos de `glosa-engine` y ajustes a los specs existentes).
- Frontend: `npx tsc --noEmit` sin errores nuevos (los únicos errores del repo son
  preexistentes y ajenos: módulos `three`/`mammoth` no instalados en otro feature);
  `npx eslint` sobre los 3 archivos tocados → 0 errores (sólo warnings preexistentes de
  estilo `any`, mismo patrón que ya usaba el archivo); `npx jest` → 86/86 tests verdes
  (2 suites que fallan son preexistentes y ajenas, falta `react-i18next` en
  `processManagement/documentsDashboard`, no tocado por este sub-plan).
- **Playwright no se corrió**: no hay `dev_url` configurado para este cliente y no se
  levantó el entorno dev en esta sesión (cambio de schema + backend nuevo primero). Queda
  pendiente para la siguiente sesión de `/verify` con el entorno arriba, revisando
  consola del navegador en: subir documento → dispara re-glosado → badges reflejan
  verde/rojo con motivo → marcar primaria/secundaria → borrado lógico → historial.

## Prisma
`npx prisma migrate status` confirmó estado limpio (420 migraciones) antes de tocar el
schema. Migración generada exclusivamente con `npx prisma migrate dev --name
add-dgo-and-expediente-glosado` (ninguna migración escrita a mano). Estado final: 421
migraciones, "Database schema is up to date!".
