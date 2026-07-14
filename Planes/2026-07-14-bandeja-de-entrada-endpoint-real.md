# Plan: Bandeja de Entrada — endpoint real `GET /inbox` y sus 5 secciones

## Contexto

Pedido directo del usuario (Ángel) en sesión de trabajo del 2026-07-14,
continuación de `/implementa` sobre
[[2026-07-10-refactor-flujo-ejecutivo-sp20-ajustes-revision-0713|SP-20]]
(rama compartida `feat/2026-07-13-rediseno-interfaz` en `carmi-digital` y
`carmi-odin-api-v2`).

Durante esa sesión de `/implementa` se detectó que
`ReferencesClient.tsx` (front) apunta hoy a un componente
`InboxDashboard.tsx` — creado hoy fuera de esa sesión, sin commitear — con
5 secciones (`alerts`, `actionableNow`, `automaticCourse`,
`waitingThirdParties`, `unidentified`) que esperan un endpoint `GET /inbox`
en `carmi-odin-api-v2`. Ese endpoint **no existe**: `src/inbox/{controllers,
dtos,services}` están vacíos y no hay `InboxModule` registrado en
`app.module.ts`. El fetch responde 404 y las 5 secciones quedan
permanentemente en "Error al cargar bandeja de entrada" — no es un problema
de mapeo de datos, es que hoy no hay ningún dato detrás.

El usuario definió el significado de negocio de cada apartado en su mensaje
inicial y en la entrevista de este plan (ver Decisiones).

Área de código: back `carmi-odin-api-v2` (nuevo módulo `src/inbox/**`,
reusando `ReferenceDocumentsService.getPendingPanel()`,
`ClassificationItem`, `FundsRequest`, `UnidentifiedWaybillsService`,
`DispatchStep`); front `carmi-digital`
(`app/(customerPortal)/references/components/inbox/InboxDashboard.tsx`, ya
existente, ajustar tipos/paginación).

## Decisiones tomadas

1. **Alertas** = `Reference.eta` próxima a vencer (el usuario usa "eta"
   coloquialmente como "fecha en que la referencia debe salir", no como
   fecha de llegada), **excluyendo** referencias cuyo shipment/operación ya
   se despachó/salió. Umbral de días propuesto por falta de cifra de
   negocio, **ajustable**: rojo si `eta` ya venció o vence en ≤1 día,
   amarillo ≤3 días, naranja ≤7 días. Fuera de esas ventanas no es alerta.
2. **Accionable ahora** = referencias con Operación de despacho activa cuyo
   paso actual (`Operation.currentDispatchStepId` → `DispatchStep.status`)
   está en `PENDING` o `ERROR`. Referencias **sin** operación de despacho
   creada no entran aquí (confirmado explícitamente).
3. **En curso automático** = mismo mecanismo que #2 pero con
   `DispatchStep.status = IN_PROGRESS`. Mismo criterio de exclusión: sin
   operación de despacho, no aparece.
4. **Esperando a terceros** = agregación **por referencia** de 3 fuentes:
   `ClassificationItem.status = PENDING` (clasificación de mercancía),
   `FundsRequest.status = SUBMITTED` (solicitud de fondos sin resolver), y
   `ReferenceLetter.status != SIGNED` (documento solicitado a cliente,
   `PENDING`/`SENT`) — reusando la lógica de
   `reference-documents.service.ts:getPendingPanel()`. Una referencia con
   más de un pendiente aparece en **una sola fila** con badges por tipo de
   pendiente, no una fila por pendiente.
5. **Por identificar** = `UnidentifiedWaybill.status = PENDING`
   (`linkedReferenceId` nulo) — movimientos/guías sin referencia vinculada.
   No se muestran las ya ligadas/con referencia creada/descartadas.
   **Corrección post-verificación (2026-07-14)**: al probar con datos reales
   se detectó que `UnidentifiedWaybill` solo se llena cuando
   `BolReconciliationService` recibe una guía por **email** y no la puede
   emparejar — es una tabla angosta (SP-17 pieza 3). Los `Shipment` sin
   referencia visibles en `/movimientos` → "sin referencia" (creados por
   otra vía: manual, wizard, seed) nunca pasan por ese pipeline, así que
   nunca aparecían en la bandeja aunque existieran en cantidad. **Se amplía
   el alcance**: "Por identificar" agrega también `Shipment` con
   `referenceId = null` (mismo criterio que ya usa `/movimientos` vía
   `withoutReference=true`), fusionado con `UnidentifiedWaybill` en la
   misma lista paginada (`kind: 'shipment' | 'waybill'`, campo ya
   contemplado en `UnidentifiedItemDto` desde el diseño original).
6. **Escala: cientos de registros por sección** en producción → el endpoint
   pagina de verdad (no un preview fijo de 10 sin paginación real). "Ver
   todas" **carga más resultados dentro del mismo panel** ("cargar más"),
   sin navegar a otra pantalla.
7. El pulido visual (colores, densidad, iconografía) se deja para **después**
   de tener datos reales en pantalla — este plan prioriza que el dato sea
   correcto; solo se ajusta el mínimo de UI necesario para soportar
   paginación ("cargar más").

## Dudas resueltas (entrevista previa a /implementa, 2026-07-14)

8. **Exclusión de Alertas ("ya despachado/salió")**: no existe un status
   literal con ese nombre en el schema; había 3 candidatos
   (`Operation.status`, `DispatchStep.status`, `DispatchOperation.status`
   de almacén). Se confirma **`Operation.status` en
   `RELEASED`/`DELIVERED`/`COMPLETED`** — el flujo principal de despacho
   aduanero, consistente con el mismo modelo que usan Accionable ahora/En
   curso automático.
9. **"Esperando a terceros" — agregación cross-referencia**:
   `getPendingPanel()` (`reference-documents.service.ts:699`) está escrito
   para una sola `referenceId`; llamarlo en loop por cada referencia sería
   N+1. Se confirma **escribir una query nueva en `InboxService`** que
   replique la lógica de las 3 fuentes (`ClassificationItem` `PENDING`,
   `FundsRequest` `SUBMITTED`, `ReferenceLetter` status `!= SIGNED`)
   agregada sobre todas las referencias a la vez (3 `findMany` + merge en
   memoria por `referenceId`), **sin** reusar/llamar directamente
   `getPendingPanel()`.
10. **Contrato de paginación de `GET /inbox`**: el front hoy solo manda
    `?limit=10` sin `page` y no tiene lógica real de "cargar más" (el botón
    "Ver todas" solo cambia de tab). El backend ya tiene 2 convenciones
    page-based distintas (`src/common/dtos/pagination.dto.ts` con Max 100,
    y `src/dtos/pagination.dto.ts` sin Max). Se confirma **reusar
    `src/common/dtos/pagination.dto.ts`** (`page`/`limit`, Max 100); cada
    sección de la respuesta de `GET /inbox` trae
    `{ items, total, page, limit, totalPages }`; el front implementa
    "cargar más" incrementando `page` y anexando resultados al panel.

## Fuera de alcance

- El resto de tareas pendientes de SP-20 (reordenar shell del detalle de
  referencia, mover factura manual, datos extraídos en Expediente Aduanero,
  bug de tipo de movimiento por tráfico, validación 1 referencia:1 ticket)
  — siguen su curso en el `/implementa` de ese plan, sin relación con este.
- Definir el umbral exacto de días de Alertas con el negocio (Germán u otro
  stakeholder de producto) — se implementa con el default de la Decisión #1,
  marcado como ajustable, no como cifra definitiva.
- Pulido visual/UX más allá de lo necesario para soportar paginación real —
  se revisita con datos reales en pantalla (Decisión #7).
- Cualquier acción de "resolver" el pendiente desde la bandeja (marcar
  clasificación hecha, firmar carta, liberar fondos, etc.) — la bandeja es
  de solo lectura/navegación hacia la referencia, no un panel de acción.
- Notificaciones/push de alertas — solo la vista dentro de la bandeja.

## Pasos

- [x] Backend: crear `InboxModule` (`src/inbox/{controllers,dtos,services}`)
  con `InboxController` (`GET /inbox`) e `InboxService`; registrar el
  módulo en `app.module.ts`.
- [x] Backend: implementar la sub-query de **Alertas** sobre `Reference.eta`
  con exclusión de referencias ya despachadas/salidas y clasificación de
  severidad por umbral (rojo/amarillo/naranja de la Decisión #1).
- [x] Backend: implementar las sub-queries de **Accionable ahora** / **En
  curso automático** sobre `DispatchStep` vía
  `Operation.currentDispatchStepId` (status `PENDING`/`ERROR` vs
  `IN_PROGRESS`), excluyendo referencias sin operación de despacho.
- [x] Backend: implementar la sub-query de **Esperando a terceros**
  agregando por referencia `ClassificationItem` (`PENDING`), `FundsRequest`
  (`SUBMITTED`) y `ReferenceLetter` (vía `getPendingPanel()` o lógica
  equivalente), una fila por referencia con badges de tipo de pendiente.
- [x] Backend: implementar la sub-query de **Por identificar** sobre
  `UnidentifiedWaybill` (`status = PENDING`), reusando
  `UnidentifiedWaybillsService`.
- [x] Backend: paginación real por sección en el shape de respuesta de
  `GET /inbox` (no solo un preview fijo de 10).
- [x] Front: ajustar `InboxDashboard.tsx` a la forma real de respuesta del
  endpoint (hoy sus tipos son un mock) y corregir los 2 errores de
  TypeScript ya detectados en `SectionState` (líneas ~326/393,
  preexistentes a este plan).
- [x] Front: conectar "Ver todas" a "cargar más" paginado dentro del mismo
  panel; retirar/adaptar el flujo actual de `ReferencesClient.tsx`
  (`handleViewAllFromInbox`, `tablePreset`) que hoy solo cambia de tab sin
  aplicar ningún filtro válido.
- [x] Verificación manual end-to-end con datos reales: al menos un caso de
  prueba por cada una de las 5 secciones en ambiente de prueba.
  (Nota: Alertas y Por identificar verificados con datos reales positivos
  — Por identificar requirió ampliar la Decisión #5, ver arriba, tras
  detectar en esta misma verificación que `UnidentifiedWaybill` estaba
  vacía en este ambiente aunque `/movimientos` sí mostraba movimientos sin
  referencia. Accionable ahora / En curso automático / Esperando a
  terceros se verificaron en su estado vacío correcto, sin datos de
  prueba disponibles en este ambiente que cumplan sus criterios.)

## Riesgos y side effects a vigilar

- La query de "Esperando a terceros" toca 3 tablas distintas
  (`ClassificationItem`, `FundsRequest`, `ReferenceLetter`) — vigilar
  performance con cientos de registros; revisar si los campos de status
  usados ya tienen índice o hace falta agregarlo.
- El criterio de exclusión de Alertas ("ya se despachó/salió") depende de
  qué status de `Operation`/`Shipment` se considera "salió" — confirmar el
  valor exacto a usar al implementar (revisar `OperationStatus`/
  `ShipmentStatus` en el schema, no asumir un nombre de status a ciegas).
- `InboxDashboard.tsx` ya tiene 2 errores de TypeScript preexistentes (no
  introducidos por este plan) que hay que resolver al conectar tipos
  reales — no silenciarlos con `any`/`as` forzado.
- Cambiar el contrato de "Ver todas" (de navegación a otro tab, a "cargar
  más" in-place) toca la relación entre `InboxDashboard` y
  `ReferencesClient.tsx` — revisar que no quede código muerto
  (`handleViewAllFromInbox`, `tablePreset`, `key` de `ReferenceClassicTable`)
  si dejan de usarse.
- El umbral de días de Alertas es un valor propuesto sin confirmar con
  producto — comunicarlo explícitamente al cerrar la implementación, no
  dejarlo pasar como decisión definitiva silenciosa.

## Criterios de verificación

- Puertas estáticas (lint/typecheck/tests) verdes en ambos repos
  (`/verify`).
- Playwright MCP, revisando consola del navegador:
  - `GET /inbox` responde 200 con el shape real (ya no 404) y las 5
    secciones dejan de mostrar "Error al cargar bandeja de entrada".
  - Cada sección muestra al menos un caso de prueba correspondiente a su
    criterio real: una referencia con `eta` próxima sin despachar →
    Alertas; una con `DispatchStep` `PENDING`/`ERROR` → Accionable ahora;
    una con `DispatchStep` `IN_PROGRESS` → Curso automático; una referencia
    con clasificación **y** fondos pendientes → una sola fila en Esperando
    a terceros con ambos badges; una guía sin referencia vinculada → Por
    identificar.
  - "Ver todas" en una sección con más de 10 items carga más resultados en
    el mismo panel, sin navegar fuera de la pantalla.
  - Sin errores nuevos de consola al cargar `/references` con la bandeja
    activa.
