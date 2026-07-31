# Plan: Unificar wizard de creación de operación en CreateOperationModal (DGO como origen) y mover la Proforma a nivel DGO

## Contexto

Pedido directo del usuario (Ángel, 2026-07-28, chat) — sin ticket ni meet de origen. Área
principal: `carmi-digital` (front); tras auditoría de código previa a `/implementa`
(mismo día), también toca `carmi-odin-api-v2` (back) para las Decisiones 9-12 — ver
"Ramas de trabajo por plan" en CLAUDE.local.md: al iniciar la implementación, crear
`feat/<slug-del-plan>` en ambos repos. Archivos centrales:

- `components/operations/CreateOperationModal.tsx` — wizard reutilizable, hoy de 6 pasos
  (`Step0ShipmentSelector`, `Step1SourceData`, `Step2OperationConfig`, `Step2FiscalConfig`,
  `Step3CustomsHeader`, `Step4PaymentForms`), con loop por grupo de pedimento
  (`activePedimentoGroupIndex`/`pedimentoGroups`).
- `stores/create-operation.store.ts` — compartido entre ambos wizards existentes.
- `app/(customerPortal)/customs-operation/createOperation/page.tsx` — wizard paralelo de 4
  pasos (a eliminar), con `StepDgoSelection.tsx` ya construido en SP-06 para seleccionar
  DGO(s) directamente.
- `app/(customerPortal)/references/components/tabs/ReferenceDGOTab.tsx` — tab DGO, con
  `DgoPedimentoForm` (SP-19, ya construido: aduana/clave/régimen/destino/observations vía
  `PATCH /dgo/:id`, bloqueado si `dgo.locked`) y botón "Crear Operación" por-DGO (a mover).
- `app/(customerPortal)/references/components/drawers/OperationProformaDrawer.tsx` —
  proforma a nivel operación (a eliminar).
- `app/(customerPortal)/references/components/tabs/ReferenceOperations.tsx` — invoca la
  proforma actual (a limpiar).

**Contradice/actualiza decisiones previas ya documentadas en el baúl:** los sub-planes
[[2026-07-10-refactor-flujo-ejecutivo-sp06-wizard-operacion-dgo]],
[[2026-07-10-refactor-flujo-ejecutivo-sp12-tab-operaciones]],
[[2026-07-10-refactor-flujo-ejecutivo-sp15-proforma-pedimento]] (marcado "Implementado") y
[[2026-07-10-refactor-flujo-ejecutivo-sp19-configuracion-pedimento-en-dgo]] (tareas sin
marcar) declaraban `CreateOperationModal.tsx` **fuera de alcance** y establecían el wizard
de 4 pasos como el flujo "real". Verificado en código real que **ambos wizards siguen
vivos** y comparten store. Confrontado con esta contradicción, el usuario decidió invertir
la arquitectura: se elimina el wizard de 4 pasos y `CreateOperationModal.tsx` pasa a ser el
único wizard reutilizable de creación de operación, alimentado por DGOs.

**Hallazgo adicional de la auditoría de código (previo a `/implementa`):** SP-19 ya había
migrado parcialmente dos de los entry points del modal (`ReferenceClassicTable.tsx`,
`ReferenceShipmentsTerrestre.tsx`) para que dejaran de abrir `CreateOperationModal` y en su
lugar navegaran (`router.push`) al wizard de página `customs-operation/createOperation`
(comentarios explícitos en código: "en vez del modal legacy basado en shipments/invoices").
El usuario **confirma revertir esa migración**: estos dos puntos vuelven a abrir
`CreateOperationModal` (ver Decisión 7).

## Decisiones tomadas

1. **Wizard único.** Se elimina `app/(customerPortal)/customs-operation/createOperation/`
   completo (`page.tsx` + `components/` + `schemas/`) después de portar su lógica útil.
   `CreateOperationModal.tsx` queda como el único wizard de crear/editar operación.
   **Por qué:** el usuario prefiere un solo componente reutilizable en vez de dos wizards
   paralelos sobre el mismo store.
2. **Wizard colapsa a un solo pase, sin loop por grupo de pedimento.** Step 0 pasa de
   "seleccionar movimientos" (`Step0ShipmentSelector`) a "seleccionar DGO(s) firmados"
   (portar `StepDgoSelection.tsx`, que ya es prácticamente reutilizable tal cual: usa
   `dgoServices.getSelectableDgos`, `dgoServices.validateDgoSelection`,
   `referenceDocumentsServices.canStartOperation`, y escribe directo a
   `selectedDgos`/`setSelectedDgos` del store). Cada DGO seleccionado = 1 pedimento (ya
   modelado en `pedimentoGroups` vía `initializeGroupsFromDgos`, existente en el store
   desde SP-06). Se elimina `Step2OperationConfig` (agrupar por clave) y el loop de
   re-visita de Steps 3-4-5 por cada grupo (`activePedimentoGroupIndex`, `isLastGroup`,
   botón "Siguiente Pedimento (n de N)"). Wizard resultante: Step 0 Seleccionar DGO(s) →
   Step de datos de operación (fechas, TC, agente, transporte) → Step de formas de pago,
   una sola vez.
   **Por qué:** ahora 1 DGO = 1 pedimento y el DGO ya trae sus propios datos de pedimento;
   no tiene sentido volver a capturarlos ni iterar el wizard por grupo.
3. **Los campos de pedimento (clavePedimento, regimen, destino, observations) no se
   vuelven a capturar en el wizard**: se leen de solo lectura desde cada
   `SelectedDgo`/`pedimentoGroup`. Correcciones se hacen en el DGO (`DgoPedimentoForm`), no
   en el wizard de operación.
   **Excepción confirmada tras auditoría de código:** `aduana` (aduana de despacho) **no**
   es un campo per-DGO en el wizard unificado — el código actual del wizard de página ya lo
   trataba así ("el DGO es la fuente única de verdad"), pero el usuario decide que tanto
   `aduana` de despacho como `patente` (Patente Aduanal, ya operación-completa en el código
   actual) son datos de **operación completa**, capturados una sola vez, no por DGO/pedimento.
3b. **Los campos de operación completa del wizard actual de `CreateOperationModal.tsx`
   (tipo de cambio con modos MANUAL/DAILY y auditoría SPEC-TC-001, transporte vía
   `TransportData`/`useSameTruck`/`containers[]`, aduana de despacho, patente, fechas,
   agente) se dejan tal cual están hoy en el modal.** No se portan ni se adaptan desde
   `stepOperationBasicData.tsx`/`stepTransportConfig.tsx` de la página (que usan modelos más
   simples, p. ej. TC con default hardcodeado `"18.50"` sin sincronizar con el store). El
   usuario confirma que esos campos del modal "ya los revisamos y están bien" — la página
   vieja se elimina sin intentar traer su lógica de estos campos en particular (contradice
   parcialmente el punto 4 original, que hablaba de portar "cualquier dato de operación...
   sin equivalente ya construido"; aquí sí hay equivalente ya construido en el modal y
   gana ese, no el de la página).
4. **Se porta al wizard unificado, antes de borrar la carpeta vieja:**
   - Selección y validación de DGO(s) (`StepDgoSelection.tsx` completo).
   - Cálculo de impuestos `preview-taxes` (hoy en `stepCustomsInfo.tsx:161`, wrapper
     `customsOperationServices.previewTaxes`) — cablear reusando
     `isCalculatingTaxes`/`CustomsData.taxCalculation` ya modelados en el store.
   **Por qué:** esa lógica ya funciona (SP-06) y no se debe reconstruir de cero.
   **Actualizado:** fechas/TC/agente/transporte **no** se portan desde
   `stepOperationBasicData.tsx`/`stepTransportConfig.tsx` — ver 3b: el modal ya tiene su
   propia versión de estos campos, revisada y confirmada por el usuario, y es la que se
   conserva.
5. **Punto de entrada desde el tab DGO:** se elimina el botón "Crear Operación" de cada
   card individual de DGO en `ReferenceDGOTab.tsx`. Se agrega un control global en el tab
   (fuera del Accordion) con checkboxes para marcar uno o varios DGOs firmados, y un botón
   "Crear Operación" que abre `CreateOperationModal` (vía store, no `router.push`) con esos
   DGOs preseleccionados.
   **Por qué:** el usuario quiere poder consolidar varios DGOs en una sola operación desde
   el propio tab.
7. **Entry points existentes de `CreateOperationModal` (confirmado tras auditoría de
   código):**
   - `ReferenceHeroHeader.tsx` (botón "Crear Operación" de la referencia, hoy
     `openModal(reference.id)` sin DGO) y `ShipmentCard.tsx` (botón "Generar Pedimento" por
     shipment, hoy `openModal(referenceId, shipment.id)`) **siguen abriendo el modal**: se
     ajustan para que, al abrir, el usuario seleccione el/los DGO(s) de esa referencia desde
     el nuevo Step 0 (en vez de asumir shipment/reference directo). No navegan a ninguna
     página.
   - `ReferenceClassicTable.tsx` y `ReferenceShipmentsTerrestre.tsx` **habían sido migrados
     en SP-19** a `router.push` hacia `customs-operation/createOperation` (comentarios en
     código: "en vez del modal legacy... desconectado del modelo DGO"). Esta migración se
     **revierte**: ambos vuelven a abrir `CreateOperationModal` (con DGOs preseleccionados
     donde ya se resuelven hoy, p. ej. `shipment.linkedDgos` en
     `ReferenceShipmentsTerrestre.tsx`).
   - `ReferenceOperations.tsx` (`openModalForEdit`, modo edición) no cambia.
   **Por qué:** el usuario confirma que la dirección es consolidar todo en el modal, no en
   la página — la migración parcial de SP-19 hacia la página queda obsoleta.
8. **Proforma:** se elimina la vista de proforma **a nivel operación completa** en
   `ReferenceOperations.tsx` (el botón/invocación que abre hoy `OperationProformaDrawer`
   para ver la proforma consolidada de toda la operación). **Se reutiliza el mismo
   componente/archivo `OperationProformaDrawer.tsx`** (no uno nuevo desde cero — corrige la
   Decisión 6 original), adaptándolo para poder invocarse también **por DGO individual**
   desde `ReferenceDGOTab.tsx`/`DgoActionsDrawer` (pestaña o acción adicional junto a
   Pedimento/Factura(s)/Incrementables/Identificadores), antes de firmar/crear la operación.
   Si el DGO aún no tiene datos capturados (facturas, gastos globales, identificadores,
   campos de pedimento), la proforma se muestra igual con esos campos vacíos/pendientes —
   se llena en vivo conforme se captura el DGO. Datos disponibles vía `GET /dgo/:id`
   (confirmado en `dgo.service.ts:115-131`: ya incluye `invoices+items`, `globalExpenses`,
   `identifiers` — sin cambios de backend para esto) más los campos de `DgoPedimentoForm`.
   **Nota de auditoría:** `OperationProformaDrawer` hoy consulta `GET
   /operations/:id/proforma` (por `operationId`, no por `dgoId`) y ya soporta
   multi-pedimento internamente (`allPedimentos`, `activePedIndex`) aunque los datos de
   cabecera se muestran a nivel operación completa. Adaptarlo a `dgoId` probablemente
   requiere: (a) un modo/prop nuevo que reciba `dgoId` en vez de `operationId` y arme el
   shape `ProformaData` desde `GET /dgo/:id` + `DgoPedimentoForm` cuando la operación aún no
   existe, o (b) si el DGO ya originó un pedimento (`dgo.locked`), seguir resolviendo por
   `operationId`/`pedimentoId` como hoy. Definir esto es parte de la auditoría del primer
   paso de implementación, no un supuesto ya cerrado.
   **Por qué:** el usuario quiere ver la proforma llenarse en vivo mientras se captura el
   DGO, antes de firmar/crear la operación, reusando el componente ya construido en vez de
   duplicar lógica de render/checklist.

**Decisiones añadidas tras auditoría de código previa a `/implementa` (2026-07-28, misma
sesión de revisión de plan):**

9. **Proforma por DGO se resuelve en backend (`carmi-odin-api-v2`), no duplicando cálculo
   en frontend.** Se extrae la lógica de armado de `ProformaData` que hoy vive dentro del
   handler de `GET /operations/:id/proforma` a un método de servicio reusable, parametrizado
   por datos crudos (invoices+items, globalExpenses, identifiers, campos de pedimento) en vez
   de por `operationId`. Nuevo endpoint `GET /dgo/:id/proforma`: si el DGO no tiene operación
   aún, arma la proforma leyendo directo del DGO (mismo método de armado); si el DGO ya está
   `locked` (ya tiene `OperationPedimento`), delega al método existente resolviendo
   `operationId`/`pedimentoId` reales. Una sola fuente de verdad para el cálculo en ambos
   casos. Las 2 acciones de `OperationProformaDrawer` que solo tienen sentido con operación
   real (`regenerate-pedimento`, `request-pedimento`) se deshabilitan/ocultan mientras el DGO
   no esté `locked`.
   **Por qué:** evita divergencia entre dos implementaciones del cálculo de totales/checklist
   (una en backend para operación, otra en frontend para DGO sin operación); no requiere
   migración de Prisma (no hay cambio de esquema, solo servicio/endpoint nuevo).
   **Actualiza el punto "Fuera de alcance" original:** este plan sí toca
   `carmi-odin-api-v2` para este endpoint.
10. **Impuestos por DGO también se resuelven en backend.** Se extiende `PreviewTaxesDto`
    (`carmi-odin-api-v2`) con un filtro opcional `dgoId` que, en vez de agregar todas las
    invoices de la referencia, agregue solo las invoices vinculadas a ese DGO (usando
    `SelectedDgo.invoiceIds`, ya presente en el store del frontend). Mismo motor de cálculo
    de impuestos existente, solo cambia el universo de invoices agregadas — no es un motor
    nuevo.
    **Por qué:** la proforma por-DGO debe poder mostrar impuestos reales, no solo
    "aproximado/por calcular".
    **Actualiza el punto "Fuera de alcance" original** (la limitación de `preview-taxes`
    por-DGO documentada ahí queda resuelta por este endpoint, ya no es una limitación
    aceptada).
11. **Se elimina el chequeo de discrepancias del endpoint de firma.** `POST /dgo/:id/sign`
    (`dgo.service.ts:273-288`) deja de llamar `comparison()`/bloquear por `hasDiscrepancies`
    antes de firmar; el botón "Firmar DGO" (`ReferenceDGOTab.tsx`) deja de estar sujeto a esa
    validación. La validación de discrepancias **no se mueve a ningún otro punto del
    flujo** (ni a la creación de operación): el usuario decide eliminarla por completo, no
    reubicarla.
    **Por qué:** el usuario determina que este chequeo no es necesario.
12. **Se elimina el UUID hardcodeado de `signedBy`.** Hoy el frontend manda
    `signedBy: reference.createdBy` al llamar `POST /dgo/:id/sign`
    (`ReferenceDGOTab.tsx` `signMutation`). Se reemplaza por el usuario real de la sesión,
    obtenido vía `useUser().user.UserID` (`app/context/UserContextProvider.tsx:208`).
    **Por qué:** `signedBy` debe reflejar quién firma realmente, no el creador original de
    la referencia.

## Seguimiento acordado (2026-07-28, tras hallazgo bloqueante de la tarea 5)

El soporte backend para crear una operación solo-por-DGO (sin `shipments`) —
migración de Prisma + `createFromDgos()` + fuente alterna de `pesoBruto`/`totalBultos`,
ver detalle en la tarea 5 — **se aborda como un sub-plan nuevo** vía `/plan` en una
sesión limpia, **al terminar de implementar este plan**, no dentro de esta sesión de
`/implementa`. El guard en `CreateOperationModal.handleSubmit` (tarea 5) deja la
creación de operación bloqueada con mensaje claro hasta que ese sub-plan se
implemente. El resto de las tareas de este plan (6+) no dependen de este pendiente y
continúan normalmente.

## Fuera de alcance

- ~~Cálculo de impuestos preciso por-DGO~~ — **resuelto por la Decisión 10**: sí se
  construye el filtro `dgoId` en `PreviewTaxesDto`. Ya no es una limitación aceptada.
- Migraciones de esquema adicionales (`dgoId` en `OperationPedimento` ya existe desde
  SP-06). Ninguna de las Decisiones 9/10 requiere cambio de esquema Prisma (solo
  servicio/endpoint nuevo) — si durante la implementación se descubriera lo contrario,
  detenerse y usar `npx prisma migrate dev --name <slug>` (nunca escribir SQL de migración
  a mano), o preguntar si el entorno no permite correr la CLI.
- ~~Tocar `carmi-odin-api-v2` salvo hallazgo~~ — **superado por las Decisiones 9, 10, 11 y
  12**: este plan sí toca `carmi-odin-api-v2` de forma planeada (endpoint de proforma
  por-DGO, filtro de impuestos por-DGO, quitar chequeo de discrepancias en `sign()`, quitar
  `signedBy` hardcodeado).
- Resolver `Operation.referenceId` singular en multi-referencia más allá de lo ya resuelto
  en SP-06.
- Editar retroactivamente operaciones ya creadas con el wizard viejo de 4 pasos (quedan
  como están en BD; solo cambia el flujo de creación hacia adelante).

## Pasos

- [x] Auditar `stepCustomsInfo.tsx` (carpeta a eliminar) solo para confirmar el cableado
      exacto de `preview-taxes` a portar (línea ~161). **No** auditar/portar
      `stepOperationBasicData.tsx`/`stepTransportConfig.tsx`: sus campos (TC, transporte,
      fechas, agente) no se portan — el modal ya tiene su propia versión, confirmada como
      correcta (ver Decisión 3b).
      **Confirmado 2026-07-28:** la lógica a portar es `handleCalculateTaxes` (líneas
      134-196) + el bloque UI que la muestra (líneas 987-1039). Calcula `referenceIds`
      desde `selectedDgos` (dedupe por `referenceId`, fallback a `selectedReferences`),
      valida `customsData.customsRegime`/`exchangeRate`, envuelve con
      `isCalculatingTaxes`/`setIsCalculatingTaxes`, calcula `excludedIdentifiers`
      (`availableGlobalIdentifiers` vs `globalIdentifiers`), llama
      `customsOperationServices.previewTaxes({ referenceIds, exchangeRate, paymentDate,
      regime, pedimentoCode, customsOfficeId, selectedGlobalExpenses, excludedIdentifiers
      })`, y mergea `taxCalculation`/`appliedExchangeRate`/`appliedExchangeRateSource` en
      `customsData` vía `setCustomsData` (forma funcional). Todo depende solo de estado ya
      existente en `create-operation.store.ts` — no requiere nada de
      `stepOperationBasicData.tsx`/`stepTransportConfig.tsx`.
- [x] Adaptar Step 0 de `CreateOperationModal` a selección de DGO(s): portar
      `StepDgoSelection.tsx` a `components/operations/steps/`, conectarlo como nuevo Step 0,
      usando `selectedDgos`/`setSelectedDgos` ya existentes en el store.
      **Hecho 2026-07-28:** se copió (no se movió) `StepDgoSelection.tsx` a
      `components/operations/steps/`, dejando el original intacto en
      `customs-operation/createOperation/components/` (todavía usado por `page.tsx`, que se
      elimina en la tarea de "Eliminar `customs-operation/createOperation/`"). Se le agregó
      un prop `onValidationChange` (nuevo, no existía en el original) para exponer al padre
      el estado de validación interno (régimen homogéneo + glosa), que antes era opaco fuera
      del componente. `CreateOperationModal.tsx`: reemplazado el import/uso de
      `Step0ShipmentSelector` por `StepDgoSelection`, agregado `selectedDgos` del store y
      estado local `isDgoSelectionValid`, y la condición `disabled` del botón "Siguiente" en
      `currentStep === 0` ahora es `selectedDgos.length === 0 || !isDgoSelectionValid` (antes
      `selectedShipmentIds.length === 0`). `Step0ShipmentSelector.tsx` se eliminó por quedar
      sin importadores (código muerto). `selectedShipmentIds` se conserva: sigue en uso por
      `Step1SourceData`/`handleSubmit`/`tieneConflictoVaT3` (tareas pendientes de este mismo
      plan). Verificado con `tsc --noEmit` (sin errores) y `eslint` (0 errores, solo warnings
      preexistentes) sobre ambos archivos.
- [x] Eliminar de `CreateOperationModal.tsx` el step de agrupar por clave
      (`Step2OperationConfig` tal como agrupa hoy) y el loop multi-grupo
      (`activePedimentoGroupIndex`, `isLastGroup`, botón "Siguiente Pedimento"), dejando
      `handleNext`/`handlePrev`/`handleSubmit` de un solo pase. Usar
      `initializeGroupsFromDgos` (ya en el store) para construir 1 `pedimentoGroup` por DGO
      seleccionado.
      **Correcciones descubiertas y confirmadas con el usuario 2026-07-28 (el texto original
      de esta tarea tenía el step equivocado):**
      - `Step2OperationConfig.tsx` **no** agrupa nada — es el step "Configuración de
        Operación" (TC, fechas, tipo de operación, agente MOMP, facturación, mandatario),
        exactamente los campos que la Decisión 3b protege. **Se conserva**, ahora corre una
        sola vez (ya no había loop sobre él).
      - El step real de "agrupar por clave" vivía dentro de `Step1SourceData.tsx` → rama
        `MultiShipmentSummary()` (selector Un pedimento/Por factura/Personalizado +
        `CustomGroupEditor`), que dependía de `selectedShipmentIds`/`availableInboundShipments`
        — vacíos en el flujo DGO. Confirmado con el usuario: **se elimina el step
        `Step1SourceData` completo** (ya no aplica, un DGO ya es un pedimento).
      - `Step2FiscalConfig.tsx` (identificadores/gastos globales por pedimento, vía
        `/operations/prepare`) y `Step3CustomsHeader.tsx` (régimen/clave/aduana/destino/
        observaciones por pedimento, dispara `preview-taxes`) **se eliminan ambos**:
        régimen/clave/destino/observaciones ya se capturan en `DgoPedimentoForm` (SP-19);
        identificadores/gastos globales se asumen resueltos a nivel DGO (Decisión 8/9).
      - `aduana` (Aduana de Despacho), que Decisión 1 marca como dato de operación completa
        (no por DGO), no tenía hogar tras eliminar `Step3CustomsHeader`. Confirmado con el
        usuario: se agregó a `Step2OperationConfig.tsx` (nuevo campo + `useCustomsOffices`)
        y a `OperationLevelData` (`customsOfficeId`, con hidratación en modo edición).
      **Wizard resultante (3 pasos):** 0 `StepDgoSelection` → 1 `Step2OperationConfig`
      (incluye aduana) → 2 `Step4PaymentForms` (último paso, submit). `handleNext` en el
      paso 0 llama `initializeGroupsFromDgos()` (salvo edición con grupos ya construidos);
      `handlePrev` es siempre `prevStep()`. El guard de VA (SPEC-VA-001 T3), antes en
      `handleNext` al dejar el viejo paso 3, se movió al inicio de `handleSubmit` (ya no hay
      paso previo al submit donde interceptarlo). Se eliminaron `Step0ShipmentSelector.tsx`
      (task 2), `Step1SourceData.tsx`, `Step2FiscalConfig.tsx`, `Step3CustomsHeader.tsx` —
      sin otros importadores (verificado por grep). `isLoadingInitialData` también se quitó
      por quedar sin lectores tras eliminar `Step1SourceData`.
      **Pendiente para tareas 4/5 de este plan:** `Step4PaymentForms` mostrará "Cálculo
      pendiente" hasta que la tarea 4 cablee `preview-taxes`; `handleSubmit` sigue leyendo
      `customsOfficeId`/`regime`/`pedimentoCode` desde `pedimentoGroups[i].formData.customsData`
      (poblado por DGO vía `initializeGroupsFromDgos`, no desde el nuevo campo operación-nivel)
      — la tarea 5 debe re-cablear esto para usar el `customsOfficeId` de
      `operationLevelData`. Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores,
      solo warnings preexistentes) sobre los archivos tocados.
- [x] Cablear el cálculo de impuestos (`preview-taxes`) portado, dentro del paso
      correspondiente de `CreateOperationModal`, reusando `isCalculatingTaxes`/
      `taxCalculation` del store.
      **Hecho 2026-07-28:** cableado dentro de `Step4PaymentForms.tsx` (último paso del
      wizard, "Formas de Pago" — el mismo lugar donde vivía en `stepCustomsInfo.tsx`).
      Se agregó `handleCalculateTaxes` (misma lógica auditada en la tarea 1) + una card
      "Cálculo de Impuestos (TaxEngine)" con botón y breakdown IGI/IVA/DTA/PRV/CNT + Total,
      antes de la matriz de formas de pago (que ahora solo se muestra una vez calculado).
      Diferencias respecto al original, por el nuevo modelo DGO:
      - `referenceIds` se arma desde `selectedDgos` (dedupe por `referenceId`) — ya no hay
        fallback a `selectedReferences` (ese campo pertenecía al flujo de shipments).
      - `exchangeRate`/`paymentDate`/`customsOfficeId` se leen de `operationLevelData`
        (Step2OperationConfig), no de `customsData` — son datos de operación completa, no
        por pedimento (Decisión 3b/1). `regime`/`pedimentoCode` siguen viniendo de
        `customsData.customsRegime`/`customsKey`, poblados por `initializeGroupsFromDgos`
        desde el DGO.
      - Toasts con `useToast` (consistente con el resto del modal) en vez de `sonner`
        (usado en el archivo original de la página, ya eliminado).
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores ni warnings) sobre
      `Step4PaymentForms.tsx`.
- [x] Ajustar `handleSubmit` de `CreateOperationModal` para construir el payload desde
      `selectedDgos`/`pedimentoGroups` (`dgoId` por grupo) en vez de desde
      shipments/invoices seleccionados a mano.
      **⚠️ Hallazgo bloqueante 2026-07-28 (requiere Decisión/tarea nueva, no cubierta por
      este plan):** auditado `carmi-odin-api-v2` antes de escribir el payload. El backend
      **no soporta crear una operación solo desde DGOs, sin shipments**:
      - `operations.service.ts:905-922` (`create()`): si `shipments` viene vacío (y no hay
        `movementInvoices`), responde `400` antes de tocar cualquier lógica de
        pedimentos/DGO.
      - Esto no es solo una validación de código: `OperationInvoice.operationShipmentId`
        es **`NOT NULL`** (`schema.prisma:2480`), FK a `OperationShipment.shipmentId`
        también `NOT NULL` (`schema.prisma:2465`). `OperationItem`/`OperationInvoice` no
        pueden existir en la BD sin un `Shipment` real — es una restricción de esquema, no
        un guard salteable.
      - Buenas noticias parciales: `Invoice.dgoId` (FK directa, `schema.prisma:1909`) ya
        vincula facturas+items al DGO sin pasar por shipments; `PedimentoShipment` ya
        tolera 0 filas; `Pedimento`/`OperationPedimento`/`PedimentoInvoice` ya funcionan por
        `dgoId`/`invoiceIds` sin depender de shipments (`operations.service.ts:1295-1394`).
      - Pendiente real de backend: migración de Prisma (`OperationShipment.shipmentId`
        nullable, o modelo alterno para `OperationInvoice` sin shipment) + nuevo método de
        servicio `createFromDgos()` (análogo a `createWithMovementInvoices`,
        `operations.service.ts:1960`) + fuente alterna de `pesoBruto`/`totalBultos` en
        `operation-dispatch.service.ts` (hoy suma desde `PedimentoShipment→Shipment`, sería
        `0` sin shipments).
      **Decisión con el usuario:** no bloquear toda la sesión por esto — se implementó todo
      lo que sí es frontend puro y correcto (ver abajo), y se agregó un guard explícito en
      tiempo de ejecución (no un TODO de código) que impide el submit real hasta que el
      backend lo soporte, mostrando un toast claro en vez de dejar que llegue un 400 crudo.
      **Implementado en `CreateOperationModal.tsx`:**
      - `pedimentos` por grupo ahora incluye `dgoId: group.dgoId` y usa
        `group.formData.customsData` directamente (sin el caso especial "grupo activo con
        valores en vivo" del wizard viejo — ya no hay form por-grupo que sincronizar,
        `initializeGroupsFromDgos` deja cada grupo ya poblado desde su DGO).
      - `customsOfficeId` (aduana) ahora se lee de `operationLevelData?.customsOfficeId`
        (dato de operación completa, Decisión 1) en vez de `cd?.customsOfficeId`
        (per-DGO, obsoleto).
      - `paymentForms` usa el cálculo único consolidado (`customsData?.taxCalculation`,
        cableado en la tarea 4) para todos los pedimentos, no un valor por-grupo inexistente.
      - `referenceIds` del payload cae a los `referenceId` distintos de `selectedDgos`
        cuando no hay `modalReferenceIds`/`modalReferenceId` (entry point "crear desde tab
        DGO", Decisión 5 — hallazgo del audit de la tarea 2).
      - `shipments` se construye vacío (ya no hay selección de shipments en el Step 0); justo
        antes de `saveOperationMutation.mutate` hay un guard que bloquea con toast si
        `shipments.length === 0`, explicando el pendiente de backend.
      - Se eliminaron `activePedimentoGroupIndex`/`transportData` del componente (quedaron
        sin lectores tras remover el caso especial de grupo activo).
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores, solo warnings
      preexistentes).
- [x] Eliminar `app/(customerPortal)/customs-operation/createOperation/` completo
      (`page.tsx`, `components/`, `schemas/`) una vez portada su lógica, y confirmar por
      grep que nada más lo importa.
      **Hallazgo 2026-07-28 (no cubierto por la Decisión 7 del plan):** grep encontró 2
      entry points reales navegando a esta página que la auditoría de código previa no
      había detectado — `ReferenceInventoryTable.tsx` (botón "Crear operación con
      selección" desde el tab de inventario) y `OperationClient.tsx` (botón "Nueva
      Operación" de la lista de operaciones, sin ningún parámetro). Confirmado con el
      usuario: se migran ambos a abrir `CreateOperationModal` (mismo tratamiento que la
      tarea 9 para `ReferenceClassicTable`/`ReferenceShipmentsTerrestre`).
      - `create-operation.store.ts`: `openModalMultiSelect(referenceId)` → `referenceId`
        ahora opcional (`referenceId?: string`), ya que Step 0 (`StepDgoSelection`) busca
        DGOs de cualquier referencia por sí mismo, no requiere partir de una referencia.
      - `ReferenceInventoryTable.tsx`: el botón ahora llama
        `openModalMultiSelect(referenceId)` en vez de `router.push(...)`. Se eliminó toda
        la UI de selección de items (checkboxes, "seleccionar todos", contador) — sin
        equivalente en el modelo DGO (preselección de items no aplica), habría quedado
        como UI muerta sin consumidor.
      - `OperationClient.tsx`: se montó `<CreateOperationModal onSuccess={refetch} />`
        (antes no estaba en el árbol de esta página) y el botón "Nueva Operación" ahora
        llama `openModalMultiSelect()` sin argumentos en vez de navegar con `Link`.
      Confirmado por grep: tras estos dos cambios, solo quedan referencias a la ruta vieja
      en `ReferenceClassicTable.tsx`/`ReferenceShipmentsTerrestre.tsx` (tarea 9, pendiente
      en este mismo plan) y `ReferenceDGOTab.tsx` (tarea 7, pendiente) — ambas ya
      contempladas y no un hallazgo nuevo.
      Carpeta eliminada (`page.tsx` + 6 archivos en `components/` + 5 en `schemas/`).
      Verificado con `tsc --noEmit` (0 errores tras limpiar `.next/types`, caché de build
      que aún referenciaba la ruta eliminada — no es código fuente) y `eslint` (0 errores,
      solo warnings preexistentes) sobre los 3 archivos tocados.
- [x] `ReferenceDGOTab.tsx`: quitar el botón "Crear Operación" de cada card de DGO; agregar
      selección múltiple (checkboxes) + botón global "Crear Operación" que abra
      `CreateOperationModal` con los DGOs marcados.
      **Hecho 2026-07-28:** eliminado el botón por-card (antes `router.push` a la página
      borrada en la tarea 6). Agregado control global fuera del `Accordion` (Decisión 5):
      contador de seleccionados + botón "Crear Operación", habilitado solo con selección
      no vacía. Checkbox por DGO **solo** para `status === "SIGNED" && !locked`
      (`selectableDgos`), colocado **fuera** de `AccordionTrigger` (envuelto en un `<div>`
      hermano, no anidado) — Radix `AccordionTrigger` renderiza un `<button>` real; anidar
      un `Checkbox` (también interactivo) adentro habría producido HTML inválido y
      conflicto de eventos (click en el checkbox disparando también el toggle del
      accordion).
      `handleCreateOperationFromSelection` arma `SelectedDgo[]` (id, dgoNumber,
      referenceId/referenceNumber de la referencia actual, regimen, aduana,
      clavePedimento, patente, destino, `invoiceIds` desde `dgo.invoices`), llama
      `openModalMultiSelect(reference.id)` + `setSelectedDgos(selected)` (vía store, no
      `router.push`), y limpia la selección local. El modal ya está montado en
      `references/[id]/page.tsx` (padre de este tab), así que no hace falta montarlo aquí.
      `useRouter`/`router` eliminados (sin otro uso en el archivo).
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores/warnings).
- [x] Eliminar la invocación/botón de proforma a nivel-operación completa en
      `ReferenceOperations.tsx`. **No eliminar el archivo `OperationProformaDrawer.tsx`**:
      se reutiliza y adapta (ver Decisión 8) para aceptar también `dgoId` y armar
      `ProformaData` desde `GET /dgo/:id` + `DgoPedimentoForm` cuando la operación aún no
      existe (DGO sin firmar/sin pedimento aún). Integrarlo en
      `ReferenceDGOTab.tsx`/`DgoActionsDrawer` como pestaña o acción adicional junto a
      Pedimento/Factura(s)/Incrementables/Identificadores.
      **Alcance dividido 2026-07-28 (confirmado con el usuario):** el texto de esta tarea
      duplicaba la tarea 14 ("adaptar `OperationProformaDrawer.tsx` para aceptar `dgoId`...
      consumir `GET /dgo/:id/proforma`"), pero esa adaptación depende de un endpoint que
      todavía no existe — lo crea la tarea 12 (`GET /dgo/:id/proforma`, backend, pendiente).
      Implementar la adaptación ahora habría sido contra un endpoint inexistente. Se hizo
      **solo la parte frontend-pura**: eliminados en `ReferenceOperations.tsx` el item
      "Ver Proforma" del dropdown de acciones, `handleViewProforma`,
      `proformaOperationId` y el montaje `<OperationProformaDrawer operationId={...} />`
      (import de `OperationProformaDrawer` y el ícono `Eye` también removidos, sin otro
      uso). El archivo `OperationProformaDrawer.tsx` **no se tocó ni se eliminó** — la
      adaptación a `dgoId` y su integración en `ReferenceDGOTab`/`DgoActionsDrawer` quedan
      íntegramente para la tarea 14, después de que las tareas 12/13 (backend) existan.
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores/warnings).
- [x] `ReferenceClassicTable.tsx` y `ReferenceShipmentsTerrestre.tsx`: revertir la
      migración de SP-19 — dejan de hacer `router.push` a
      `customs-operation/createOperation` y vuelven a abrir `CreateOperationModal`
      (preseleccionando DGOs donde ya se resuelven, p. ej. `shipment.linkedDgos` en
      `ReferenceShipmentsTerrestre.tsx`).
      **Hecho 2026-07-28:**
      - `ReferenceClassicTable.tsx`: `handleCreateOperationFromReferences` ahora llama
        `openModalFromReferences(selectedIds)` (store, ya existía) en vez de
        `router.push`; se montó `<CreateOperationModal onSuccess={refetch} />` (no estaba
        en el árbol de esta página) y `useCreateOperationStore`/`CreateOperationModal`
        importados. Sin preselección de DGOs aquí (la selección es de referencias, no de
        DGOs) — el usuario los elige en el Step 0 del wizard, igual que antes de SP-19.
      - `ReferenceShipmentsTerrestre.tsx`: `handleCreateOperationFromSelection` resuelve
        `shipment.linkedDgos` de los movimientos seleccionados (dedupe por `dgo.id`,
        conservando `dgoNumber`), arma `SelectedDgo[]` (con `referenceId`/`referenceNumber`
        del prop `reference`; `regimen: null` y el resto de campos de pedimento sin
        rellenar porque `linkedDgos` no los trae — Step 0 revalida régimen/glosa al
        montar, así que no afecta la validación, solo el detalle mostrado antes de esa
        revalidación) y llama `openModalMultiSelect(reference.id)` + `setSelectedDgos(...)`
        en vez de `router.push`. El modal ya estaba montado en `references/[id]/page.tsx`
        (padre de ambos tabs), no hizo falta agregarlo aquí. `useRouter`/`router` se
        eliminaron de este archivo (sin otro uso).
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores, solo warnings
      preexistentes — `ReferenceShipmentsTerrestre.tsx` ya tenía ~45 warnings de
      variables/imports sin usar antes de este cambio, ninguno nuevo introducido).
- [x] `ReferenceHeroHeader.tsx` y `ShipmentCard.tsx`: ajustar para que, al abrir
      `CreateOperationModal`, el Step 0 deje elegir el/los DGO(s) de la referencia/shipment
      en vez de asumir un shipment directo (ya no aplica `openModal(referenceId,
      shipmentId)` como Happy Path que salta a Step 1).
      **Hallazgo 2026-07-28:** `openModal(referenceId, sourceShipmentId)` hacía
      `currentStep = sourceShipmentId ? 1 : 0` — con la renumeración de la tarea 3 (0=DGO,
      1=Config Operación, 2=Formas de Pago), pasar un `sourceShipmentId` aterrizaba
      directo en **Step2OperationConfig**, saltándose la selección de DGO(s) por completo
      y dejando `selectedDgos`/`pedimentoGroups` vacíos — regresión real, no solo el "Happy
      Path" que el texto de la tarea ya anticipaba.
      **Hecho:**
      - `ShipmentCard.tsx` ("Generar Pedimento"): `openModal(referenceId!, shipment.id)` →
        `openModalMultiSelect(referenceId!)`. El `shipment` de este componente está
        tipado `any` y no expone `linkedDgos` (a diferencia de
        `ReferenceShipmentsTerrestre.tsx`, tarea 9) — sin evidencia de ese campo, no se
        preseleccionó ningún DGO; el usuario los elige en el Step 0.
      - `ReferenceHeroHeader.tsx` ("Crear Operación" del header): `openModal(reference.id)`
        ya aterrizaba en step 0 (sin `sourceShipmentId`, sin regresión), pero se cambió
        igual a `openModalMultiSelect(reference.id)` para dejar de usar la acción
        deprecada.
      - `create-operation.store.ts`: la acción `openModal` (marcada
        "DEPRECATED: usar openModalMultiSelect") quedó sin ningún llamador tras estos dos
        cambios (confirmado por grep en todo el repo) — eliminada del store (interfaz +
        implementación) por quedar como código muerto.
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores, solo warnings
      preexistentes) sobre los 3 archivos tocados.
- [x] Actualizar el estado de SP-19 en el baúl (sus tareas de "auditar campos de Aduanas" y
      "arreglar bug de grupos congelados" quedan obsoletas al colapsar el wizard a un solo
      pase) y dejar nota en SP-06/SP-12/SP-15 de que este plan los actualiza/revierte.
      **Hecho 2026-07-28:**
      - **SP-19:** tareas 2 ("auditar campos de Aduanas") y 3 ("arreglar bug de grupos
        congelados") marcadas `[x]` con nota "Obsoleta" explicando por qué (aduana/patente
        ya resueltas como dato de operación completa en `Step2OperationConfig`; el loop
        multi-grupo que originaba el bug de "grupos congelados" se eliminó por completo).
        Nota agregada en su sección "Estado". Tareas 1/4/5 de SP-19 sin tocar (no forman
        parte de este hallazgo).
      - **SP-06:** nota "Revertido/superado" en su "Estado" — el wizard de 4 pasos que
        construyó se eliminó completo; su trabajo de backend (migración `dgoId`,
        `GET /dgo/selectable`, `validateHomogeneousRegimen`, `assertNotLocked`) sigue
        vigente y es la base que este plan reutiliza.
      - **SP-12:** nota en su "Estado" — este plan quitó el botón "Ver Proforma" de
        `ReferenceOperations.tsx` (mismo archivo que SP-12 reusa), sin afectar la
        consulta/agrupación vía DGO que SP-12 implementó.
      - **SP-15:** nota en su "Estado" — la proforma se extiende a nivel DGO (Decisión
        8/9), reusando el mismo `OperationProformaDrawer.tsx` que SP-15 dejó como base (no
        se duplica); pendiente la adaptación a `dgoId` en las tareas 12-14 de este plan.
- [x] `carmi-odin-api-v2`: extraer el armado de `ProformaData` de
      `GET /operations/:id/proforma` a un método de servicio reusable parametrizado por
      datos crudos (no por `operationId`); crear `GET /dgo/:id/proforma` que arme la
      proforma desde el DGO (invoices+items, globalExpenses, identifiers, campos de
      pedimento) cuando no esté `locked`, o delegue al método existente cuando sí lo esté
      (ver Decisión 9). Sin migración de Prisma.
      **BUG IDENTIFICADO (2026-07-30):** el endpoint backend existe y está implementado,
      pero al presionar el botón "Ver proforma" desde la card del DGO en `ReferenceDGOTab`
      (frontend `carmi-digital`) no ocurría nada — el modal de proforma no abría. El
      backend respondía correctamente (confirmado vía curl), pero la invocación desde el
      frontend no llegaba a mostrarse.
      **Causa raíz encontrada y corregida 2026-07-30 (`carmi-digital`), reproducida y
      verificada con Playwright MCP contra `localhost:3000`/`8090`:** no era un problema de
      wiring del botón (`onClick={() => setProformaOpen(true)}` en `ReferenceDGOTab.tsx` ya
      estaba correcto) ni del fetch (la query con `enabled: !!source && open` ya estaba
      bien armada). El bug era un conflicto de foco entre dos `Sheet` (Radix Dialog)
      anidados montados simultáneamente: `OperationProformaDrawer` se abre como Sheet
      hermano **encima** del Sheet "Acciones — DGO #N" (`DgoActionsDrawer`), que sigue
      montado detrás. `OperationProformaDrawer` no aplicaba ninguna mitigación para Sheets
      anidados (a diferencia de `InvoiceFormModal.tsx`/`permit-requests/page.tsx`, que ya
      resuelven exactamente este mismo escenario): al abrir con `modal` default (`true` en
      Radix), su `onOpenAutoFocus` movía el foco al nuevo Portal; el Sheet padre
      ("Acciones"), no-modal, detecta ese foco saliendo de su propio layer como una
      interacción "fuera" y se autocierra — lo que desmonta *todo* el árbol (incluido el
      Proforma que apenas se estaba montando) antes de que llegara a pintarse. Confirmado
      empíricamente con Playwright (`document.querySelectorAll('[role="dialog"]').length`
      pasaba de 1 a 0 justo tras el click en "Ver Proforma", sin que "Proforma de
      Pedimento" llegara a aparecer en el DOM).
      **Fix:** replicado en `OperationProformaDrawer.tsx` el mismo patrón ya establecido en
      el repo para Sheets anidados (`InvoiceFormModal.tsx`, `permit-requests/page.tsx`):
      `modal={false}` + `onOpenChange` que solo actúa sobre el cierre (y limpia
      `document.body.style.pointerEvents` al cerrar, ya que Radix omite esa limpieza en
      modo no-modal) en el `Sheet`; `hideOverlay` + `onInteractOutside`/`onOpenAutoFocus`
      con `preventDefault` en el `SheetContent` — este último es la pieza que evita el
      robo de foco que disparaba el autocierre en cascada.
      Verificado con Playwright MCP (no solo gates estáticos, a diferencia de la
      implementación original de esta tarea en 2026-07-28): firmar/abrir DGO → "Acciones" →
      "Ver Proforma" → el drawer abre con datos reales del DGO sin operación (factura
      PR338698, USD 20,774.49, checklist de validación, "Sin pedimento vinculado — se
      actualiza en vivo"), ambos Sheets (`[role="dialog"]`) presentes en el DOM, 0 errores
      de consola (solo 3 warnings preexistentes: aspect-ratio del logo y
      `aria-describedby` faltante en `SheetContent`, no relacionados con este fix).
      `tsc --noEmit`/`eslint` sobre `OperationProformaDrawer.tsx`: 0 errores, mismos
      warnings preexistentes de `any`.
- [x] `carmi-odin-api-v2`: extender `PreviewTaxesDto` y el servicio de `previewTaxes` con
      un filtro opcional `dgoId` que agregue solo las invoices vinculadas a ese DGO (ver
      Decisión 10). Sin migración de Prisma.
      **Hecho 2026-07-28.** `PreviewTaxesDto` gana `dgoId?: string` (`@IsUUID()
      @IsOptional()`). En `OperationsService.previewTaxes()`, el cálculo de
      `totalCommercialValue` ahora se ramifica: si `dto.dgoId` viene, agrega directo desde
      `Invoice.findMany({where:{dgoId, deletedAt:null}, include:{items}})` — mismo patrón
      directo usado en la tarea 12 (`Invoice.dgoId`, sin pasar por
      shipments/`invoiceShipmentLinks`) — en vez de la ruta `shipmentId`/fallback
      existente. El fallback original (agregar TODAS las invoices de la referencia vía
      shipments) se protegió con `!dto.dgoId` explícito: sin ese guard, un DGO con 0
      invoices habría caído al fallback y agregado de más (todas las facturas de la
      referencia en vez de las del DGO). El resto del motor (TC efectivo, gastos
      incrementables, identificadores activos, `TaxEngine.calculateTaxes`) no se tocó —
      Decisión 10 es explícita en que solo cambia el universo de invoices, no el motor.
      Identificadores (`allIdentifierCodes`) siguen agregándose a nivel referencia, no se
      filtraron por DGO — no estaba pedido por la Decisión 10 (solo habla de invoices).
      Verificado con `tsc --noEmit` (0 errores nuevos, mismos 67 preexistentes de specs/
      tests) y `nest build` (compiló sin errores).
- [x] `carmi-digital`: adaptar `OperationProformaDrawer.tsx` para aceptar `dgoId` (además
      de `operationId`), consumir `GET /dgo/:id/proforma`, y deshabilitar/ocultar
      `regenerate-pedimento`/`request-pedimento` mientras el DGO no esté `locked`.
      **Hecho 2026-07-28.** `OperationProformaDrawerProps` gana `dgoId?: string | null`
      (`operationId` pasa a opcional). El fetch de proforma se ramifica por `source.kind`
      (`'dgo'` → `GET /dgo/:id/proforma`, `'operation'` → `GET /operations/:id/proforma`
      como antes), query key `['proforma', kind, id]`.
      - `isDgoOnlyUnlocked` (derivado: `source.kind === 'dgo' && !proforma.pedimento &&
        allPedimentos.length === 0`) detecta cuándo el DGO aún no originó un pedimento —
        si el backend ya delegó a la operación real (DGO `locked`), `proforma.id` pasa a
        ser el `operationId` real (`getProformaFromDgo` delega completo a `getProforma`,
        tarea 12), de ahí `effectiveOperationId = isDgoOnlyUnlocked ? null : proforma.id`.
      - Mientras `isDgoOnlyUnlocked`: se oculta la pestaña "Documento Anexo 22" (requiere
        pedimento real) y el botón "Solicitar Número" del bloque "Sin pedimento vinculado"
        se reemplaza por un mensaje informativo ("se actualiza en vivo conforme se
        captura"), sin acción — Decisión 8. `regeneratePedimento`/
        `requestPedimentoMutation` usan `effectiveOperationId` (no `operationId` directo),
        así que quedan inertes automáticamente sin `effectiveOperationId`.
      - `SheetDescription` del header ya no depende de `operationNumber` (llega `''` en
        modo DGO no bloqueado desde el backend, lo que rompía el fallback `|| 'Cargando...'`
        de forma silenciosa/permanente) — ahora muestra referencia + "DGO en captura" en
        ese modo.
      **Integración en `ReferenceDGOTab.tsx`/`DgoActionsDrawer`** (Decisión 8, diferida de
      la tarea 8): como **acción adicional** (botón "Ver Proforma" en el header del Sheet
      de Acciones), no como pestaña anidada — `OperationProformaDrawer` ya es un `Sheet` de
      pantalla completa; meterlo como contenido de una `TabsContent` habría anidado un
      Sheet dentro de otro Sheet. El botón abre un segundo `Sheet` independiente
      (`<OperationProformaDrawer dgoId={dgo.id} .../>`, hermano del `Sheet` de Acciones, no
      anidado dentro de su `SheetContent`).
      Confirmado por grep antes de tocar el archivo: `OperationProformaDrawer` se había
      quedado sin ningún consumidor tras la tarea 8 (que quitó su única invocación en
      `ReferenceOperations.tsx`) — esta tarea le da su primer y único consumidor actual.
      Verificado con `tsc --noEmit` (0 errores) y `eslint` (0 errores, solo warnings
      preexistentes de `any`).
- [x] `carmi-odin-api-v2`: en `dgo.service.ts` `sign()` (líneas ~273-288), quitar la
      llamada a `comparison()`/el bloqueo por `hasDiscrepancies` antes de firmar (ver
      Decisión 11). No mover esta validación a ningún otro punto del flujo.
      **Hecho 2026-07-28.** `sign()` ya no llama `comparison()` ni bloquea por
      `hasDiscrepancies`/`requiresManualGlosa` — ahora solo valida existencia
      (`assertExists`, mismo helper ya usado por `manualGlosa`/`splitByClave`) y actualiza
      `status: 'SIGNED'`. `comparison()` en sí **no se tocó** — sigue existiendo y
      sirviendo a `GET /dgo/:id/comparison` (consumido por `DgoComparisonPanel` en
      `ReferenceDGOTab.tsx` para mostrar la comparación, ya no para bloquear firma). La
      validación no se reubicó a ningún otro punto (ni a creación de operación), tal como
      pide la Decisión 11.
      Test desactualizado encontrado y corregido: `dgo.service.spec.ts` tenía un test que
      afirmaba el comportamiento viejo ("blocks signing when there are discrepancies") —
      habría fallado tras este cambio. Reemplazado por dos tests: firma exitosa aun con
      discrepancias (comportamiento nuevo) y `NotFoundException` si el DGO no existe.
      Verificado: `jest src/dgo/services/dgo.service.spec.ts` (18/18 tests pasan),
      `eslint` (0 errores/warnings), `tsc --noEmit` y `nest build` (0 errores).
- [x] `carmi-digital`: en `ReferenceDGOTab.tsx` (`signMutation`), reemplazar
      `signedBy: reference.createdBy` por el `UserID` real de la sesión vía
      `useUser().user.UserID` (`app/context/UserContextProvider.tsx:208`) (ver Decisión 12).
      **Hecho 2026-07-28.** Importado `useUser` de `app/context/UserContextProvider`;
      `signMutation` ahora envía `signedBy: user?.user?.UserID` (mismo patrón
      `user?.user?.UserID` ya usado en `CreateOperationModal.tsx`) en vez de
      `reference.createdBy`. Verificado con `tsc --noEmit` y `eslint` (0 errores).
- [x] Verificar con Playwright el flujo completo: firmar un DGO → seleccionar 1 o varios
      DGOs firmados en el tab → crear operación desde el modal unificado → confirmar
      pedimento(s) con datos correctos del DGO; abrir la proforma por DGO antes de firmar y
      ver que refleja los datos capturados en vivo, con impuestos reales (no aproximados).
      **Hecho 2026-07-28** (subagente `general-purpose` con modelo Haiku, herramientas
      Playwright MCP, contra `localhost:3000`/`8090` ya corriendo). 8/9 pasos PASS, 0
      errores de JavaScript en consola:
      - Firmar DGO sin bloqueo por discrepancias (confirma tarea 15).
      - Selección múltiple → contador + botón "Crear Operación" se habilitan (tarea 7).
      - Wizard abre en Step 0 con los DGO(s) preseleccionados.
      - Step 1 (Configuración de Operación) muestra TC/fechas/agente/**Aduana de
        Despacho** (confirma tarea 3).
      - Step 2 (Formas de Pago): botón "Calcular impuestos" + desglose IGI/IVA/DTA/PRV/CNT.
      - "Crear Operación" final muestra el mensaje de bloqueo esperado (tarea 5) sin
        crash ni 400/500 crudo.
      - "Ver Proforma" desde el DGO abre el drawer con datos reales sin operación creada
        (confirma tarea 14).
      **PARTIAL:** al hacer clic en "Calcular impuestos" el subagente recibió un toast de
      error de validación en vez de un cálculo exitoso — no crasheó, pero no llegó a
      confirmar el caso de impuestos reales calculados con éxito. Causa probable: el
      subagente no llenó todos los campos requeridos (tipo de cambio) en el Step 1 antes
      de calcular — no confirmado como bug del wizard. **Pendiente de confirmar
      manualmente** (o en una verificación posterior con datos de prueba completos) antes
      de dar el flujo de impuestos por completo cerrado.

## Riesgos y side effects a vigilar

- `CreateOperationModal.tsx` y el wizard a eliminar comparten `create-operation.store.ts`;
  revisar que ningún campo/acción usado solo por el wizard viejo se pierda sin portar.
- Los 4 entry points confirmados de `CreateOperationModal` (`ReferenceHeroHeader`,
  `ShipmentCard`, y `ReferenceClassicTable`/`ReferenceShipmentsTerrestre` tras revertir su
  migración a la página) asumen hoy un flujo basado en shipments; cambiar Step 0 a DGO
  puede romperlos si no se ajustan explícitamente los 4.
- La proforma por-DGO (Decisión 8/9) depende de un modo nuevo en `OperationProformaDrawer`
  que arme `ProformaData` sin `operationId` (DGO aún sin operación). El componente hoy
  tiene 4 sitios internos que dependen de `operationId` (pedimento GET, proforma GET,
  regenerate-pedimento POST, request-pedimento POST) — confirmado en auditoría, no solo 1
  como se asumía originalmente. Validar en el primer paso de implementación que la
  extracción del método de armado en backend (Decisión 9) cubre esto sin más refactor del
  previsto.
- Al quitar el chequeo de discrepancias en `sign()` (Decisión 11) sin reubicarlo, un DGO
  con discrepancias factura↔capturado sin resolver podrá firmarse y usarse para crear
  operación sin ninguna advertencia — riesgo aceptado explícitamente por el usuario, no un
  descuido.
- Dejar tareas fantasma en SP-19/SP-06/SP-12/SP-15 en el baúl si no se actualiza su estado
  tras este plan — desalinea `/hoy` y el grafo.

## Criterios de verificación

- Gate estático verde (lint/typecheck) en `carmi-digital`.
- Playwright: crear una operación desde el tab DGO seleccionando 2 DGOs firmados de
  referencias distintas (mismo régimen) → operación creada con 2 pedimentos, cada uno con
  los datos de su DGO, sin ningún step de "configurar pedimento" manual.
- Playwright: abrir la proforma de un DGO en estado DRAFT/IN_REVIEW (aún sin firmar) y
  confirmar que refleja facturas/gastos/identificadores/campos de pedimento capturados
  hasta ese momento, incluyendo impuestos reales calculados vía el nuevo filtro `dgoId`.
- Confirmar por grep que `customs-operation/createOperation/` ya no tiene importadores
  tras el borrado (`OperationProformaDrawer.tsx` se conserva y se reusa, no se borra).
- Confirmar que `POST /dgo/:id/sign` ya no falla por discrepancias y que `signedBy`
  guardado corresponde al usuario real de la sesión, no a `reference.createdBy`.
- Sin errores de consola en ninguno de los flujos anteriores.

## Estado
✅ Implementado (2026-07-28, `/implementa`) — las 17 tareas marcadas `[x]`, más el bug de
"Ver Proforma" (tarea 12) diagnosticado y corregido el 2026-07-30. Las 18 tareas del plan
quedan `[x]`. Repos `carmi-digital`/`carmi-odin-api-v2` en la rama compartida
`feat/unificar-wizard-operacion-dgo-proforma`, sin commitear (queda a revisión humana,
sesión por sesión, tarea por tarea) — el fix del 2026-07-30 está sobre esa misma rama,
sin commit.

**Pendientes reales que sobreviven a este plan:**
- ~~Ver sección "Seguimiento acordado" arriba: soporte backend para crear una operación
  solo-por-DGO (sin `shipments`) — requiere migración de Prisma + `createFromDgos()`,
  bloqueado en la tarea 5. Se aborda como sub-plan nuevo vía `/plan` en sesión limpia.~~
  **Resuelto (2026-07-29/30)** por el sub-plan
  `Planes/2026-07-29-backend-crear-operacion-solo-dgo.md` (15/15 pasos completos, criterios
  de verificación satisfechos): migración de Prisma (`OperationInvoice.operationShipmentId`
  nullable + `operationId`), `createFromDgos()`, fuente alterna de `pesoBruto`/`totalBultos`,
  guard de sync-a-legacy resuelto eliminando por completo la rama legacy de `create()`. El
  guard de `CreateOperationModal.handleSubmit` que bloqueaba la creación ya se quitó. Mismo
  branch compartido `feat/unificar-wizard-operacion-dgo-proforma`, mergeado por el usuario.
- Verificación Playwright (tarea 17): "Calcular impuestos" devolvió un toast de
  validación en vez de un cálculo exitoso — no confirmado si es dato de prueba
  incompleto o un bug real. Repetir la verificación con los campos de TC/régimen
  completos antes de dar el flujo de impuestos por cerrado.
- Tareas 8/14 dejaron `OperationProformaDrawer.tsx` con su único consumidor actual vía
  `dgoId` (`ReferenceDGOTab`); si en el futuro se requiere volver a ver la proforma a
  nivel operación completa, ya soporta `operationId` también (prop opcional).
