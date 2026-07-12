# Manifiesto — SP-05: DGO — Datos Glosados para Operación

Implementado 2026-07-12 en la misma sesión/subagente que SP-04 (dependencia circular
declarada entre ambos). Ver el manifiesto de SP-04 para el detalle simétrico de la
resolución del ciclo; aquí se documenta la mitad propia de SP-05.

## Ramas
- `carmi-digital` y `carmi-odin-api-v2`: `refactor/customs-operation-sp05` (encadenada
  desde `sp04`, que a su vez parte de `sp02`). Nada comiteado.

## Decisiones de diseño del modelo (el paso más sensible del sub-plan)
- **La "Partida" ya existe**: `InvoiceItem` en D1 ya concentra ~90 campos por partida
  (fracción arancelaria, TLC/`treatyCode`, PROSEC, identificadores JSON, valor agregado,
  regla 8, etc.) — exactamente lo que el sub-plan pedía ("la partida concentra todos los
  datos"). **No se creó un modelo `Partida` nuevo** para no duplicar ~90 campos ya
  maduros y arriesgar una migración de datos innecesaria.
- **Modelo `Dgo` nuevo** (`prisma/schema.prisma`): campos de pedimento embebidos como
  columnas planas (`aduana`, `clavePedimento`, `patente`, `regimen`, `destino`,
  `observations`) — mismo patrón que ya usa `Reference` para sus campos legacy de
  pedimento (`Reference.aduana`, `Reference.clavePedimento`, etc.), en vez de agregar
  4 relaciones FK nuevas hacia `CustomsRegime`/`CustomsOffice`/`CustomsDeclarationCode`/
  `CustomsPatents` (habría exigido tocar esos 4 modelos para la relación inversa,
  ampliando innecesariamente la superficie de la migración). `dgoNumber` + `isDefault` +
  `status` (`DRAFT|IN_REVIEW|DISCREPANCY|SIGNED`) + `requiresManualGlosa` +
  `manualGlosaChecklist` (Json) + `signedAt`/`signedBy` + soft delete.
- **`Invoice.dgoId`** (nullable, `onDelete: SetNull`): una factura pertenece a lo más un
  DGO — implementa "1 DGO = 1 pedimento, se separa por clave".
- **`GlobalExpense.dgoId` + `GlobalExpense.guideNumber`**: incrementable/decrementable
  asignado a un DGO; `guideNumber` habilita "incrementable por guía + auto-suma" (f#6)
  agrupando por número de guía sin importar el DGO.
- **`GlobalIdentifier.dgoId`**: identificador nivel G aplicado a un DGO; para aplicar el
  mismo identificador a varios DGOs se crea una fila por DGO (mismo `code`, distinto
  `dgoId`) — evita un modelo de junction table adicional para un caso de uso simple.
- Migración generada con `npx prisma migrate dev --name add-dgo-and-expediente-glosado`
  (cubre también los campos de SP-04 sobre `ReferenceDocument`, una sola pasada de
  schema para ambos sub-planes). `npx prisma migrate status` confirmó estado limpio
  antes (420 migraciones) y después (421, sin conflictos).

## Archivos tocados (odin)
- `prisma/schema.prisma`: modelo `Dgo` + enum `DgoStatus`; `dgoId` en `Invoice`,
  `GlobalExpense`, `GlobalIdentifier`; `guideNumber` en `GlobalExpense`; `dgos Dgo[]` en
  `Reference`.
- `src/dgo/` (nuevo, módulo completo):
  - `dgo.module.ts`: registra `DgoService`, `DgoConsistencyValidatorService` y el binding
    `DGO_CONSISTENCY_VALIDATOR` (cierra SP-04 (b), ver su manifiesto); importa
    `ReferenceDocumentsModule` con `forwardRef`.
  - `dtos/dgo.dto.ts`: `UpdateDgoDto`, `SplitDgoByClaveDto`, `SignDgoDto`,
    `ManualGlosaDto`, `AllocateIncrementableDto`.
  - `services/dgo.service.ts`: `ensureDefault` (1 DGO por default, hereda campos de
    pedimento de `Reference`), `listByReference`, `findOne`, `update`, `splitByClave`
    (separa por clave de pedimento, reasigna facturas), `comparison` (JSON capturado —
    suma de partidas — vs factura real, inspirado en `StepPedimento`), `sign` (bloquea si
    hay discrepancias sin glosa manual), `manualGlosa` (checklist + firma cuando CEUS no
    disponible, M9), `incrementablesByGuide` (auto-suma f#6), `allocateIncrementable`
    (prorratea un incrementable sin asignar entre uno o varios DGOs, preserva el monto
    total).
  - `services/dgo-consistency-validator.service.ts`: implementación real de
    `DgoConsistencyValidator` (contrato de `GlosaEngineService`, SP-04) — compara el
    total extraído de una factura (JSON Zeus/CEUS) contra el total capturado en las
    partidas del DGO.
  - `controllers/dgo.controller.ts`: endpoints REST de todo lo anterior bajo `/dgo`.
  - `services/dgo.service.spec.ts` (nuevo): tests de `ensureDefault`, `comparison`,
    `sign`, `allocateIncrementable`.
- `src/app.module.ts`: registra `DgoModule`.

## Archivos tocados (digital)
- `app/(customerPortal)/references/components/tabs/ReferenceDGOTab.tsx` (nuevo): tab DGO
  — lista los DGO de la referencia (crea el default si no existe), muestra estatus,
  campos de pedimento, panel de comparación JSON-vs-factura (verde/rojo, mismo lenguaje
  visual que `StepPedimento`), botón "Firmar DGO", y un drawer "Acciones" con 3 tabs
  (Mercancía / Gastos-Incrementables / Identificadores) que reutilizan
  `ReferenceMerchandiseTab`, `ReferenceGlobalExpenses`, `ReferenceLegalConfiguration` tal
  cual (ver limitación abajo).
- `app/(customerPortal)/references/components/ReferenceDetailShell.tsx`: las secciones
  `inventory` (Mercancía), `globalExpenses` (Costos Globales) y `legalConfig` (Config
  Legal) se retiran del menú lateral (dejan de ser tabs propios, absorbidas por DGO);
  nueva sección `dgo` (ícono `Boxes`).

## Desviaciones / reducciones de alcance (documentadas explícitamente)
- **Filtrado por DGO en endpoints heredados**: `ReferenceMerchandiseTab`,
  `ReferenceGlobalExpenses` y `ReferenceLegalConfiguration` siguen consultando por
  `referenceId` (los endpoints `/invoice-items`, `/global-expenses`,
  `/identifiers` de D1 no tienen filtro por `dgoId`). Para el caso "1 DGO por default"
  (el más común hoy) el resultado es correcto porque todo pertenece al único DGO. Cuando
  una referencia se separa en N DGO (`splitByClave`), las Acciones muestran el universo
  completo de la referencia, no sólo lo del DGO abierto — se necesita agregar el filtro
  `dgoId` a esos 3 endpoints como siguiente paso (no lo hice para no tocar/ampliar el
  contrato de esos endpoints sin una revisión de quién más los consume).
- **Indicador de consistencia contra el Previo y MV electrónica**: fuera de esta pasada.
  No existe en el codebase una fuente de resultado del Previo (SP-09, fuera de alcance de
  SP-05) ni un flujo de Manifestación de Valor electrónica distinto del ya cableado en
  `Invoice`/`InvoiceItem` (`valorAgregado*`). `DgoService.comparison` sólo cubre
  partidas-vs-factura; se puede extender cuando SP-09 exista.
- **Happy path Ángel/German**: no se corrió — requeriría datos reales de esos clientes y
  un entorno levantado con Playwright; no ejecutable de forma verificable en esta sesión
  (no hay `dev_url` configurado y no se levantó el entorno). Pendiente en la siguiente
  sesión de `/verify`.
- **Escala (paginación/virtualización de partidas)**: no se tocó `ReferenceMerchandiseTab`
  más allá de embeberlo en el drawer de Acciones — ya usa acordeón por factura sin
  virtualizar; la advertencia "cientos/miles de partidas" del paraguas queda como deuda
  visible, no introducida por este sub-plan (ya existía en D1) pero tampoco resuelta.

## Verificación
- Backend: `npx tsc --noEmit` limpio (sin errores nuevos); `npx eslint src/dgo
  src/reference-documents src/app.module.ts` sin errores; `npx jest reference-documents
  dgo` → 28/28 verdes (incluye `dgo.service.spec.ts` nuevo: `ensureDefault` crea/reutiliza
  el default y hereda campos de pedimento, `comparison` marca discrepancia, `sign` bloquea
  sin glosa manual, `allocateIncrementable` rechaza sumas que no cuadran).
- Frontend: `npx tsc --noEmit` sin errores nuevos; `npx eslint` sobre
  `ReferenceDGOTab.tsx` y `ReferenceDetailShell.tsx` → 0 errores; `npx jest` → 86/86
  verdes (mismas 2 suites preexistentes/ajenas fallando, no relacionadas).
- **Playwright no se corrió** (mismo motivo que SP-04: sin `dev_url` configurado, entorno
  no levantado en esta sesión). Flujo pendiente de validar en la próxima sesión de
  `/verify`: crear referencia → 1 DGO default → separar por clave en 2 DGO → capturar
  partidas → ver discrepancias en rojo/verde contra la factura → prorratear un
  incrementable entre DGOs → firmar; revisando consola del navegador.
