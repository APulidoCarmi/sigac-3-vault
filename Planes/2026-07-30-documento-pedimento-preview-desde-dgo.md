# Plan: Ver el formato de pedimento ("Documento Anexo 22") en vivo desde el DGO, antes de crear la operación

## Contexto

Pedido directo del usuario (Ángel, 2026-07-30, chat) — sin ticket ni meet de origen. Surge
al cerrar la sesión de `/implementa` del plan
[[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]] (donde además se
diagnosticó y corrigió el bug del botón "Ver Proforma" que no abría el drawer): el usuario
pidió ver también el formato visual del pedimento mientras el DGO está en captura, sin
operación creada aún.

**Contradice/actualiza una decisión previa del plan anterior:** la Decisión 8 de
[[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]] decidió explícitamente
ocultar la pestaña "Documento Anexo 22" del `OperationProformaDrawer` mientras el DGO no
esté `locked` ("el documento Anexo 22 requiere un pedimento real"). El usuario ahora pide
lo contrario: mostrarla también en modo DGO-preview.

Área principal: `carmi-odin-api-v2` (back, nuevo endpoint) y `carmi-digital` (front, tab
del drawer). Archivos centrales, identificados en auditoría de código previa a este plan
(agente Explore, solo lectura):

- `carmi-odin-api-v2/src/operations/services/operation-dispatch.service.ts`:
  `buildHbsFromPedimento(p, fo)` (línea 5196) arma el objeto `hbsData` que consume el
  template Handlebars, hoy acoplado a un `Pedimento`/`Operation` real vía
  `getOperationPedimento()` (línea 4907, `NotFoundException` si no hay
  `OperationPedimento` — líneas 4911, 4940-5042).
- `carmi-odin-api-v2/src/reports/templates/pedimento/pedimento_completo.hbs`: template
  consumido vía `POST /reports/pedimento/html?tipo=completo`
  (`src/reports/reports.controller.ts:426`); tolerante a campos vacíos/`undefined`
  (Handlebars no rompe, solo imprime cadena vacía).
- `carmi-odin-api-v2/src/operations/services/operations.service.ts:2761-2959`
  (`getProformaFromDgo`): patrón ya existente y validado a reusar — si el DGO no está
  `locked`, arma una "proforma sintética" leyendo `Dgo` + `Dgo.reference` +
  `Dgo.invoices/items/specificAdjustments` + `Dgo.globalExpenses` + `Dgo.identifiers`,
  resolviendo `customsOffice`/`regime` por código y sintetizando un `shipments[]`
  envolvente ficticio (líneas 2883-2906). El endpoint hermano nuevo de este plan
  (`GET /dgo/:id/pedimento-preview`) sigue el mismo patrón: si `locked`, delega a
  `getOperationPedimento` real; si no, arma un `hbsData` sintético.
- `carmi-odin-api-v2` `PreviewTaxesDto`/`OperationsService.previewTaxes()` (ya extendido
  con filtro `dgoId` en la Decisión 10 del plan anterior, tarea 13, 2026-07-28): se reusa
  tal cual para poblar tasas/liquidación del preview, sin motor nuevo.
- `carmi-digital/app/(customerPortal)/references/components/drawers/OperationProformaDrawer.tsx`:
  hoy oculta el `TabsTrigger`/`TabsContent` "documento" cuando `isDgoOnlyUnlocked` (líneas
  ~518-525, ~1177-1239); consume `GET /operations/:id/pedimento` vía
  `effectiveOperationId` (línea ~336, `enabled: !!effectiveOperationId && open &&
  activeTab === "documento"`).
- `carmi-digital/app/(customerPortal)/references/components/drawers/pedimento/PedimentoHtmlViewer.tsx`:
  simple proxy que hace `POST /reports/pedimento/html?tipo=completo` con el `hbsData`
  recibido y muestra el HTML en un iframe `srcDoc` — no valida ni transforma nada, así que
  no necesita cambios.

## Decisiones tomadas

1. **Layout completo con huecos, no ocultar secciones.** El documento preview se ve
   completo, tal cual luciría el Anexo 22, con las secciones que dependen de una operación
   real (pago, número de pedimento, tasas oficiales post-validación, mandatario MOMP)
   vacías/"pendiente" en vez de omitidas.
   **Por qué:** el usuario prefiere ver el layout real desde ya, para acostumbrarse al
   formato conforme captura el DGO, en vez de una vista recortada.
2. **Mismo criterio de aproximación que la proforma por-DGO ya construida** (Decisión 9/10
   del plan anterior): incrementables/decrementables, proveedores y partidas se arman
   desde `Dgo.invoices/items/globalExpenses/identifiers` con la misma lógica que
   `getProformaFromDgo`; peso bruto usa el mismo fallback ya usado ahí
   (`InvoiceItem.grossWeight/netWeight`) cuando no hay `PedimentoShipment` reales.
   **Por qué:** consistencia entre Resumen (proforma) y Documento (pedimento) del mismo
   drawer — evita dos criterios de aproximación divergentes para el mismo DGO.
3. **Banner explícito de "Vista previa"** en el documento cuando se sirve desde
   `pedimento-preview` (DGO sin operación): algo como "Vista previa — DGO sin operación,
   pendiente de validar".
   **Por qué:** evita que alguien confunda este documento con el pedimento oficial ya
   validado/pagado si lo comparte o imprime.
4. **Tasas/liquidación (IGI/IVA/DTA/PRV/CNT) sí se muestran, calculadas automáticamente al
   abrir el drawer** (sin botón manual) usando `previewTaxes` con el filtro `dgoId` ya
   construido (tarea 13 del plan anterior, 2026-07-28). Se resolvió así porque hoy **no
   existe** ningún botón "Calcular Impuestos" en `OperationProformaDrawer` — ese control
   solo vive en `Step4PaymentForms.tsx` del wizard `CreateOperationModal`, un componente
   completamente separado sin conexión de estado con este drawer (hallazgo de la
   entrevista de este plan, corrige una asunción incorrecta inicial de "reusar el cálculo
   ya hecho en Resumen": ese cálculo no existe en el drawer, hay que dispararlo aquí).
   **Por qué:** el usuario quiere ver la liquidación real (aunque preliminar) sin pasos
   manuales adicionales.
5. **Solo visualización en pantalla, sin exportar/descargar/imprimir** — mismo alcance que
   el "Documento Anexo 22" ya tiene hoy para operaciones reales (que tampoco lo soporta).
   **Por qué:** no agregar una funcionalidad que ni siquiera existe para el caso de
   operación real; fuera del pedido original del usuario.

6. **`buildHbsFromDgo` como función hermana, no extracción literal.** `buildHbsFromPedimento`
   resuelve `aduana`/`sección`/`régimen` vía `fo.customsOfficeId`/`fo.regimeRelation` (UUIDs,
   FK a `Operation`); `getProformaFromDgo` resuelve esos mismos campos por **código string**
   (`dgo.aduana`/`dgo.regimen` contra `customsOffice.findFirst({where:{code}})`), porque el
   DGO no tiene esas FKs. El paso 1 no es una extracción mecánica de piezas compartidas: se
   crea `buildHbsFromDgo(dgo, ...)` como función hermana de `buildHbsFromPedimento`, con la
   misma forma de salida (`hbsData`) pero resolviendo aduana/régimen por código (mismo
   patrón que `getProformaFromDgo`).
   **Por qué:** los dos orígenes de datos (Operation real vs DGO crudo) tienen semánticas de
   FK distintas para estos campos; forzar una sola función con ramas condicionales por tipo
   de dato sería más frágil que dos funciones con el mismo contrato de salida.
7. **Limpiar los `console.log` de debug de `buildHbsFromPedimento`** (~20 líneas, ej. 5197,
   5198, 5233-5236, 5242-5249, 5263-5269, 5280, 5300-5301, 5313-5314) como parte del paso 1,
   ya que ese método se toca/extrae de todas formas.
   **Por qué:** evita arrastrar el mismo ruido de logging a los dos caminos (real y
   sintético) una vez que existan ambos.
8. **`GET /dgo/:id/pedimento-preview` devuelve el `hbsData` crudo**, sin wrapping manual ni
   metadata explícita de preview (`isPreview`, etc.) — mismo criterio que el endpoint
   hermano `GET /dgo/:id/proforma` (`getProforma` en `dgo.controller.ts`), que tampoco
   envuelve porque el `TransformInterceptor` global ya lo hace.
   **Por qué:** el frontend ya tiene toda la información que necesita para decidir que está
   en modo preview (`isDgoOnlyUnlocked`/`source.kind === 'dgo'`), no hace falta que el
   backend la repita.

## Fuera de alcance

- Exportar/descargar/imprimir el documento preview (no existe tampoco para operaciones
  reales; no se agrega en ningún caso).
- Campos que dependen 100% de que exista una operación/pedimento real y por tanto **siempre
  quedarán vacíos** en modo preview (auditado en la investigación de este plan):
  `numeroPedimento`, `codigoAceptacion`, datos de "Pago" (`banco`, `linea_de_captura`,
  `importe_pagado`, `numero_de_operacion_bancaria`, `numero_de_transaccion_sat`,
  `fecha_de_pago`), `mandatario`/agente vía `mompCustomsAgent` (en DGO no hay este vínculo
  resuelto), `fecha_entrada` real de `Operation.entryExitDate`, peso/bultos consolidados
  reales desde `PedimentoShipment`/`shipmentContainers`/`packageUnits` (se aproximan desde
  `InvoiceItem`, no se resuelven de forma exacta).
- No se toca el flujo de "Documento Anexo 22" para operaciones ya creadas (DGO `locked`) —
  sigue exactamente igual, resolviendo por `operationId`/`pedimentoId` real vía
  `getOperationPedimento`.
- No se modifica `CreateOperationModal`/`Step4PaymentForms.tsx` (el wizard de crear
  operación no se toca en este plan).
- No se agrega ningún botón manual "Calcular Impuestos" al drawer (Decisión 4: el cálculo
  es automático al abrir).
- No se cambia el template Handlebars (`pedimento_completo.hbs`) ni su tolerancia a campos
  vacíos — ya es suficientemente permisivo.

## Pasos

- [x] `carmi-odin-api-v2`: extraer/generalizar la lógica de `buildHbsFromPedimento` (línea
      5196 de `operation-dispatch.service.ts`) en piezas reusables que puedan alimentarse
      con datos crudos del DGO en vez de solo con un `Pedimento`/`Operation` real —
      específicamente incrementables/decrementables, proveedores y partidas (mismo cálculo
      que `getProformaFromDgo`, líneas 2883-2906 de `operations.service.ts`, ya reusa este
      patrón para la proforma). Dejar explícito en el código qué campos del `hbsData`
      quedan siempre en blanco en modo preview (ver "Fuera de alcance").
- [x] `carmi-odin-api-v2`: crear `GET /dgo/:id/pedimento-preview` (mismo patrón que
      `GET /dgo/:id/proforma`): si el DGO está `locked` (ya tiene `OperationPedimento`),
      delega a `getOperationPedimento`/`buildHbsFromPedimento` reales resolviendo
      `operationId`/`pedimentoId`; si no, arma un `hbsData` sintético desde el DGO usando
      las piezas reusables del paso anterior. Sin migración de Prisma.
- [x] `carmi-odin-api-v2`: dentro del armado del `hbsData` sintético, invocar
      `previewTaxes` con el filtro `dgoId` ya existente (Decisión 10 del plan anterior,
      tarea 13) para poblar la sección de tasas/liquidación (IGI/IVA/DTA/PRV/CNT) del
      documento preview.
- [x] `carmi-digital`: en `OperationProformaDrawer.tsx`, mostrar el `TabsTrigger`
      "Documento Anexo 22" también cuando `isDgoOnlyUnlocked` (quitar la condición
      `!isDgoOnlyUnlocked` de las líneas ~520-525); ajustar la query de `pedimentoData`
      (línea ~328-344) para consumir `GET /dgo/:id/pedimento-preview` cuando
      `source.kind === 'dgo'` sin operación real, en vez de depender exclusivamente de
      `effectiveOperationId`. Mantener el camino actual (`GET /operations/:id/pedimento`)
      sin cambios para DGOs `locked`/operaciones reales.
- [x] `carmi-digital`: agregar el banner "Vista previa — DGO sin operación, pendiente de
      validar" en la pestaña "Documento Anexo 22" cuando se está sirviendo desde el modo
      preview (mismo criterio `isDgoOnlyUnlocked`/`source.kind === 'dgo'` ya usado en el
      resto del drawer).
- [x] Verificar con Playwright: abrir "Ver Proforma" de un DGO sin operación (borrador/en
      captura) → pestaña "Documento Anexo 22" visible con el banner de vista previa;
      secciones con datos capturados (facturas, incrementables/decrementables,
      proveedores, partidas) muestran datos reales del DGO; secciones que dependen de
      operación real (número de pedimento, pago, mandatario) aparecen vacías/"pendiente"
      sin crashear; liquidación con tasas calculadas (IGI/IVA/DTA/PRV/CNT) reales vía
      `previewTaxes`+`dgoId`. Confirmar también que para un DGO `locked` (con operación
      real) el comportamiento no cambió respecto a hoy.

## Riesgos y side effects a vigilar

- Dos rutas de armado de `hbsData` (una para operación real, otra sintética para DGO)
  pueden divergir con el tiempo si se agregan campos nuevos al template y solo se
  actualiza una de las dos — mismo riesgo ya aceptado en la Decisión 9 del plan anterior
  para la proforma (dos rutas de armado, una por escenario).
- Cálculo automático de impuestos al abrir el drawer (Decisión 4) agrega una llamada de
  red extra cada vez que se abre "Ver Proforma" — vigilar que no genere carga/latencia
  perceptible si el usuario abre/cierra el drawer repetidamente sobre el mismo DGO.
- El banner de "Vista previa" debe ser suficientemente visible para no generar confusión
  con el documento oficial — validar visualmente en la verificación Playwright, no solo
  que el texto exista en el DOM.
- Campos que aproximan desde `InvoiceItem` (peso, bultos) pueden no coincidir con el
  peso/bultos reales una vez exista una operación real con shipments — es una
  aproximación aceptada explícitamente (Decisión 2), no un bug a corregir aquí.

## Criterios de verificación

- Gate estático verde (lint/typecheck) en `carmi-digital` y `carmi-odin-api-v2`.
- Playwright: flujo completo descrito en el último paso, sin errores de consola.
- Confirmar que el comportamiento de "Documento Anexo 22" para operaciones ya creadas
  (DGO `locked`) no cambió respecto a antes de este plan.
- Sin errores de consola en el flujo de apertura del drawer ni al cambiar entre pestañas
  "Resumen"/"Documento Anexo 22".

## Estado
✅ Implementado y verificado (2026-07-31) — 6/6 tareas completas, verificación Playwright
en verde para DGO sin operación (preview) y DGO `locked` (real, sin regresión). Diff sin
commitear en ambos repos, rama `feat/documento-pedimento-preview-desde-dgo`, pendiente de
revisión humana. Durante la verificación se encontró y corrigió un bug bloqueante en
`ReferenceDGOTab.tsx` (Sheets anidados) — ver
[[Sheets no-modales anidados y onInteractOutside]] y la bitácora
[[2026-07-31 - Documento pedimento preview desde DGO]].
