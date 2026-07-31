# Plan: Soporte backend para crear una operación solo-por-DGO (sin shipments)

## Contexto

Sub-plan hijo, pendiente acordado en
[[2026-07-28-unificar-wizard-operacion-dgo-y-proforma-por-dgo]] (sección "Seguimiento
acordado", hallazgo bloqueante documentado en su tarea 5). Se implementa en la **misma
rama** que el plan padre (`feat/unificar-wizard-operacion-dgo-proforma`, ya existente en
`carmi-odin-api-v2` y `carmi-digital`) — **no crear rama nueva** al iniciar `/implementa`.

El wizard unificado (`CreateOperationModal.tsx`) ya arma un payload real desde
`selectedDgos`/`pedimentoGroups` (sin shipments), pero antes de llegar al backend hay un
guard explícito en `handleSubmit` (línea 559,
`carmi-digital/components/operations/CreateOperationModal.tsx`) que bloquea el submit
con un toast si `payload.shipments.length === 0`, con un comentario que documenta el
bloqueo real de backend.

**Área de código (verificado en código real 2026-07-29, no solo por el resumen del plan
padre):**

- `carmi-odin-api-v2/src/operations/services/operations.service.ts`:
  - `create()` (L925-1525): la validación explícita que rechaza el caso sin
    shipments/movementInvoices está en L938-942, pero el bloqueo real es estructural —
    L1084-1110 crea `OperationShipment→OperationInvoice→OperationItem` encadenado a un
    `shipmentId` real; L1360-1385 (`groupShipmentIds`) hace lo mismo para
    `PedimentoShipment` dentro del bloque de creación de pedimentos (L1255-1414).
  - `createWithMovementInvoices()` (L1980-2084): precedente de "Operation sin Shipment"
    vía `MovementInvoice` (N:N Operation↔Invoice), pero deliberadamente simplificado — no
    crea Pedimento/OperationPedimento/PedimentoInvoice, no calcula taxes/customsValue/
    proration de gastos globales, no arma respuesta con relaciones anidadas, no dispara
    los 3 sync a legacy. Sirve de referencia parcial, no de plantilla completa.
  - `getProformaFromDgo()` (L3203-3401): precedente reciente (mismo día del plan padre)
    de armar una respuesta "solo desde DGO sin operación real" — sintetiza un único
    "shipment" envolvente (L3322-3348) para reusar la forma anidada
    `shipments[].invoices[].items[].item` que ya consume el frontend. Es solo-lectura (no
    persiste), pero establece el patrón de compatibilidad de forma de respuesta a seguir.
- `prisma/schema.prisma`: `OperationInvoice.operationShipmentId` (L2480) y
  `OperationShipment.shipmentId` (L2465) son `String` **NOT NULL** — bloqueo real de
  esquema. `PedimentoShipment.shipmentId` (L7570) también NOT NULL. `PedimentoInvoice`
  (L10358-10380) **no** depende de shipment (solo `pedimentoId`+`invoiceId`) — no
  necesita cambio. `Invoice.dgoId` (L1909) ya es FK directa nullable — patrón de
  referencia para el fix elegido (ver Decisión 1).
- `carmi-odin-api-v2/src/operations/services/operation-dispatch.service.ts` (5972
  líneas): calcula `pesoBruto`/`totalBultos` sumando desde
  `PedimentoShipment→Shipment.grossWeight`/`Shipment.packageUnits`/`Shipment.pieces` en
  múltiples puntos (confirmados: L5014-5018, L5376-5380, L5399-5405, L1257-1261,
  L1503/1525, L3071-3096, L4238-4263, L4929-4948). Sin shipments reales esto da 0 (ya
  tolerado, no rompe) salvo el fallback de L5399-5405 que cae a `invoices.length` como
  placeholder de bultos.
  - `InvoiceItem` (schema L1663-1785) sí tiene `grossWeight`/`netWeight`/`weightUnit`
    (L1715/1720/1737) como fuente alterna real de peso. **No** tiene ningún campo
    equivalente a `pieces`/`packageUnits` — no hay fuente alterna real de bultos.
- `carmi-digital/components/operations/CreateOperationModal.tsx` L552-563: guard de
  bloqueo confirmado vigente, con comentario que documenta el mismo hallazgo.

## Decisiones tomadas

1. **Esquema:** `OperationInvoice.operationShipmentId` pasa a nullable y se agrega un FK
   directo `OperationInvoice.operationId` (mismo patrón que `Invoice.dgoId`) para el caso
   sin shipment. Se descarta la alternativa de hacer `OperationShipment.shipmentId`
   nullable y crear filas "virtuales" de `OperationShipment`.
   **Por qué:** el usuario prefiere que "sin shipment" sea explícito en el modelo (FK
   directa) en vez de un shipment falso — más limpio semánticamente, aunque implica
   auditar cada lugar que hoy recorre `operation.shipments[].invoices[]` (ver Decisión 4).
   Migración vía `npx prisma migrate dev --name <slug>` (nunca SQL a mano, regla global).
2. **`totalBultos` sin shipments — REVISADA de nuevo (ver Hallazgos 17-20): se calcula
   real, por DGO específico (no solo por Referencia completa), aéreo ya resuelto para
   todos los casos; terrestre resuelto solo para el caso de 1 DGO por Reference.**
   Decisión original (dejarlo vacío) descartada tras verificar que sí existe una fuente
   real: no se llega por `Shipment`/`InvoiceItem` ligado a la operación, sino por la
   `Reference`/vínculo guía-DGO.
   **Aéreo — resuelto para 1 o varios DGOs por Reference (Hallazgo 20):** sumar
   `Guia.piezas`/`Guia.bultos` de las `Guia` que pertenezcan específicamente a ese DGO.
   Requiere primero que se implemente el pendiente futuro de persistir `Guia.dgoId`
   (Hallazgo 20) — mientras eso no exista, usar como fallback temporal el mismo cruce que
   ya usa el frontend (`Guia.pedimentoCode === Dgo.clavePedimento`, dentro de la misma
   `Reference`), pero con los guardas que el frontend NO tiene (ver Hallazgo 20).
   **Terrestre — solo resuelto para 1 DGO por Reference:** sumar `Shipment.pieces`
   (schema.prisma:7340) de los `Shipment` ligados a esa `Reference` (directo vía
   `Shipment.referenceId`, schema.prisma:7335, o vía `ShipmentReference`,
   schema.prisma:6911-6927) — sin pasar por `OperationShipment`/`PedimentoShipment`. El
   caso de varios DGOs por Reference en terrestre queda fuera de alcance (no se verificó
   un mecanismo de vínculo shipment↔DGO específico análogo al de guías).
   **Por qué:** el usuario prefirió una solución real (vincular guía↔DGO en origen) en
   vez de aceptar el caso borde sin resolver o duplicar la heurística frágil que ya tiene
   el frontend.
3. **`createFromDgos()` con paridad completa respecto a `create()`:** replica el bloque de
   creación de pedimentos reales (`OperationPedimento`, `PedimentoInvoice`), cálculo de
   `customsValue`/taxes (`TaxEngine`), proration de gastos globales — no es un corte
   simplificado tipo `createWithMovementInvoices()`.
   **Por qué:** el frontend ya asume una operación con pedimento real y taxes reales
   (proforma con impuestos vía filtro `dgoId`, acciones `regenerate-pedimento`/
   `request-pedimento` en `OperationProformaDrawer`) — un corte simplificado dejaría el
   flujo a medias otra vez, repitiendo el mismo problema que motivó este sub-plan.
4. **Alcance downstream: auditar y ajustar consumidores conocidos**, no solo el método
   nuevo. Incluye: la forma de respuesta que arma `create()` (`shipments[].invoices[]...`,
   usada por el frontend), los ~8 puntos de `operation-dispatch.service.ts` que asumen
   `PedimentoShipment→Shipment`, y los 3 sync-a-legacy.
   **Por qué:** el usuario prefiere no dejar bugs latentes en producción por invoices sin
   shipment que ningún consumidor sepa leer. Guía de implementación: seguir el patrón ya
   establecido en `getProformaFromDgo()` (shipment sintético envolvente) para mantener el
   contrato de forma que consume el frontend, en vez de inventar una forma nueva.
   **Resuelto — id del shipment sintético (sesión de dudas 2026-07-29):** verificado que
   `getProformaFromDgo()` (`operations.service.ts:3328`) usa `shipment.id = dgo.id` (no
   `null`, sin flag `isVirtual`), y que el único consumidor real de esa forma
   (`OperationProformaDrawer.tsx`) nunca lee `.shipment.id` — solo `.grossWeight` y
   `.invoices` para sumar/agrupar. `createFromDgos()` sigue el mismo patrón
   (`shipment.id = dgo.id`, sin flag nuevo) por consistencia. No agregar un flag
   `isVirtual` preventivo sin evidencia de que haga falta — en vez de eso, la auditoría
   de consumidores downstream de esta misma Decisión 4 debe confirmar explícitamente que
   ninguno lee `.shipment.id` esperando un `Shipment` real; si se encuentra uno que sí,
   ahí se agrega el flag.
5. **No se hace sync a legacy para operaciones DGO-only.** Los 3 side-effects asíncronos
   de `create()` (`syncOperationToLegacy`, `syncPedimentoToLegacy`,
   `syncExpedienteForOperation`, L1483-1522) se saltan explícitamente (guard, no
   silenciado) cuando la operación no tiene shipments reales.
   **Por qué:** el usuario decide no arriesgar sincronizar al legacy datos que ese sistema
   podría no tolerar (operación/pedimento sin shipment real). Queda como pendiente futuro
   explícito habilitar el sync cuando se confirme que el legacy lo soporta — no se
   investiga en este sub-plan.

## Hallazgos y decisiones adicionales (sesión de dudas pre-implementación, 2026-07-29)

Verificados en código real antes de iniciar `/implementa`. Complementan las Decisiones
1-5 sin contradecirlas.

6. **Bug real encontrado: `calculateProrationV2.ts:21` no encontraría items DGO-only.**
   La query busca `OperationItem` navegando `Item→Invoice→OperationShipment→operationId`;
   como las `OperationInvoice` de DGO-only cuelgan directo de `operationId` (Decisión 1,
   sin `OperationShipment`), el prorrateo de gastos globales daría 0 silenciosamente,
   contradiciendo la Decisión 3 (paridad completa, no simplificado). **Fix:** ampliar esa
   query para que también encuentre items vía `OperationInvoice.operationId` directo
   (además de la ruta actual por `OperationShipment`). El nivel de agrupación del
   prorrateo (por Operation/pedimento) no cambia — confirmado con el usuario que "gasto
   global" es a nivel pedimento, no a nivel referencia; la selección de gastos por
   DGO/referencia ya la arma el frontend en `pedimentoGroups` y se reparte igual que hoy
   dentro de cada pedimento creado.
7. **Consumidor adicional no listado originalmente:**
   `funds-requests/services/funds-request-calculator.service.ts:236-246`
   (`calculateOperationWeight`) suma peso solo vía `operationShipment.findMany({ where:
   { operationId } })`, sin fuente alterna. Se trata igual que `pesoBruto` en
   `operation-dispatch.service.ts` (Decisión 2): agregar fuente alterna desde
   `InvoiceItem.grossWeight`/`netWeight` cuando no hay `OperationShipment`.
8. **Validación de input mixto:** `createFromDgos()` rechaza explícitamente (400) si el
   payload trae `dto.shipments` no vacío **y** `dto.pedimentos` con `dgoId` a la vez.
   Defensivo — el wizard actual nunca los mezcla (shipments solo aplica a terrestre, DGOs
   siempre existen), pero es barato de validar y evita bugs silenciosos ante un futuro
   caller que arme el payload distinto.
9. **Gating del flujo: payload-driven, no por modo de transporte.** Se decide por la
   forma del payload (`dto.shipments` vacío/ausente + `dto.pedimentos` con `dgoId` →
   DGO-only), **no** por `Reference.trafficTypeId` ni ningún campo de transporte.
   Verificado que no existe hoy un campo confiable de modo de transporte a nivel
   `Reference`/`Dgo` (`trafficTypeId` es un catálogo genérico marcado
   `@deprecated SPEC-001: Moved to Shipment level`, contradictorio para aéreo porque ahí
   nunca hay `Shipment`). Confirmado con el usuario: para aéreo, la operación **siempre**
   parte del DGO inicial de la referencia, nunca de un "movimiento de entrada"
   (shipment) — DGO-only no es un caso borde para aéreo, es el flujo normal.
10. **Contexto de negocio (no cambia el alcance de este sub-plan):** flujo real para
    aéreo — llegan guías a almacén → se crea la Reference → se suben documentos → el DGO
    inicial de la Reference trae los datos de pedimento (proforma) → al crear la
    operación se llenan los campos reales de operación para los DGOs de la referencia.
    Los "movimientos de entrada" (shipments) quedan como indicador de mercancía que
    llega a almacén solo para terrestre; para aéreo no existen.

11. **Alcance confirmado: universal, no solo aéreo — ya lo es por construcción.** El
    usuario aclaró que la operación debe crearse siempre a partir de DGOs, nunca de
    movimientos de entrada, sin importar el tráfico (terrestre/aéreo/marítimo). Se
    verificó en `carmi-digital/components/operations/CreateOperationModal.tsx` que el
    wizard **ya no captura shipments para ningún tráfico** (L427: `shipments` se declara
    vacío y nunca se puebla; comentario L425-426: "Wizard unificado (DGO): ya no hay
    selección de shipments... este payload siempre viaja sin shipments" — cambio ya
    hecho en el plan padre). Existe un componente separado (`ShipmentCreationModal`) que
    sigue capturando movimientos de entrada, pero vive en otros flujos (tab de
    referencia terrestre `ReferenceShipmentsTerrestre.tsx`, `movimientos/create`), no en
    este wizard. **Conclusión: el gating payload-driven de `createFromDgos()` (Hallazgo
    9) ya cubre todos los tráficos sin trabajo adicional** — no se requiere ningún
    cambio extra de alcance para "migrar terrestre", porque el frontend ya unificó la
    captura para todos los casos.

## Hallazgos y decisiones adicionales (segunda sesión de dudas, 2026-07-29, pre-`/implementa`)

Verificados con dos subagentes de exploración contra el código real. Complementan (y en
el punto 12 **modifican**) las Decisiones 1-5 y Hallazgos 6-11 sin invalidarlos.

12. **Se elimina el requisito de DGO firmado (`SIGNED`) para crear operación —
    cambio de comportamiento también en el `create()` existente, no solo en
    `createFromDgos()`.** Verificado: el gate que hoy exige `Dgo.status === SIGNED` vive
    en `DgoService.validateHomogeneousRegimen()` (`dgo.service.ts:488-528`), llamada desde
    `operations.service.ts` `create()` en L964. Esa misma función hace DOS chequeos en un
    solo método:
    - L500-507: rechaza DGOs que no estén `SIGNED` — **este se elimina**.
    - L509-518: rechaza DGOs ya vinculados a un `OperationPedimento` existente (query
      `operationPedimento.findMany({where:{dgoId:{in:uniqueIds}}}`) — **este se
      conserva**, es el único guard real contra reuso de DGO (no hay `@unique` a nivel de
      schema en `OperationPedimento.dgoId`, solo índice — confirmado, ver Hallazgo 13).
    **Fix:** modificar `validateHomogeneousRegimen()` para quitar el chequeo de `SIGNED`
    (no duplicar ni bifurcar la función) — así `createFromDgos()` puede llamarla tal cual,
    igual que `create()`, y ambos comparten el mismo guard de reuso sin repetir lógica.
    **Por qué:** el usuario decidió que la operación debe poder crearse siempre,
    independientemente del status de firma del DGO — el status de firma ya no es un gate
    de negocio para este flujo.
    **Efecto colateral confirmado y aceptado:** esto relaja `create()` también para el
    caso con shipments (terrestre legacy) — el usuario lo confirmó explícitamente, no es
    un efecto no deseado.
13. **Reuso de DGO: confirmado que hoy solo se previene a nivel de servicio, no de
    schema.** `OperationPedimento.dgoId` (`schema.prisma:2422`) es nullable con solo
    `@@index([dgoId])` (línea 2430), sin `@unique`. El único guard es el
    `operationPedimento.findMany` de `validateHomogeneousRegimen()` (Hallazgo 12).
    `createFromDgos()` debe llamar a esta función (ya sin el chequeo `SIGNED`) para
    heredar el guard de reuso — si no la llama, el reuso de DGO vuelve a ser posible sin
    que nada lo impida.
14. **Alcance confirmado: un `createFromDgos()` puede recibir DGOs de varias
    `Reference` distintas en una sola llamada** — no se restringe a que todos los
    `dgoId` del payload pertenezcan a la misma referencia. No agregar una validación de
    "misma referencia" que no fue pedida.
15. **`dto.pedimentos[].dgoId` ya existe hoy, no es una estructura nueva** (corrige una
    premisa imprecisa del cuerpo original de este plan). Verificado:
    `create-operation.dto.ts` ya define `pedimentos?: PedimentoGroupDto[]` (línea ~698)
    con `dgoId?: string` opcional en `PedimentoGroupDto` (línea ~212), y `create()` ya lo
    consume en el bloque L1255-1414. `createFromDgos()` debe **reusar/parsear el mismo
    campo del DTO**, no inventar una forma nueva de payload para los DGOs.
16. **Decisión: se borra por completo la rama de `create()` que depende de
    `shipments` reales — pero solo al final, después de que `createFromDgos()` esté
    funcionando end-to-end**, no como parte de los primeros pasos. Verificado con
    subagente (2026-07-29): hoy **ningún** caller (frontend ni backend) llega a `create()`
    con `shipments` no vacío — el propio frontend se autobloquea (guard de
    `CreateOperationModal.tsx` L552-565) — por lo que la rama ya es código muerto en
    cuanto a quién la dispara. Pero borrarla *antes* de tener `createFromDgos()` lista
    dejaría el sistema sin ninguna forma de crear operaciones (ni por shipment, porque se
    borró; ni por DGO, porque aún no existe). **Orden obligatorio:**
    1. Construir y probar `createFromDgos()` end-to-end (Pasos existentes).
    2. Quitar el guard del frontend (`CreateOperationModal.tsx` L552-565) y confirmar que
       el flujo real funciona (paso de Playwright ya en el plan).
    3. Solo entonces: borrar de `operations.service.ts` la rama de `shipments` en
       `create()` (bloques L1084-1110 y L1360-1385/`groupShipmentIds`, y cualquier código
       muerto relacionado que quede tras el borrado), y actualizar/borrar el test que la
       cubre directamente: `operations.service.create-pedimentos.spec.ts`, describe block
       `'shipmentIds resolution per group'` (L135-160+).
    **Confirmado que no rompe otros flujos:** `ReferenceShipmentsTerrestre.tsx` /
    `ShipmentCreationModal.tsx` (terrestre) siguen vivos pero para otro propósito —
    gestionar el movimiento físico de entrada a almacén contra el módulo de shipments —
    no llaman a `create()` de operaciones. No se tocan.
    **Por qué:** el usuario decidió que ya no debe existir lógica para crear operaciones
    a partir de shipments (siempre es desde DGO), y el proyecto prohíbe dejar código
    muerto — pero borrar antes de tiempo es más riesgoso que el beneficio de limpieza.
    **Nota de contexto (confirmada con el usuario):** este flujo completo aún no existe
    en producción — estamos en desarrollo. El orden (construir primero, borrar al final)
    no es por miedo a romper producción, es simplemente para no quedarnos sin ninguna
    forma de crear operaciones mientras se termina de construir el reemplazo. El paso de
    borrado final es obligatorio, no opcional — no dejar el código de `shipments` vivo
    "por si acaso".

17. **Fuente real de `totalBultos` confirmada (verificado 2026-07-29, sesión de dudas):**
    - **Aéreo:** modelo `Guia` (`prisma/schema.prisma:7028-7066`) tiene `piezas Int?`
      (L7048) y `bultos Int?` (L7049), ligado a `Reference` vía `referenceId` (L7040,
      relación inversa `Reference.guias Guia[]` en L3980) — 1 Reference : N Guia. `Guia`
      **no** tiene `dgoId` directo; la única ruta es `Dgo.referenceId → Reference ←
      Guia.referenceId`. Servicio real que gestiona esto: `src/guias/services/
      guias.service.ts` (`linkGuiasToReference`, ~L393).
    - **Terrestre:** `Shipment.referenceId` (schema.prisma:7335, FK opcional directa) y/o
      tabla puente `ShipmentReference` (schema.prisma:6911-6927) conectan un `Shipment`
      a una `Reference` **independientemente** de `OperationShipment`/`PedimentoShipment`.
      `Shipment.pieces` (schema.prisma:7340) sigue disponible por esa ruta aunque el
      shipment nunca se ligue formalmente a una operación.
18. **[SUPERADO por el Hallazgo 20 — dejado solo como historial]** Se había encontrado
    que `linkGuiasToReference` agrupaba guías por `Guia.regimen`, ignorando
    `Guia.pedimentoCode`. **Verificado de nuevo en esta misma sesión: esto ya cambió** —
    `linkGuiasToReference` (`guias.service.ts:449-455`) ahora agrupa por
    `guia.pedimentoCode` (el `regimen` solo se deriva para setearlo en cada grupo, ya no
    se usa para decidir cuántos DGOs crear). El comentario del método sigue
    desactualizado (dice "por régimen"), pero el código real ya agrupa por clave de
    pedimento — el cambio pedido por el usuario ya está implementado.
19. **Gap real que persiste tras el Hallazgo 18: la agrupación guía↔DGO no se persiste
    en base de datos.** Verificado: `linkGuiasToReference` arma en memoria
    `dgosCreated.push({ dgo, pedimentoCode, guiaIds: [...] })` (`guias.service.ts:475`)
    con la relación exacta, pero el `$transaction` final (líneas ~476-495) solo hace
    `guia.updateMany({ data: { referenceId, status } })` — nunca escribe el `dgo.id`
    recién creado de vuelta en la `Guia`. No existe `Guia.dgoId` en el schema ni tabla
    puente `GuiaDgo`. Esa relación se calcula una vez y se pierde.
20. **Hallazgo adicional que sí resuelve el caso general, y decisión de arreglarlo en
    origen (persistir `Guia.dgoId`) en vez de reconstruirlo con un cruce frágil:**
    el modelo `Dgo` sí guarda su propia clave de pedimento en `Dgo.clavePedimento`
    (`schema.prisma:3634-3670`, `VarChar(2)`). El frontend (`ReferenceDGOTab.tsx:796-816`)
    ya usa esto para mostrar "qué guías pertenecen a este DGO", cruzando
    `Guia.pedimentoCode === Dgo.clavePedimento` dentro de la misma `Reference` — pero esa
    heurística es cliente-side y tiene dos bugs reales confirmados: (a) nada impide que
    dos DGOs de la misma Reference compartan `clavePedimento` (solo `[referenceId,
    dgoNumber]` es único), causando que la misma guía se cuente en ambos; (b) si
    `clavePedimento` aún no se ha capturado (`null`), el cruce no encuentra nada y el
    bloque de UI simplemente no se muestra (parece "sin guías" sin serlo).
    **Decisión (confirmada con el usuario):** en vez de reusar/duplicar esa heurística
    frágil en el backend, **vincular la guía a su DGO respectivo en el momento en que se
    crea** — agregar `Guia.dgoId` (nullable, FK a `Dgo`) y poblarlo dentro del mismo
    `$transaction` de `linkGuiasToReference` (usando el `dgosCreated`/`guiaIds` que ya se
    calculan en memoria en la línea 475, solo falta persistirlos). Esto resuelve
    `totalBultos` por DGO específico para aéreo en todos los casos (1 o varios DGOs por
    Reference), sin ambigüedad ni casos borde.
    **Por qué:** el usuario prefirió arreglar la causa raíz (falta de persistencia) en
    vez de aceptar el caso borde sin resolver o replicar en el backend una heurística que
    ya se sabe que tiene bugs en el frontend.

### Pendiente futuro explícito (fuera de este sub-plan, requiere su propio `/plan`)

- **Persistir `Guia.dgoId`** (Hallazgo 19-20): agregar el campo (migración Prisma,
  `nunca SQL a mano`), poblarlo en `linkGuiasToReference` con los `guiaIds` que ya se
  calculan en memoria (`guias.service.ts:475`), y hacer que `ReferenceDGOTab.tsx` (front)
  use ese campo real en vez del cruce por `clavePedimento` (arreglando de paso los dos
  bugs del Hallazgo 20). Esto es lo que permitiría a este sub-plan calcular
  `totalBultos` correctamente por DGO en aéreo para el caso de varios DGOs por
  Reference — mientras no exista, `createFromDgos()` debe usar el mismo cruce por
  `clavePedimento` como fallback temporal, aplicando los guardas que el frontend no
  tiene (rechazar o marcar explícitamente el caso de clave duplicada o nula, no
  calcular un número ambiguo en silencio). Requiere su propio `/plan` — módulo distinto
  (`src/guias/`) al de este sub-plan (`src/operations/`).
- Selección de aduana (sucursal) al iniciar sesión, catálogo de qué tipos de tráfico
  maneja cada aduana, y select condicional de tipo de tráfico al crear una `Reference`
  (mostrarlo solo si la aduana del usuario maneja más de un tráfico). Implica eliminar
  el campo actual `Reference.trafficTypeId` y reemplazarlo por este mecanismo nuevo. No
  bloquea ni se toca en este sub-plan — el gating de `createFromDgos()` es
  payload-driven (Hallazgo 9) precisamente para no depender de este campo que va a
  desaparecer.

## Fuera de alcance

- Habilitar el sync a legacy para operaciones DGO-only (Decisión 5) — deshabilitado a
  propósito, pendiente futuro.
- Auditar reportes de despacho o consumidores de `operation-dispatch.service.ts` más allá
  de los puntos ya identificados en esta sesión — si aparece uno nuevo durante la
  implementación, se documenta como hallazgo y se decide entonces si bloquea o no.
- Tocar `createWithMovementInvoices()` (flujo paralelo existente, no relacionado).
- Migrar retroactivamente operaciones ya creadas con shipments reales a este modelo.
- Persistir `Guia.dgoId` y migrar `ReferenceDGOTab.tsx` para usarlo (Hallazgo 19-20) —
  requiere su propio `/plan`, módulo `src/guias/`, no se toca en este sub-plan. Mientras
  no exista, `createFromDgos()` usa el cruce por `clavePedimento` como fallback (con
  guardas, ver Decisión 2).
- Calcular `totalBultos` correctamente por DGO en **terrestre** cuando una `Reference`
  tiene más de un DGO/pedimento simultáneo — no se verificó un mecanismo de vínculo
  shipment↔DGO específico (análogo a `Guia.dgoId`) para ese caso; queda fuera de
  alcance de este sub-plan.
- Rediseñar `PedimentoShipment` (ya tolera 0 filas sin romper, confirmado en auditoría —
  no requiere cambio).

## Pasos

- [x] Migración de Prisma: `OperationInvoice.operationShipmentId` nullable +
      `OperationInvoice.operationId` (FK directa a `Operation`, nullable). Vía
      `npx prisma migrate dev --name make-operation-invoice-optional-shipment`. Verificar
      que el esquema resultante permite ambos casos (invoice colgada de shipment, o
      directa de operación) sin romper las relaciones inversas existentes de
      `Operation`/`OperationShipment`/`Invoice`.
- [x] `operations.service.ts`: nuevo método `createFromDgos()` (o rama dentro de
      `create()`, análoga a la de `movementInvoices` en L926-934) que, para
      `dto.pedimentos` con `dgoId` y sin `dto.shipments`, cree la `Operation` +
      `OperationInvoice` directas (`operationId`, sin `operationShipmentId`) +
      `OperationItem` + pedimentos reales (`OperationPedimento`, `PedimentoInvoice`) +
      taxes/customsValue/proration — replicando el bloque L1036-1422 de `create()` pero
      sin la parte que depende de `OperationShipment`/`groupShipmentIds`/
      `PedimentoShipment`.
- [x] Ajustar la construcción de la respuesta (equivalente al `findUnique` de
      L1427-1464) para que incluya las invoices colgadas directo de `operationId`, con la
      forma que el frontend ya consume — seguir el patrón de shipment sintético
      envolvente ya usado en `getProformaFromDgo()` (L3322-3348) para no romper el
      contrato `shipments[].invoices[].items[].item`.
- [x] **Hallazgo post-Paso 3 (2026-07-29, no estaba en la lista original):**
      `OperationsService.findOne()` (`operations.service.ts:2549`, detrás de
      `GET /operations/:id`) y `OperationsService.getProforma()` (usado por
      `OperationProformaDrawer` y por `getProformaFromDgo()` cuando el DGO ya tiene
      operación real) armaban su respuesta solo desde `include: { shipments: {...} }` —
      no incluían `operationInvoices` colgadas directo de `operationId`. Sin este ajuste,
      una operación DGO-only creada por `createFromDgos()` se vería sin facturas/items al
      recargarla, editarla o abrir su drawer de proforma (aunque la respuesta inmediata de
      creación, Paso 3, sí las trae). **Implementado:** ambos métodos ahora incluyen
      `operationInvoices` (con `invoice.dgoId`, `items.item`) y, cuando `shipments` viene
      vacío, sintetizan un shipment envolvente por DGO (`shipment.id = dgoId`) agrupando
      por `invoice.dgoId` — mismo patrón que `createFromDgos()`/`getProformaFromDgo()`. Hoy
      son mutuamente excluyentes (una operación tiene shipments reales O operationInvoices
      directas, nunca ambas); si eso cambiara habría que fusionar en vez de reemplazar.
      Typecheck verde.
- [x] `operation-dispatch.service.ts`: ajustar los puntos que calculan `pesoBruto`
      (agregar fuente alterna desde `InvoiceItem.grossWeight`/`netWeight` cuando no hay
      `PedimentoShipment`) y `totalBultos` (Decisión 2 revisada — Hallazgo 20):
      - Aéreo: sumar `Guia.piezas`/`Guia.bultos` de las guías del DGO específico. Usar
        `Guia.dgoId` si ya existe (Hallazgo 19-20, pendiente futuro en `src/guias/`); si
        no existe todavía, usar como fallback `Guia.pedimentoCode === Dgo.clavePedimento`
        dentro de la misma `Reference`, con guardas explícitas: si hay ambigüedad (dos
        DGOs de la misma Reference comparten `clavePedimento`) o `clavePedimento` es
        `null`, NO calcular un número silencioso — dejar el bulto explícitamente sin
        determinar para ese DGO y documentarlo, igual que se decidía en la versión
        anterior de esta decisión.
      - Terrestre: sumar `Shipment.pieces` vía `Shipment.referenceId`/`ShipmentReference`
        de esa `Reference` — sin pasar por `OperationShipment`/`PedimentoShipment`. Solo
        válido para el caso de 1 DGO por Reference (ver Fuera de alcance).
      - Quitar el fallback `invoices.length` de L5399-5405 y reemplazarlo por esta
        lógica real.
      Revisar los demás puntos identificados (L1257-1261, L1503/1525, L3071-3096,
      L4238-4263, L4929-4948) uno por uno para confirmar cuáles aplican al caso DGO-only.
- [x] Guard explícito en `create()`/`createFromDgos()`: saltar `syncOperationToLegacy`,
      `syncPedimentoToLegacy`, `syncExpedienteForOperation` cuando la operación no tiene
      shipments reales (Decisión 5). Dejar comentario claro (no un TODO ambiguo)
      explicando por qué y qué falta para habilitarlo.
      **Resuelto de forma más completa de lo previsto** (Step 11, con aprobación explícita
      del usuario para ampliar el alcance): en vez de un guard condicional, se eliminó por
      completo la rama legacy de `create()` que llamaba a los tres métodos de sync —
      `syncOperationToLegacy`, `syncPedimentoToLegacy` y `syncExpedienteForOperation` ya no
      existen en el archivo (confirmado con `grep -rn` sobre todo `src/`). `create()` ahora
      es un router puro: `movementInvoices` → `createWithMovementInvoices()`, pedimentos con
      `dgoId` → `createFromDgos()`, cualquier otro caso → 400. No queda ningún camino de
      código que pueda invocar el sync legacy, así que no hace falta guard: no hay nada que
      saltar. `createFromDgos()` (línea 1499 de `operations.service.ts`) conserva el
      comentario explícito original documentando por qué no sincroniza a legacy y qué falta
      para habilitarlo en el futuro.
- [x] `operations.service.calculateProrationV2.ts` (L21): ampliar la query de items a
      prorratear para que también encuentre `OperationItem` vía
      `OperationInvoice.operationId` directo, además de la ruta actual por
      `OperationShipment` (Hallazgo 6). No cambiar el nivel de agrupación del prorrateo.
- [x] `funds-requests/services/funds-request-calculator.service.ts` (L236-246,
      `calculateOperationWeight`): agregar fuente alterna desde
      `InvoiceItem.grossWeight`/`netWeight` cuando no hay `OperationShipment` (Hallazgo 7,
      mismo criterio que Decisión 2/pesoBruto).
- [x] `createFromDgos()`: validación explícita (400) si el payload trae `dto.shipments`
      no vacío y `dto.pedimentos` con `dgoId` a la vez (Hallazgo 8, input mixto). El DTO
      (`create-operation.dto.ts` L628, `shipments`) debe volverse opcional a nivel
      decorador para permitir el flujo DGO-only.
- [x] `carmi-digital`: quitar el guard de bloqueo en `CreateOperationModal.tsx`
      (L552-563) ahora que el backend soporta el flujo. Typecheck/lint verdes. La prueba
      end-to-end real contra el backend se hace en el Paso 9 (Playwright), no aquí.
- [x] **Hallazgo post-Paso 8 (2026-07-30, no estaba en la lista original):**
      `ReferenceDGOTab.tsx` (`selectableDgos`, `canSelect`) seguía ocultando el checkbox
      de selección para DGOs sin firmar, aunque el backend (Paso 10) ya no lo exige.
      **Implementado:** ambos filtros ahora son solo `!dgo.locked`; texto de ayuda
      actualizado ("Marca uno o varios DGO(s) para crear una operación"). Typecheck/lint
      verdes.
- [x] **Verificación con Playwright (2026-07-30):** referencia REF-00073 (DGO recién
      creado por el tab, status `DRAFT` sin firmar — capturé régimen IMD/clave A1/aduana
      200 vía el modal de "Acciones" del DGO, que no existían aún en este ambiente de
      dev). Seleccioné el DGO desde el checkbox de `ReferenceDGOTab.tsx` (ya sin el gate
      SIGNED, hallazgo anterior) y completé el wizard unificado (Aduana, Tipo de
      Operación, Agente Aduanal, cálculo de impuestos) hasta enviar.
      - `POST /operations` → **201 Created**, `OP-00041`, `customsValue: 166038.03` real
        (no 0), `taxesCalculated` con desglose real IGI/IVA/DTA/PRV/CNT vía TaxEngine,
        `pedimentoId` poblado, respuesta con `shipments[0].shipment.id` = id del DGO
        (shipment sintético) conteniendo las 2 invoices reales con sus items.
      - DB: `operation_invoices` con `operation_id` poblado y `operation_shipment_id`
        NULL para ambas invoices; `operation_shipments` con 0 filas para esta operación;
        `operation_pedimentos.dgo_id` apunta al DGO correcto; `warehouse.pedimento_invoices`
        con las 2 facturas y sus `valorDolares` reales; `legacy_system_data` con 0 filas
        (Decisión 5, sin sync).
      - Reuso de DGO: reintentar `POST /operations` con el mismo `dgoId` → **400**
        "DGO(s) ya vinculado(s)...". La UI también lo refleja (`ReferenceDGOTab` muestra
        el DGO como "Bloqueado", sin checkbox).
      - Visible en `ReferenceOperations.tsx` (tab Operaciones de la referencia): OP-00041,
        aduana 200, "Pendiente Docs", valor USD 8,829.68.
      - `GET /operations/:id` (modo edición) y `GET /operations/:id/pedimento`
        (dynamicData) confirmados: misma forma con shipment sintético; `pesoBruto: 0`
        (correcto — las `InvoiceItem` de prueba no tienen `grossWeight`/`netWeight`
        capturado en este ambiente, no es un placeholder falso); `totalBultos: null` en
        el pedimento (el paso de dispatch `generatePedimento` aún no corrió sobre esta
        operación, PENDING_DOCS).
      - Consola del navegador: 0 errores atribuibles al flujo en todo el recorrido. Los
        únicos 2 errores vistos (`GET /api/entities` 500) son preexistentes, de un widget
        de búsqueda global no relacionado — confirmado que aparecen igual en cualquier
        página del sitio, no solo en este flujo.
      - **Hallazgo aparte, no bloqueante:** el modo edición (`Editar Operación`) no
        hidrata los combos "Aduana de Despacho"/"Tipo de Operación"/"Agente Aduanal" con
        los valores ya guardados (`customsOfficeId`/`operationTypeId`/`mompCustomsAgentId`
        sí vienen poblados en la respuesta de `GET /operations/:id`, confirmado). Es un
        gap de `hydrateFromOperation` en el mapeo store↔combos, preexistente y
        reproducible también en el flujo legacy con shipments — no es específico de
        DGO-only ni de esta migración, fuera de alcance de este sub-plan.
- [x] `DgoService.validateHomogeneousRegimen()` (`dgo.service.ts:488-528`): eliminar el
      chequeo de `Dgo.status === SIGNED` (L500-507), conservando el chequeo de reuso
      (`operationPedimento.findMany` sobre `dgoId`, L509-518) sin cambios (Hallazgo 12).
      Confirmar que `createFromDgos()` llama a esta función (ya sin el chequeo de
      firmado) para heredar el guard de reuso, en vez de duplicarlo.
- [x] **Último paso (2026-07-30, Hallazgo 16):** con confirmación explícita del usuario
      de ampliar el alcance más allá de los 2 bloques nombrados originalmente, se borró
      **toda** la rama legacy de `create()` (no solo `L1084-1110`/`groupShipmentIds`):
      `create()` quedó como router puro (`movementInvoices` → `createWithMovementInvoices()`,
      pedimentos con `dgoId` → `createFromDgos()`, cualquier otro caso → 400). También se
      eliminaron `syncOperationToLegacy()`, `syncPedimentoToLegacy()` y
      `syncExpedienteForOperation()` — solo se llamaban desde el final de esa misma rama
      borrada, en ningún otro lugar del código (confirmado por grep en todo `src/`).
      Se borró el describe `'shipmentIds resolution per group'` de
      `operations.service.create-pedimentos.spec.ts` (probaba lógica ya eliminada). Se
      encontró y corrigió además un test roto preexistente en `dgo.service.spec.ts`
      (`'throws when any selected DGO is not SIGNED'`) que probaba el chequeo `SIGNED`
      ya eliminado en el Paso 10 — reemplazado por un test que confirma que DGOs no
      firmados sí pasan la validación (Hallazgo 12). Suite completa verde: 542/542 test
      suites, 3922 tests. Typecheck y lint limpios.
- [x] **Pedido directo del usuario (2026-07-30, fuera de los 11 pasos originales):**
      quitar el botón "Firmar DGO" del card de DGO en `ReferenceDGOTab.tsx`, coherente
      con que el status de firma ya no es gate de negocio (Hallazgo 12) — el botón
      "Acciones" (edición de datos del pedimento) se conserva. Se eliminó también
      `signMutation` (quedó sin uso) y los imports/hooks (`useUser`, `toast`,
      `queryClient` a nivel de este componente) que solo ella consumía; el prop
      `onRefresh` se conservó en la firma pública (tiene un caller real,
      `ReferenceDetailShell.tsx`) pero se renombró a `_onRefresh` al desestructurar
      (convención del proyecto para variables no usadas). Verificado en el navegador:
      el botón ya no aparece, sin errores de consola nuevos. Typecheck/lint limpios.

## Riesgos y side effects a vigilar

- Nada a nivel de base de datos impide hoy que ambas FKs de `OperationInvoice`
  (`operationShipmentId`/`operationId`) queden simultáneamente nulas o simultáneamente
  pobladas — no hay constraint que fuerce "exactamente una". Si el código nuevo tiene un
  bug, podría crear filas ambiguas; vigilar en code review de la implementación.
- Dejar el sync a legacy deshabilitado permanentemente para DGO-only (Decisión 5) es un
  hueco funcional aceptado a propósito — cualquier proceso legacy que dependa de ver
  estas operaciones no las verá hasta que se resuelva el pendiente futuro.
- `totalBultos` calculado sumando guías/shipments por `Reference` (Decisión 2 revisada)
  puede quedar mal repartido si esa `Reference` termina teniendo más de un DGO
  simultáneo (Hallazgo 18) — caso borde explícitamente fuera de alcance, pero vale la
  pena avisar al equipo de despacho mientras no se resuelva (Hallazgo 19).
- Los ~8 puntos de `operation-dispatch.service.ts` fueron localizados por grep en esta
  sesión de `/plan`; es posible que existan más consumidores de
  `pedimento.shipments`/`operation.shipments` no cubiertos por ese grep — auditar de
  nuevo al iniciar la implementación, no asumir que la lista de hoy es exhaustiva.
- Este sub-plan comparte rama con el plan padre — cualquier regresión debe distinguirse
  de las 17 tareas ya implementadas de ese plan.

## Criterios de verificación

- Gate estático verde (lint/typecheck/`nest build`) en `carmi-odin-api-v2`; gate estático
  verde (`tsc --noEmit`/eslint) en `carmi-digital`.
- Playwright: crear una operación desde el tab DGO seleccionando 1-2 DGOs (sin importar
  su status de firma) sin ningún shipment → operación creada exitosamente (sin el toast
  de bloqueo), con pedimento(s) reales y taxes calculados, visible en
  `ReferenceOperations.tsx`.
- Confirmar que un DGO ya vinculado a un `OperationPedimento` existente sigue siendo
  rechazado (400) al intentar reusarlo en una segunda operación — el guard de reuso de
  `validateHomogeneousRegimen()` debe seguir activo tras quitarle el chequeo de firmado.
- Al final de la implementación (tras el último paso de borrado, Hallazgo 16): gate
  estático y suite de tests verdes en `carmi-odin-api-v2` sin la rama de `shipments` en
  `create()`, confirmando que ningún test roto quedó dependiendo de código borrado.
- Confirmar por query directa (o endpoint) que la `Operation` creada tiene
  `OperationInvoice` con `operationId` poblado y `operationShipmentId` nulo, sin ningún
  `OperationShipment` real asociado.
- Confirmar que `syncOperationToLegacy`/`syncPedimentoToLegacy`/
  `syncExpedienteForOperation` NO se disparan para esta operación (guard activo).
- Confirmar `pesoBruto` calculado desde `InvoiceItem.grossWeight` (no 0 si las invoices
  tienen peso capturado) y `totalBultos` calculado real desde `Guia.piezas`/`bultos`
  (aéreo) o `Shipment.pieces` (terrestre) por `Reference` (Decisión 2 revisada), en
  cualquier vista que los muestre — para el caso normal de 1 DGO por Reference.
- Sin errores de consola en el flujo de Playwright.
