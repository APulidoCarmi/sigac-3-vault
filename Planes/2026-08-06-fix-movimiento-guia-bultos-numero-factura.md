# Plan: Corregir movimiento automático de Guía House — bultos, número de movimiento y vínculo de factura

## Contexto

Origen: pedido directo del usuario (sesión 2026-08-06), tras probar en
`http://localhost:3000/guias` el formulario `GuiaFormModal.tsx` — creó una
guía con `bultos = 2` y notó que el movimiento generado por detrás no
refleja esos 2 bultos. El usuario pidió, en el mismo turno, ampliar el
alcance a otros dos síntomas del mismo flujo: número de movimiento y
vínculo factura↔movimiento.

Este plan es una **corrección puntual sobre una feature ya implementada**:
[[2026-08-05-movimiento-entrada-automatico-guia-house]] (movimiento
automático por Guía House, cerrado 2026-08-05) y
[[2026-08-01-facturas-en-guias]] (facturas vinculadas a Guía). No es una
feature nueva.

Área de código (auditada con código real, no solo el texto de los planes
anteriores — ver [[Auditar codigo antes de confiar en el texto del plan]]):

- **Backend** (`carmi-odin-api-v2`):
  - `src/guias/services/guias.service.ts` — `createShipmentSkeletonForGuia()`
    (L753-786) crea el `Shipment` esqueleto; `buildShipmentFieldsFromGuia()`
    (L803-817) mapea campos de la Guía al Shipment.
  - `src/shipments/services/shipments.service.ts` — `generateShipmentCode()`
    (L438-478): genera el código de 8 dígitos con ceros a la izquierda
    (`00000001`, `00000002`, ...) buscando el máximo existente + 1,
    verificando unicidad. Ya se usa en `create()` (L147) y en otro punto
    (L2188) del flujo normal de creación de Shipment.
  - `prisma/schema.prisma`: `Shipment.shipmentCode String? @unique @db.VarChar(8)`
    (el campo real del "número de movimiento", no `shipmentNumber` que
    también existe pero está sin uso en ningún lado). `ShipmentPackageUnit`
    (esquema `warehouse`, L6594) con FK a `WarehousePackageType` (L6277) —
    es el modelo real de "bultos" de un movimiento (confirmado por uso real
    en frontend, ver Riesgos/decisión 1). `Invoice.guiaId` (L1912) y
    `InvoiceShipmentLink` (tabla M:N Invoice↔Shipment, L2038) ya existen.
  - `src/invoices/services/invoices.service.ts` — `create()` (L76-105:
    resuelve `guiaShipmentId` desde `guia.shipment.id`; L277-288: crea el
    `InvoiceShipmentLink` con ese `guiaShipmentId`) — **este código ya
    existe**, viene del mismo commit `607650ea` de la sesión del movimiento
    automático (el alcance de esa sesión creció más de lo documentado en el
    plan original).
- **Frontend** (`carmi-digital`):
  - `app/(customerPortal)/guias/components/GuiaFormModal.tsx` — envía
    `bultos` tal cual al DTO, sin problema en este lado.
  - `app/(customerPortal)/references/components/tabs/shipments/ReferenceShipmentsAereo.tsx`
    (L602-663): `getGuiaInvoices()`/`renderGuiaInvoicesBadge()` ya muestran
    un badge "N facturas" por movimiento de guía, leyendo `shipment.guia.invoices`
    (vía `Invoice.guiaId`, no vía `InvoiceShipmentLink` — ver Decisiones).
  - `components/movements/movement-form/tab-bultos.tsx` +
    `app/(customerPortal)/movements/[id]/components/detail-sections.tsx`
    (`DetailBultos`, L84-101): consumen `shipment.packageUnits`
    (`ShipmentPackageUnit`) — confirmado como el modelo real y activo para
    "bultos" de un movimiento (no `ShipmentPackage`, que no tiene ningún
    consumidor en frontend, ni `Package`, que es del dominio de
    verificación/recepción física de almacén, no del movimiento).

## Decisiones tomadas

1. **Bultos: una fila de `ShipmentPackageUnit` con `quantity = guia.bultos`**,
   no N filas de `quantity: 1`. **Corrección tras auditar código real (ver
   [[Auditar codigo antes de confiar en el texto del plan]]):** el catálogo
   que ve el usuario en el selector real de bultos
   (`components/movements/movement-form/tab-bultos.tsx`, usado en
   `http://localhost:3000/movements/create`) **no es `WarehousePackageType`**
   (el de `package-types.seed.ts`), es el modelo `Bulto` (tabla
   `warehouse.bulto`, endpoint `GET /bultos`). Ese catálogo **ya tiene** un
   código genérico para este caso: `BULTOS` (`descriptionEs: "Bultos"`,
   `id: 6678daf2-428e-456b-a9d9-56c77a1e59b6`, `isActive: true`). Se usa ese
   código existente, sin agregar nada nuevo a ningún seed:
   `tx.shipmentPackageUnit.create/upsert({ shipmentId, packageTypeCode: 'BULTOS', quantity: guia.bultos })`,
   **sin** poblar `packageTypeId` (FK a `WarehousePackageType`, un catálogo
   distinto) — mismo patrón que ya usan `createSubdivisionShipment()` y
   `declarePackageUnits()` en `shipments.service.ts`, que escriben
   `packageTypeCode` como string libre sin `packageTypeId`.
   - Porqué: decisión explícita del usuario tras ver la respuesta real de
     `GET /bultos?activeOnly=true` — usar el código ya existente evita crear
     un segundo concepto de "bulto genérico" que confundiría con `BULTOS`.
     `quantity` ya es un campo `Int` pensado para agregar unidades del mismo
     tipo; una sola fila es más simple de resincronizar en `update()` que N
     filas.
   - **Ya no aplica** el paso de agregar código a `package-types.seed.ts`
     (ver Pasos) — ese catálogo no es el que consume el frontend de bultos.
2. **Dejar de mezclar `piezas` y `bultos` en `Shipment.pieces`**:
   `buildShipmentFieldsFromGuia()` hoy hace `pieces: guia.piezas ?? guia.bultos ?? null`
   — son conceptos distintos (piezas = unidades de mercancía, bultos =
   unidades de empaque físico) y el `??` hace que `bultos` casi nunca llegue
   porque `piezas` casi siempre tiene valor. Se corrige a
   `pieces: guia.piezas ?? null` (sin fallback a `bultos`); `bultos` se
   representa exclusivamente vía `ShipmentPackageUnit` (decisión 1).
   - Porqué: es la causa raíz reportada por el usuario — mezclar los dos
     campos en uno solo pierde información sin importar el orden de
     precedencia que se use.
3. **Resincronización en `update()`, mismo criterio que el resto de campos
   mapeados desde la Guía** (pieces, grossWeight, eta, arrivalDate ya se
   resincronizan ahí): al editar `bultos` de una Guía, se busca la fila
   `ShipmentPackageUnit` existente del Shipment con `packageTypeCode = 'BULTOS'`
   (`findFirst({ shipmentId, packageTypeCode: 'BULTOS' })`):
   - Si `guia.bultos` es un entero positivo: `upsert` (actualiza `quantity`
     si la fila existe, la crea si no).
   - Si `guia.bultos` queda en `null`/`0`: se **elimina la fila con hard
     delete** (`delete`, no soft-delete). **Corrección tras auditar código
     real:** el plan original sugería respetar `deletedAt` (soft-delete) si
     el resto del código de `ShipmentPackageUnit` lo usa consistentemente —
     verificado que no es el caso: el único flujo real de sincronización
     masiva (`shipments.service.ts`, sync de `packageUnits` en `update()`)
     hace `deleteMany` con comentario explícito "hard delete" en el código, y
     `createSubdivisionShipment()`/`declarePackageUnits()` tampoco tocan
     `deletedAt` al crear/actualizar. `deletedAt` solo se respeta en
     **lecturas** (`where: { deletedAt: null }`), nunca se escribe. El
     usuario confirmó seguir esa convención real del repo (hard-delete) en
     vez de introducir soft-delete nuevo aquí.
   - Porqué: mismo patrón "lectura en vivo desde Guía, sin desincronía" ya
     aplicado a los demás campos del Shipment esqueleto en el plan de
     2026-08-05.
4. **Número de movimiento = `Shipment.shipmentCode`**, confirmado por el
   usuario: el movimiento generado desde guía debe generar `shipmentCode`
   igual que el flujo normal de Shipments (no `shipmentNumber`).
   **Corrección tras auditar código real:** `shipmentNumber` **no es** campo
   muerto como decía la versión anterior de este plan — se genera
   (`generateShipmentNumber()`) y se consume activamente en
   `receipts.service.ts` y `legacy-receipts.service.ts` (fallback de BOL).
   No cambia la decisión (el "número de movimiento" que ve el usuario en la
   guía sigue siendo `shipmentCode`, no `shipmentNumber`), pero el fix no
   debe asumir que `shipmentNumber` está libre para tocar o ignorar sin
   revisar esos consumidores — este plan no lo toca en absoluto. Se reutiliza
   el algoritmo ya existente y ya en producción de
   `ShipmentsService.generateShipmentCode()` (buscar máximo
   `shipmentCode` existente, +1, `padStart(8,'0')`, verificar colisión), pero
   **ejecutado dentro de la misma transacción (`tx`) de `GuiasService`**, no
   contra `this.prisma` como hace hoy `ShipmentsService` — porque
   `createBulk()` crea varias guías (y sus Shipments) dentro de una sola
   transacción interactiva, y una consulta fuera de esa `tx` no vería los
   `shipmentCode` ya insertados-pero-no-committeados de guías anteriores del
   mismo lote, arriesgando duplicados dentro de un mismo manifiesto.
   - Se agrega un método privado equivalente en `GuiasService`
     (`generateShipmentCode(tx)`), en vez de intentar reusar el método
     privado de `ShipmentsService` (que no acepta un `tx` y no está
     exportado) — evita acoplar `GuiasModule` a `ShipmentsModule` solo para
     esto y mantiene la generación dentro del mismo límite transaccional que
     ya usa el resto de `createShipmentSkeletonForGuia`.
   - Porqué (alcance): el usuario confirmó que la generación **ya funciona**
     para movimientos creados por el flujo normal de Shipments; el gap es
     específicamente que `createShipmentSkeletonForGuia()` nunca llama a
     ningún generador — construye el `data` de creación sin `shipmentCode`.
     Por eso el fix es puntual a ese método, sin tocar `ShipmentsService` ni
     generalizar a otros flujos.
5. **Vínculo factura↔movimiento: solo verificación + exposición en UI**, no
   nuevo código de backend salvo que la verificación revele un bug real. El
   código de `InvoicesService.create()` que crea el `InvoiceShipmentLink`
   desde `guia.shipment.id` ya existe (commit `607650ea`) y el front
   (`ReferenceShipmentsAereo.tsx`) ya muestra un badge de facturas por
   movimiento — aunque lo hace vía `Guia.invoices` (`Invoice.guiaId`), no vía
   `Shipment.invoiceShipmentLinks`. Ambos caminos deberían apuntar a las
   mismas facturas en la práctica (toda factura de guía con Shipment
   resuelto crea ambos: el escalar `guiaId` y el link M:N), así que no se
   duplica UI — se **verifica** que el flujo real (crear guía → crear
   factura desde `GuiaInvoicesModal` → confirmar que aparece en el badge del
   movimiento aéreo, y que `InvoiceShipmentLink` existe en BD) funciona de
   punta a punta, y solo se corrige código si la verificación encuentra una
   falla concreta.
   - Porqué: el usuario eligió "verificar que funciona de punta a punta" +
     "mostrarlo en la UI" (no "backfill"), y auditar el código real mostró
     que ambas piezas ya existen — escribir código nuevo sin haber
     verificado primero violaría la regla de no hacer trabajo redundante.

## Hallazgo derivado (fuera de alcance, plan aparte)

Verificando este plan (2026-08-07), al editar una factura ya vinculada a un
movimiento de guía (cambiar la cantidad de una partida), la "Disponibilidad"
en `ReferenceShipmentsAereo.tsx` no reflejaba el cambio. Causa raíz
identificada: `InvoicesService.update()` (`invoices.service.ts:798-838`)
borra TODAS las `InvoiceItem` de la factura y crea partidas nuevas con IDs
distintos cuando el DTO trae `items`; `ShipmentItem.invoiceItemId` tiene
`onDelete: Cascade` (`schema.prisma:7576`), así que ese borrado elimina en
cascada el `ShipmentItem` (el ledger de disponibilidad) sin recrearlo para
las partidas nuevas. Es un bug de integridad de datos preexistente, no
introducido por este plan, y no exclusivo de guías aéreas — afecta cualquier
factura con mercancía ya vinculada a un movimiento/operación. Se abre un
plan dedicado para diseñar la reconciliación correcta (`ShipmentItem`,
`ShipmentItemTransaction`, `OsdLog`) en vez de parchearlo aquí.

## Fuera de alcance

- Backfill de facturas de guía creadas **antes** del commit `607650ea` sin
  `InvoiceShipmentLink` (el usuario no marcó esa opción).
- Backfill de `Shipment.shipmentCode` para movimientos de guía ya existentes
  sin número (solo aplica hacia adelante, mismo criterio que el resto de
  esta línea de trabajo — [[2026-08-05-movimiento-entrada-automatico-guia-house]]
  ya estableció "solo hacia adelante" para casos análogos).
- Backfill de bultos (`ShipmentPackageUnit`) para movimientos de guía ya
  existentes — solo aplica a guías creadas/editadas desde ahora.
- Extender la generación de `shipmentCode` a otros flujos de creación de
  Shipment que no sean guía (ya funcionan, no es parte del bug reportado).
- Cambios a `ShipmentPackage` o `Package` (modelos no usados por la vista de
  movimiento, fuera del bug reportado).
- Cualquier ajuste al badge de facturas en `ReferenceShipmentsAereo.tsx` más
  allá de confirmar que ya funciona — si la verificación no encuentra fallas,
  no se toca ese código.

## Pasos

- [ ] ~~**Backend — schema**: agregar código nuevo a `package-types.seed.ts`~~
      — **ya no aplica**: el código `BULTOS` ya existe en el catálogo real que
      consume el frontend (`Bulto`/`GET /bultos`, confirmado por el usuario
      con la respuesta real del endpoint). No se toca ningún seed.
- [x] **Backend — `createShipmentSkeletonForGuia()`**
      (`guias.service.ts:753-789`): tras crear el `Shipment`, si
      `guia.bultos` es un entero positivo, crear una fila
      `tx.shipmentPackageUnit.create({ shipmentId, packageTypeCode: 'BULTOS', quantity: guia.bultos })`
      — sin `packageTypeId` (ese campo es FK a `WarehousePackageType`, un
      catálogo distinto y no relacionado; mismo patrón que ya usan
      `createSubdivisionShipment()`/`declarePackageUnits()` en
      `shipments.service.ts`, que tampoco lo pueblan).
- [x] **Backend — `buildShipmentFieldsFromGuia()`** (`guias.service.ts:803-817`):
      cambiar `pieces: guia.piezas ?? guia.bultos ?? null` a
      `pieces: guia.piezas ?? null`.
- [x] **Backend — `generateShipmentCode(tx)`**: nuevo método privado en
      `GuiasService` (mismo algoritmo que
      `ShipmentsService.generateShipmentCode()` L438-478, pero recibiendo y
      usando el `tx` de la transacción activa en vez de `this.prisma`).
      Llamarlo en `createShipmentSkeletonForGuia()` y setear
      `shipmentCode` en el `data` de `tx.shipment.create()`.
- [x] **Backend — resincronización en `update()`**: en el bloque que ya
      resincroniza `pieces`/`grossWeight`/`eta`/`arrivalDate` al editar una
      Guía con Shipment asociado, agregar el upsert/hard-delete de
      `ShipmentPackageUnit` (código `BULTOS`) descrito en la Decisión 3.
- [x] **Backend — specs**: actualizar/agregar casos en
      `guias.service.spec.ts` cubriendo: creación de Shipment desde guía con
      `bultos` → existe `ShipmentPackageUnit` con `quantity` correcto y
      `Shipment.pieces` solo refleja `piezas`; `shipmentCode` generado y
      único entre las guías de un mismo `createBulk()`; edición de `bultos`
      (aumenta, disminuye, se pone en `null`) resincroniza correctamente la
      fila de bultos.
- [x] **Bug encontrado durante verificación manual (2026-08-07)**: el usuario
      reportó que al editar una Guía desde `GuiaFormModal.tsx`, la edición no
      se reflejaba en el movimiento. Auditado con un subagente Explore:
      - La tabla del tab Guías de una Referencia
        (`ReferenceShipmentsAereo.tsx`) NO tiene este problema — lee
        `shipment.guia.X` en vivo vía el join de
        `ShipmentsService.findByReference()`.
      - El bug real está en `/movements/[id]` (`detail-overview.tsx`, sección
        "Notas"): ese endpoint (`GET /shipments/:id` →
        `ShipmentsService.findOne()`) **no hace join a `Guia`**, así que
        depende 100% de los campos snapshotteados en `Shipment`. `pieces` /
        `grossWeight` / `eta` / `arrivalDate` ya se resincronizaban
        (`buildShipmentFieldsFromGuia`), pero `Shipment.notes` (texto con
        `guiaHouse` embebido) y `Shipment.metadata`
        (`guiaMaster`/`guiaHouse`/`guiaId`) se llenaban solo al crear el
        Shipment y nunca se tocaban en `update()` — quedaban congelados con
        el valor de la guía al momento de crear el movimiento.
      - **Fix**: se movieron `notes` y `metadata` a `buildShipmentFieldsFromGuia()`
        (antes solo `pieces`/`grossWeight`/`grossWeightUnit`/`eta`/`arrivalDate`),
        eliminando la duplicación que existía en
        `createShipmentSkeletonForGuia()`. Como `update()` ya llama a
        `buildShipmentFieldsFromGuia()` en su `tx.shipment.updateMany()`,
        `notes`/`metadata` quedan resincronizados automáticamente sin lógica
        nueva — mismo patrón que los demás campos mapeados.
      - No se tocó `ShipmentsService.findOne()` (agregarle join a `Guia`
        sería un cambio más amplio, fuera de alcance mientras el fix de
        `notes`/`metadata` resuelve el síntoma reportado).
      - Specs actualizados en `guias.service.spec.ts`: el test de
        resincronización ahora exige `notes`/`metadata` en el
        `updateMany()`, más un caso nuevo que edita `guiaHouse`/`guiaMaster`
        y verifica que ambos campos se regeneran. 64/64 tests en verde.
- [ ] **Verificación end-to-end factura↔movimiento** (sin cambios de código
      salvo que se encuentre un bug real): crear una guía nueva, subir una
      factura desde `GuiaInvoicesModal`, confirmar en BD que existe el
      `InvoiceShipmentLink` con el `shipmentId` del movimiento de esa guía, y
      confirmar visualmente que el badge de facturas en
      `ReferenceShipmentsAereo.tsx` (si la guía ya tiene Referencia) refleja
      esa factura. Si algo falla, documentar la causa raíz encontrada y
      corregirla en este mismo plan (no se abre plan aparte para un bug de
      esta misma cadena).
- [ ] **Verificación end-to-end completa**: crear una guía House individual
      con `bultos = 2` desde `http://localhost:3000/guias` (el caso reportado
      por el usuario) → el movimiento generado tiene `shipmentCode` con
      formato `00000183` (8 dígitos) y su tab "Bultos"
      (`tab-bultos.tsx`/`DetailBultos`) muestra 1 tipo de bulto con
      `quantity = 2`. Repetir con alta masiva (`createBulk`, manifiesto de
      varias guías House) y confirmar que cada Shipment recibe un
      `shipmentCode` distinto (sin duplicados dentro del mismo lote).

## Riesgos y side effects a vigilar

- `generateShipmentCode(tx)` duplicado entre `ShipmentsService` y
  `GuiasService`: mismo algoritmo en dos archivos. Es una divergencia
  consciente (ver Decisión 4) para no acoplar módulos ni cambiar la firma de
  un método ya en producción en `ShipmentsService` — si en el futuro se
  necesita esta lógica en un tercer lugar, vale la pena extraerla a un
  helper compartido (ej. `src/shipments/utils/`), pero no en este plan.
- `createBulk()` ya es una transacción interactiva más pesada desde el plan
  de 2026-08-05 (guía + shipment por fila); agregar la consulta de
  `shipmentCode` (que además reintenta recursivamente si hay colisión) suma
  una consulta más por fila — vigilar el timeout de `{ timeout: 20_000, maxWait: 10_000 }`
  ya configurado con manifiestos grandes (~20 guías House).
- ~~El código `BULTO` nuevo en `WarehousePackageType`...~~ — ya no aplica:
  se usa el código `BULTOS` ya existente y `ACTIVE` en el catálogo real
  (`Bulto`/`GET /bultos`), no se crea nada en `WarehousePackageType`.
- Eliminar la fila `ShipmentPackageUnit` cuando `bultos` se pone en `null`
  (Decisión 3) se hace con **hard delete** (`delete` físico), confirmado
  como la convención real del repo tras auditar código (`deleteMany` con
  comentario explícito "hard delete" en la sync de `packageUnits` de
  `shipments.service.ts`; `deletedAt` solo se respeta en lecturas, nunca se
  escribe en ningún flujo existente de `ShipmentPackageUnit`).
- Código huérfano preexistente (no introducido por este plan, pero cercano
  a la zona que se toca): `createSubdivisionShipment()` y
  `declarePackageUnits()` en `shipments.service.ts` ya escriben
  `packageTypeCode: 'BULTO'` (singular, sin la `S` final) — un código que no
  existe en ningún catálogo. Este plan usa `'BULTOS'` (el código real y
  activo en `Bulto`), lo que introduce una divergencia de nomenclatura ya
  existente en el repo entre `'BULTO'` (huérfano) y `'BULTOS'` (real) — fuera
  de alcance corregir esos dos métodos, pero vale la nota para no confundir
  ambos strings al tocar `guias.service.ts`.
- La verificación de factura↔movimiento puede destapar un bug no
  documentado (mismo patrón que la sesión de 2026-08-05, donde cada
  verificación end-to-end encontró un problema real nunca antes ejercitado)
  — si aparece, corregirlo aquí mismo y documentarlo en el cierre del plan,
  no descartarlo como "fuera de alcance".

## Criterios de verificación

- Crear guía individual con `bultos = 2` (sin `piezas`) → el `Shipment`
  generado tiene `pieces: null` y una fila `ShipmentPackageUnit` con
  `packageTypeCode: 'BULTO'`, `quantity: 2`.
- Crear guía individual con `bultos = 2` y `piezas = 50` → `Shipment.pieces = 50`
  (nunca 2), y la fila de bultos sigue teniendo `quantity: 2` de forma
  independiente.
- Editar esa guía cambiando `bultos` de 2 a 5 → la misma fila
  `ShipmentPackageUnit` pasa a `quantity: 5` (no se crea una segunda fila).
  Editar `bultos` a vacío/null → la fila se elimina (soft-delete si aplica).
- Alta masiva (`createBulk`) de 5 guías House en un mismo manifiesto → 5
  Shipments con `shipmentCode` consecutivos y únicos (ej. `00000180` a
  `00000184`), sin colisiones.
- El tab "Bultos" del formulario de movimiento (`tab-bultos.tsx`) y la vista
  de solo lectura (`DetailBultos`) muestran el bulto generado desde la guía
  igual que cualquier otro bulto capturado manualmente.
- Crear una factura desde `GuiaInvoicesModal` sobre una guía con movimiento
  ya generado → existe `InvoiceShipmentLink(invoiceId, shipmentId)` en BD y
  (si la guía tiene Referencia) el badge de facturas en
  `ReferenceShipmentsAereo.tsx` refleja la factura nueva.
- Suite de backend (`guias.service.spec.ts` y los specs que toquen
  `invoices.service.ts` si la verificación de factura requiere cambios) en
  verde; `/verify` (lint + typecheck + tests) antes de cerrar.
