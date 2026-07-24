# Plan: Tablero de control de guías (pre-referencia) + régimen por guía — Flujo Querétaro (aéreo)

> Sub-plan 01 del paraguas [[2026-07-22-flujo-queretaro-aereo]]. Cubre las
> tareas 13 y 19 de `Tareas.md` (sección Flujo Querétaro (aéreo)).

## Contexto

Origen: [[2026-07-20 - Recap visita Querétaro]] (reparto de trabajo
confirmado: Ángel — "generación de la tabla/tablero de control de guías").
Refinado técnicamente en [[2026-07-21 - Análisis de flujo de referencias (visita Querétaro)]]
y [[2026-07-22 - Refinamiento de tareas Querétaro]].

Corresponde a las tareas de `Tareas.md`:
- **13**: "Construir tablero de control de 'guías' pre-referencia" — ingesta
  del manifiesto diario de Terminal, alta de guías con estatus de
  seguimiento, comparación contra prealerta cuando exista, creación de
  referencia(s) desde selección de guías.
- **19**: "Asignar régimen por guía desde el tablero de Guías, heredable a
  la Referencia al crearla" — fusionada aquí porque el punto anterior
  (creación de referencia desde selección) no puede funcionar sin ella.

Área de código:
- Back: `carmi-odin-api-v2/src/references/services/references.service.ts`
  (fix de bug en `create()` + nuevo endpoint batch de creación desde
  selección), módulo nuevo `src/guias/**` (controller, service, DTOs), y
  `dgo.service.ts`/`operations.service.ts` (resolución de `regimeId` desde
  `dgo.regimen` al armar `pedimentoGroups`, ver Decisión 16).
- Front: `carmi-digital`, pantalla nueva del tablero (ruta a definir, ej.
  `app/(customerPortal)/guias/**`), reusa componentes UI existentes
  (`components/ui/table.tsx`, `dialog.tsx`, `badge.tsx`) y el patrón de
  selector de compañía de `createReference` — `SearchableSelect`
  (`components/ui/searchable-select.tsx`) con `fetch` a
  `/companies/clients/importer-exporter`, tal como lo usa
  `stepBasicDataReference/index.tsx` (**no** `companyService`: hay dos
  implementaciones distintas en el repo con otros propósitos —
  `features/companies/services/companyService.ts`, catálogo CRUD, y
  `features/appointments/api/companyService.ts`, búsqueda para citas —
  ninguna es la que usa `createReference`). También toca
  `app/(customerPortal)/customs-operation/createOperation/stepCustomsInfo.tsx`
  y `stores/create-operation.store.ts` (preselección de `regimeId`, ver
  Decisión 16).

Es una pantalla **nueva y separada**, distinta de:
- El `air-manifest` ya existente (`AirManifestBoard.tsx`, SP-10) — vive
  DENTRO de una Referencia ya creada y rastrea etapas *post-referencia*
  (`RECEIVED → ... → READY_FOR_PREVIO`). El tablero de guías es
  *pre-referencia*.
- "Por identificar" del Inbox (`UnidentifiedWaybill`, `InboxDashboard.tsx`,
  SP-17) — ver Decisión 1 para el porqué de no fusionarlos.

## Decisiones tomadas

1. **No reemplaza "Por identificar"**. Esa sección sigue siendo el camino
   de excepción cuando el matching automático por número de guía
   (`bol-reconciliation.service.ts`) falla y el cliente es una
   incertidumbre real. El tablero nuevo es el flujo normal día a día para
   guías cuyo cliente ya se conoce desde la captura. Por qué: fusionarlas
   sería incorrecto porque `UnidentifiedWaybill` significa "no sabemos de
   quién es" y en este flujo siempre se sabe desde el alta.
2. **Modelo de datos nuevo y propio: `Guia`** (no se reusa
   `UnidentifiedWaybill`). Por qué: la semántica no calza (Decisión 1);
   reusarla generaría acoplamiento con un ciclo de vida
   (`PENDING/LINKED_EXISTING/REFERENCE_CREATED/DISMISSED`) que no aplica
   aquí.
3. **Captura manual en v1, para uso real de producción** (no es maqueta).
   Querétaro necesita trabajar con esto ya; la ingesta automática del
   correo/Excel (tarea 14, sub-plan 02 del paraguas) debe poder alimentar
   el mismo modelo `Guia` sin perder lo capturado a mano.
4. **Volumen bajo** (decenas de guías/día) — tabla simple sin paginación
   agresiva ni carga asíncrona especial en v1.
5. **"Comparación contra prealerta" fuera de este sub-plan** — no existe
   hoy ninguna fuente de datos de prealerta en sigac-3; se documenta como
   pendiente, no bloquea el resto del tablero.
6. **Régimen por guía incluido** (tarea 19 fusionada): al crear
   referencia(s) desde la selección, se agrupa por régimen distinto y se
   genera 1 Referencia/DGO por grupo.
7. **`regimen` es string libre `VARCHAR(3)`** en el schema (igual que
   `Reference.regimen` `schema.prisma:3945` y `Dgo.regimen`
   `schema.prisma:3652`) — no hay catálogo Prisma. Se reutiliza la lista de
   códigos válidos ya definida en el front
   (`carmi-digital/lib/schemas/new-reference-basicData.ts:6`, export
   `Regimen`) como opciones del selector, en vez de duplicarla.
8. **Estatus kanban: set inicial genérico, hipótesis a validar con
   Querétaro** (no hay Excel de referencia todavía):
   `RECIBIDA → EN_REVISION → REGIMEN_ASIGNADO → REFERENCIA_CREADA`. Se
   modela como enum de Prisma, documentado en el propio schema (comentario)
   como sujeto a cambio — ajustar valores exige una migración nueva vía
   CLI.
9. **Cambiar de estatus manualmente exige capturar fecha (no siempre =
   timestamp del cambio) y comentario como mínimo** (regla de negocio de la
   reunión del 20-jul). Se modela con tabla de bitácora
   `GuiaStatusHistory` (`guiaId`, `statusAnterior`, `statusNuevo`, `fecha`,
   `comentario`, `registeredBy`, `createdAt`), no solo un campo de estatus
   plano, para no perder trazabilidad.
10. **Creación de referencia(s) desde selección de guías**: endpoint nuevo
    que agrupa la selección por régimen distinto y llama a
    `ReferencesService.create()` una vez por grupo. No existe hoy ningún
    endpoint batch equivalente — el patrón más cercano
    (`UnidentifiedWaybill POST :id/create-reference`) solo marca una
    referencia YA creada como resuelta, no crea nada. Cada `Guia` de la
    selección queda vinculada (`referenceId`) a la Referencia resultante de
    su grupo, y transiciona a estatus `REFERENCIA_CREADA`.
11. **Fix de bug pre-existente incluido**: `ReferencesService.create()`
    (`references.service.ts:399-528`) recibe `dto.regimen` en el DTO pero
    nunca lo mapea al `data` de creación — solo `update()` lo persiste
    (`references.service.ts:2884`). Sin esta corrección, el régimen
    heredado de la guía se perdería silenciosamente. Se corrige el mapeo en
    `create()`, revisando antes los callers existentes (ver Riesgos).
12. **Migraciones de Prisma solo vía CLI**
    (`npx prisma migrate dev --name ...`), nunca escritas a mano.
13. **Régimen se mantiene en los campos legacy `Reference.regimen`/
    `Dgo.regimen` (VARCHAR(3)), no en `customsRegimeId`**. Investigación de
    código: existen 5 campos de "régimen" en el dominio —
    `Reference.regimen` (el del bug, decisión original), `Reference.
    customsRegime`/`customsRegimeOld` (legacy string, el que sí usa hoy el
    front al editar), `Reference.customsRegimeId`→catálogo `CustomsRegime`,
    `Operation.regime`/`regimeId`→catálogo `Regime`, y `Dgo.regimen`.
    `customsRegimeId` está marcado `@deprecated` en el propio DTO
    (`create-reference.dto.ts:382-390`, comentario "se movió a Operation")
    y su único escritor dedicado (`updateRegimen()`/
    `PATCH /references/:id/regimen`) es código huérfano sin llamador desde
    el front. El régimen "vivo" real vive por `Operation`
    (`Operation.regimeId`→`Regime`), una etapa posterior a la creación de
    `Reference`/`Dgo` que este sub-plan no alcanza (no hay `Operation`s
    todavía en esa etapa). Por qué: mapear al campo deprecado o al
    catálogo de `Operation` en este sub-plan sería prematuro. Se mantiene
    el alcance original (fix del bug en el campo legacy) y se compensa la
    herencia real de régimen con la Decisión 16.
15. **Rediseño 2026-07-23 (post implementación de los pasos 5 y 9) — reemplaza
    la Decisión 10**: la creación automática de Referencia(s) directamente
    desde el tablero se abandona a petición del usuario ("cambiar el flujo
    de creación de referencia a partir de las guías para que me envíe a
    createReference y poder crearla desde ahí y no automático"). El endpoint
    batch `POST /guias/create-references` (`GuiasService.
    createReferencesFromGuias`) se elimina por completo y se reemplaza por
    `POST /guias/link-to-reference` (`GuiasService.linkGuiasToReference`).
    Flujo nuevo: el modal de selección del tablero
    (`CreateReferencesFromGuiasModal.tsx`) ya no crea nada — solo agrupa por
    régimen para mostrar el preview ("se creará 1 referencia con N DGOs") y
    navega al wizard existente
    (`/references/createReference?clientId=...&fromGuiaIds=...`,
    reutilizando el mismo query param `clientId` que ya prellenaba cliente
    en otros flujos). El usuario completa el wizard manualmente (incluyendo
    `Tipo de Operación`, que ya NO se hardcodea — ver nota en Riesgos). Al
    confirmar (`handleFinish`), si viene `fromGuiaIds`, se llama
    `linkGuiasToReference` de forma automática y no bloqueante: agrupa las
    guías por régimen, asigna el primer grupo al DGO default de la
    Reference recién creada (`DgoService.ensureDefault` + `update`) y crea
    un DGO nuevo por cada régimen adicional (`DgoService.
    createAdditionalWithRegimen`, método nuevo — no depende de clave de
    pedimento/facturas como `splitByClave`), vinculando cada `Guia` a la
    Reference (`referenceId` + estatus `REFERENCIA_CREADA` + bitácora). Por
    qué: permite al usuario capturar el resto de los datos de la referencia
    (documentos, tipo de operación real, sucursal, etc.) en vez de una
    creación 100% automática con valores mínimos/hardcodeados.
16. **Preselección de `regimeId` al crear la Operation desde el Dgo**:
    cuando el wizard de creación de operación
    (`app/(customerPortal)/customs-operation/createOperation/**`) arma los
    `pedimentoGroups` a partir de los DGOs seleccionados, debe preseleccionar
    también el `regimeId` — no solo el texto `regime`, que YA se
    preselecciona hoy vía `initializeGroupsFromDgos`
    (`stores/create-operation.store.ts:472-504`). Por qué: cierra el
    círculo de herencia de régimen (Guia→Reference/Dgo→Operation) pedido
    por el usuario; hoy no hay ningún mapeo automático `dgo.regimen→
    regimeId` ni selector de `regimeId` en ese wizard (solo existe en el
    flujo separado `CreateOperationModal.tsx`/`Step3CustomsHeader.tsx`). El
    lookup es directo porque `Regime.code` y `Dgo.regimen` son ambos
    VARCHAR(3) con el mismo formato de código — no requiere mapeo de
    formato, solo `WHERE code = dgo.regimen`.

## Fuera de alcance

- Ingesta automática del manifiesto de correo/Excel línea por línea (tarea
  14, sub-plan 02) — este sub-plan solo captura manual.
- Matching automático Destinatario→Company vía fuzzy/LLM (tarea 15,
  sub-plan 03).
- Entidad `Team` y su vínculo con `Company` (tarea 16, sub-plan 04) — el
  tablero no filtra "por mi equipo" en esta ronda.
- Extender el ciclo de vida propio de `UnidentifiedWaybill` (tarea 17) y el
  puente `UnidentifiedWaybill↔AirManifest` (tarea 18) — sub-plan 05.
- Comparación contra prealerta (dato externo inexistente hoy).
- Cualquier cambio a `InboxDashboard.tsx` / "Por identificar".
- Equivalente marítimo (tarea diferida aparte).
- Campos completos del manifiesto real (Descripción, Peso, Piezas, Bulto,
  Valor, Domicilio Destinatario/Remitente, Vuelo) — en v1 manual se
  capturan solo los campos mínimos necesarios para régimen + creación de
  referencia (ver Pasos); los campos completos llegan con la ingesta
  automática (sub-plan 02).

## Pasos

- [x] **1. Modelo Prisma `Guia` + `GuiaStatusHistory`** — migración vía
  `npx prisma migrate dev --name add-guia-pre-referencia`. Campos mínimos
  de `Guia`: `id`, `companyId` (FK `Company`, cliente conocido desde
  captura), `destinatario` (texto libre), `guiaMaster`/`guiaHouse` (texto,
  opcionales), `regimen` (`VARCHAR(3)`, nullable hasta asignarse), `status`
  (enum `GuiaStatus`), `referenceId` (FK opcional, se llena al crear la
  Referencia), `registeredBy`, `notes`, timestamps. `GuiaStatusHistory`:
  `guiaId`, `statusAnterior`, `statusNuevo`, `fecha`, `comentario`,
  `registeredBy`, `createdAt`.
- [x] **2. Backend: CRUD de `Guia`** — `POST /guias` (alta manual),
  `GET /guias` (listado con filtro por `status`/`companyId`),
  `PATCH /guias/:id` (edición de campos), guardas de autenticación
  consistentes con el resto del módulo (`IdentityJwtGuard`).
- [x] **3. Backend: cambio de estatus con bitácora** —
  `PATCH /guias/:id/status`, DTO exige `status`, `fecha`, `comentario`;
  persiste en `GuiaStatusHistory` y actualiza `Guia.status`.
- [x] **4. Fix del bug de `regimen` en `ReferencesService.create()`** —
  mapear `dto.regimen` al `data` de creación
  (`references.service.ts:399-528`); revisar callers existentes de
  `create()` para confirmar que ninguno depende del comportamiento actual
  (regimen ignorado) antes de aplicar el fix. Agregar test de regresión que
  confirme que `regimen` se persiste desde `create()`.
- [x] **5. Backend: endpoint de vinculación de guías a una referencia** —
  ~~endpoint de creación de referencia(s) desde selección de guías~~
  **reemplazado por la Decisión 15 (rediseño 2026-07-23)**: la Reference ya
  no la crea este endpoint (la crea el usuario en el wizard normal). El
  endpoint final es `POST /guias/link-to-reference`
  (`GuiasService.linkGuiasToReference`): recibe `guiaIds` + `referenceId`
  ya existente, agrupa por `regimen` distinto, asigna el primer grupo al
  DGO default (`DgoService.ensureDefault` + `update`) y crea un DGO nuevo
  por cada régimen adicional (`DgoService.createAdditionalWithRegimen`), y
  en cada `Guia` del grupo escribe el `referenceId` + transición de
  estatus a `REFERENCIA_CREADA` (con su entrada en `GuiaStatusHistory`).
- [x] **6. Front: pantalla del tablero de guías** — tabla nueva (reusa
  `components/ui/table.tsx`) con columnas de guía, cliente, régimen,
  estatus (badge/kanban visual con color), comentarios; entrada nueva de
  navegación (sin tocar el menú ni la lógica de "Por identificar").
- [x] **7. Front: formulario de alta/edición manual de guía** — modal o
  página, campos mínimos del paso 1, selector de cliente (`Company`,
  reusando el patrón `SearchableSelect` + fetch a
  `/companies/clients/importer-exporter` de
  `stepBasicDataReference/index.tsx`, no `companyService`), selector de
  régimen (opciones desde `lib/schemas/new-reference-basicData.ts`).
- [x] **8. Front: acción de cambio de estatus** — modal que exige fecha +
  comentario (permite editar la fecha, no defaultea el timestamp de forma
  forzosa), llama al endpoint del paso 3.
- [x] **9. Front: selección múltiple + "Crear referencia"** — checkbox por
  fila, botón que muestra preview de agrupación por régimen antes de
  confirmar. **Reemplazado por la Decisión 15**: el preview ahora dice "se
  creará 1 referencia con N DGOs (uno por régimen distinto): régimen IMD (3
  guías), régimen ITR (1 guía)" — ya no "N referencias". Al confirmar
  (`CreateReferencesFromGuiasModal.tsx`) navega al wizard de
  `createReference` (`?clientId=...&fromGuiaIds=...`) en vez de llamar un
  endpoint de creación; el endpoint del paso 5 se llama automáticamente al
  terminar el wizard (`handleFinish` en `createReference/page.tsx`), que
  refresca el tablero mostrando el nuevo estatus/referencia vinculada.
- [x] **10. Backend + Front: preselección de `regimeId` en el wizard de
  creación de operación** (Decisión 16) — Backend: al armar
  `pedimentoGroups` desde los DGOs seleccionados (`dgo.service.ts`/
  `operations.service.ts`), resolver `regimeId` por
  `Regime.code = dgo.regimen` (cerca de la lógica existente de
  `validateHomogeneousRegimen`, `dgo.service.ts:435-476`) y devolverlo
  junto con el grupo. Front: extender `initializeGroupsFromDgos`
  (`stores/create-operation.store.ts:472-504`) para guardar el `regimeId`
  resuelto, y añadir un selector de régimen en `stepCustomsInfo.tsx`
  (reusando el patrón de `Step3CustomsHeader.tsx` + `useCustomsRegimes()`)
  preseleccionado con ese valor pero editable.

## Riesgos y side effects a vigilar

- El fix del bug de `regimen` en `create()` (paso 4) puede afectar a otros
  flujos que ya llaman a `ReferencesService.create()` sin pasar `regimen`
  esperando que se ignore — revisar todos los callers antes de mergear, no
  solo el flujo nuevo de este sub-plan.
- Estatus kanban genérico sin validar con Querétaro — probable que necesite
  ajuste temprano; el enum Prisma elegido requerirá una migración nueva si
  cambian los valores, comunicarlo como deuda conocida.
- Convivencia de dos "entidades de guía" en el dominio (`Guia` nueva vs.
  `UnidentifiedWaybill` existente) — documentar la diferencia semántica
  claramente (comentario en el modelo Prisma) para que futuros
  desarrolladores no las confundan ni intenten fusionarlas sin revisar esta
  decisión.
- Ninguna migración de datos existente se ve afectada — `Guia` es tabla
  nueva, no toca datos de `UnidentifiedWaybill`/`AirManifest`.
- El paso 10 toca una pantalla fuera del área original de este sub-plan
  (`customs-operation/createOperation`, no `guias/**`) — mantener el
  cambio acotado a la preselección de `regimeId`, sin tocar el resto del
  wizard ni el flujo paralelo `CreateOperationModal.tsx`/
  `Step3CustomsHeader.tsx` (que ya tiene su propio selector de `regimeId`
  y no necesita cambios).
- No confundir el endpoint huérfano `updateRegimen()`/
  `PATCH /references/:id/regimen` (Decisión 13) con el trabajo del paso
  10 — son mecanismos distintos en modelos distintos (`Reference.
  customsRegimeId` vs. `Operation.regimeId`); no reactivar ni reusar ese
  endpoint huérfano como parte de este sub-plan.
- **Deuda del paso 7 resuelta durante la implementación**: se investigó el
  catálogo `Regime` completo (consulta directa a la BD dev: `DFI, ETE, ETR,
  EXD, IMD, ITE, ITR, RFE, RFS, TRA`) contra la lista estática `Regimen` de
  `lib/schemas/new-reference-basicData.ts` (`BA, BO, K1, H1, A1, A3, C1, C2,
  D1, IN`) — **no coinciden en absoluto** (catálogos distintos, la lista
  estática no sirve para este selector). Además, `customs_regimes` (la
  tabla que sugiere el nombre `useCustomsRegimes()`) está **vacía** en la
  BD dev. Pero el endpoint real detrás de `useCustomsRegimes()`
  (`GET /catalogs/customs-regimes` → `catalogs.service.ts:98`,
  `prisma.regime.findMany`) en realidad consulta el modelo `Regime`, no
  `CustomsRegime` — el nombre es engañoso pero el dato es el correcto y
  coincide exactamente con el catálogo que usará el paso 10. Se usó
  `useCustomsRegimes()` (mismo patrón que `Step3CustomsHeader.tsx`) para el
  selector de régimen del paso 7 en vez de la lista estática, garantizando
  que los códigos que captura el tablero de guías sean válidos para el
  lookup `WHERE code = dgo.regimen` del paso 10.
- **Gap detectado en el paso 5**: `CreateReferenceDto.operationTypeId` es
  obligatorio (TS + class-validator) pero ningún paso del plan lo captura
  desde el tablero de guías. Confirmado con el usuario (22-jul): el
  endpoint de creación de referencia(s) desde selección resuelve siempre
  `operationTypeId` al `OperationType` con `code='I'` (Importación) —
  hardcodeado en `GuiasService.createReferencesFromGuias`, sin exponerlo al
  usuario. También se resuelve `trafficTypeId` al `TransportMode` con
  `code='4'` (Aéreo) por ser el flujo de este sub-plan (opcional, no
  bloquea si el catálogo no lo tiene). Si algún día Querétaro necesita
  crear referencias de **exportación** desde el tablero, este hardcode
  debe revisarse — no lo cubre este sub-plan. **Superado por la Decisión
  15 (rediseño 2026-07-23)**: `GuiasService.createReferencesFromGuias` se
  eliminó por completo; `operationTypeId` ahora lo captura el usuario a
  mano en el wizard de `createReference` (paso "Tipo de Operación"), ya no
  hay ningún hardcode de `code='I'`/Importación en el backend.
- **Agrupación paso 5 es por `(companyId, regimen)`, no solo `regimen`**:
  una `Reference` tiene un único `clientCompanyId` obligatorio, así que dos
  guías con el mismo régimen pero cliente distinto no pueden fundirse en
  una sola `Reference`. En el caso normal (selección de un solo cliente)
  esto se comporta igual que "agrupar solo por régimen", como dice la
  Decisión 10 — la extensión a `companyId` es una corrección técnica
  forzada por el modelo de datos, no un cambio de alcance. **Vigente tras
  la Decisión 15**: `linkGuiasToReference` sigue validando que todas las
  guías seleccionadas pertenezcan al `clientCompanyId` de la Reference ya
  creada (`wrongCompany` check) — incompatibilidad de compañía sigue
  siendo un error, solo que ahora contra una Reference existente en vez de
  al armar los grupos a crear.
- **La vinculación de guías (paso 5, rediseñada por la Decisión 15) no es
  atómica con la creación de la Reference**: la Reference se crea primero,
  de forma independiente, en el wizard normal; `linkGuiasToReference` se
  llama después desde `handleFinish` de forma no bloqueante (si falla, se
  muestra un toast de advertencia pero no se revierte la Reference ya
  creada). Mismo trade-off que documentaba la Decisión 10 original
  (Reference sin guías vinculadas es recuperable manualmente, no
  corrupción de datos), aceptado por el volumen bajo (Decisión 4).
- **Bug ajeno al plan, arreglado durante el paso 6 con autorización del
  usuario**: `lib/api/axios.client.ts` (front) tenía sus 3 interceptores de
  refresh (`apiIdentityClient`, `apiDbClient`, `apiOdinClient`) llamando
  `apiIdentityClient.post('/auth/refresh')`, que con `baseURL: '/api/identity'`
  resolvía a `/api/identity/auth/refresh` — ruta inexistente salvo que
  `NEXT_PUBLIC_CARMI_IDENTITY_API` esté seteada (rewrite en
  `next.config.ts`), y el propio comentario de
  `app/api/auth/refresh/route.ts` dice explícitamente no usar ese rewrite
  para refresh (deja que identity decida las cookie flags). Efecto real:
  toda sesión expiraba cada 15 min sin refrescar (logout forzado en
  cualquier pantalla, no solo `/guias`). Corregido para llamar
  `fetch('/api/auth/refresh', { credentials: 'include' })` directamente,
  igual que ya hace `lib/utils/authToken.ts`. No relacionado con el guard
  `IdentityJwtGuard` usado en el paso 2 (ese sí es el correcto para tokens
  Odin/identity SSO — confirmado comparando con `JwtAuthGuard`, que exige
  `payload.type==='access'` y usuario en la tabla `User` propia de Odin,
  incompatibles con el token SSO descrito por el equipo de auth).
- **Paso 10 — precisión sobre dónde vive `pedimentoGroups`**: no existe un
  "armado de pedimentoGroups" del lado del backend — `pedimentoGroups` es
  100% estado del front (`create-operation.store.ts`), construido por
  `initializeGroupsFromDgos` directamente desde `state.selectedDgos`. Esos
  DGOs vienen de `GET /dgo/selectable` (`DgoService.listSelectableReferences`,
  `dgo.service.ts:371`) — ahí es donde se agregó la resolución de
  `regimeId` (justo antes de `validateHomogeneousRegimen`, como pedía el
  plan), no en `operations.service.ts` (ese archivo solo consume
  `pedimentoGroups` ya armados por el cliente al crear la Operation, no los
  arma).
- **Paso 10 — bug de closure obsoleto evitado en `stepCustomsInfo.tsx`**: el
  `useEffect` que sincroniza el form hacia `customsData` (`watch(...)` →
  `setCustomsData(...)`) tenía deps `[watch, setCustomsData]` (sin
  `customsData`) y llamaba `setCustomsData({...})` con un objeto plano —
  eso reemplaza `customsData` completo, no lo mezcla. Si el nuevo selector
  de régimen hubiera leído `customsData.regime` desde ese closure para
  preservarlo, habría leído un valor obsoleto (el de cuando el efecto se
  montó) y lo habría vuelto a escribir en cada tecleo de cualquier otro
  campo del Step, deshaciendo silenciosamente la selección del usuario. Se
  cambió esa llamada a la forma funcional `setCustomsData(prev => ({ ...prev, ... }))`
  — `prev` siempre es el estado vivo — lo cual de paso corrige el mismo
  riesgo para otros campos no gestionados por ese form (`taxCalculation`,
  `globalIdentifiers`, etc.), sin tocar nada más del wizard.
- **Paso 10 — bug real encontrado y corregido durante la verificación con
  Playwright (2026-07-24): el `regimeId` NO se preseleccionaba**. La
  verificación inicial mostró el selector de "Régimen Aduanero" vacío pese a
  que el DGO tenía régimen asignado; se descartó timing (esperar no lo
  arregló) y se confirmó con `console.log` temporal que `state.customsData`
  llegaba `null` al montar `stepCustomsInfo.tsx`. Causa raíz: el wizard monta
  los 4 tabs (`Referencias/Datos Básicos/Transporte/Despacho`) de entrada —
  no de forma perezosa por paso — así que el `useEffect` que siembra el
  form desde `customsData` corre primero en el montaje inicial (antes de
  que el Step 0 llame a `initializeGroupsFromDgos()`), con
  `activePedimentoGroupIndex` en su valor por defecto `0`. Cuando el Step 0
  sí llama a `initializeGroupsFromDgos()`, en el caso normal de 1 solo grupo
  también deja `activePedimentoGroupIndex` en `0` — el número nunca cambia,
  así que React nunca vuelve a correr el efecto (su única dependencia era
  `[activePedimentoGroupIndex]`) y `customsData`/`regimeId` nunca se
  siembran en el form real. Fix aplicado en
  `app/(customerPortal)/customs-operation/createOperation/components/stepCustomsInfo.tsx`:
  se añadió `activeGroup?.id` al array de dependencias (además del índice)
  — el `id` del grupo sí cambia (de `undefined`/`g-all` a
  `g-dgo-<dgoId>`) cuando `initializeGroupsFromDgos()` corre, disparando el
  re-seed correctamente. Verificado de nuevo con Playwright tras el fix:
  selector muestra "ITR" preseleccionado (checkmark confirmado por
  screenshot) y sigue siendo editable. Este bug es preexistente al patrón
  de montaje-eager del wizard, no introducido por este sub-plan, pero solo
  se manifestó (y se detectó) al agregar este primer consumidor de
  `customsData` que depende de un valor sembrado post-mount con índice de
  grupo sin cambios — vale la pena que futuros campos de este Step que se
  precargan de forma similar revisen esta misma dependencia.

## Criterios de verificación

- ~~Flujo de front original (crear N referencias directo desde el
  tablero)~~ — **reemplazado por la Decisión 15**; ver el nuevo criterio
  de flujo completo abajo.
- **Verificado con Playwright (2026-07-23/24), flujo completo end-to-end
  post-rediseño**: dar de alta 2 guías manualmente con distinto régimen
  (ITR, IMD) para el mismo cliente → aparecen en el tablero con estatus
  `RECIBIDA` → seleccionar ambas → "Crear referencia (2)" → preview
  correcto ("1 referencia con 2 DGOs: régimen ITR, régimen IMD") → "Ir a
  crear referencia" navega al wizard de `createReference` con cliente
  prellenado (`clientId`) → completar el wizard manualmente (Tipo de
  Operación editable, ya no hardcodeado) → al confirmar, `POST
  /references` (201) seguido automáticamente de `POST
  /guias/link-to-reference` (201, no bloqueante) → ambas guías quedan en
  estatus `REFERENCIA_CREADA` en el tablero, con 2 DGOs creados bajo la
  Reference resultante (uno por régimen, vía `ensureDefault`+`update` para
  el primero y `createAdditionalWithRegimen` para el resto). 0 errores de
  consola atribuibles al código nuevo (un 401 transitorio en `/references`
  fue el interceptor de refresh de sesión existente, reintentado con
  éxito — comportamiento preexistente).
- Verificado que "Por identificar" del Inbox sigue funcionando exactamente
  igual (sin regresión) — no se tocó su código.
- Test backend: `ReferencesService.create()` persiste `regimen`
  correctamente — cubierto en `references.service.spec.ts`. Suite completa
  backend: 541/541 test suites, 3774/3780 tests (6 todo preexistentes).
- **Verificado con Playwright (2026-07-24), paso 10**: se firmaron (`POST
  /dgo/:id/sign`) los 2 DGOs de la referencia recién creada para poder
  seleccionarlos en `GET /dgo/selectable`; se creó una Operation
  seleccionando el DGO de régimen ITR → en `stepCustomsInfo.tsx` (tab
  "Aduanas" del paso Despacho) el selector de régimen mostró "ITR"
  preseleccionado (checkmark confirmado por screenshot) y editable. Esta
  verificación expuso y permitió corregir el bug real documentado arriba
  (dependencia de `useEffect` que nunca cambiaba de valor).
- `/verify` corrido (2026-07-24): lint y typecheck limpios en ambos repos
  (0 errores/warnings nuevos — los pocos preexistentes no tocan archivos
  de este sub-plan); test suites completas verdes en ambos repos; flujo de
  front verificado con Playwright como se documenta arriba.

---

## ✅ **COMPLETADO** (2026-07-23)

Todas las tareas del sub-plan SP-01 completadas y verificadas. Diff en rama
`feat/flujo-queretaro-aereo-sp01-tablero-guias` sin commitear, listo para
revisión humana. Bug real encontrado durante paso 10 (preselección de régimen)
y corregido. Sesión sincronizada en [[2026-07-23 - SP-01 paso 10 fix regimeId preselection]].

**Cambios principales:**
1. Tablero de guías con CRUD manual + estatus + bitácora
2. Modal de preview "Crear referencia(s)" con navegación al wizard
3. Wizard de createReference con preselección de cliente y vinculación post-creación
4. Backend: DGO signing, homogeneidad de régimen, `linkGuiasToReference`
5. **Fix crítico**: useEffect dependencies en stepCustomsInfo para re-seed de regimeId
