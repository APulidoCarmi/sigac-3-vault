# Plan: Backend — `guiaId` en Invoice + migración automática a DGO (sp01 de facturas en Guías)

> Sub-plan hijo de [[2026-08-01-facturas-en-guias]]. Repo: `carmi-odin-api-v2`.

## Contexto

Ver paraguas [[2026-08-01-facturas-en-guias]]. Hoy el modelo `Invoice`
exige `referenceId` y solo admite `dgoId` opcional; no existe ningún campo
que lo asocie a una `Guia`. La guía House puede existir antes de que exista
Referencia (tablero pre-referencia de Querétaro), así que la factura subida
desde el módulo de Guías debe poder crearse sin `referenceId`.

## Decisiones tomadas

1. `guiaId` es nullable, FK a `Guia` (con su lado inverso `invoices
   Invoice[]` en el modelo `Guia`, obligatorio en Prisma para una relación
   bidireccional). Una factura puede tener `guiaId` sin `referenceId`, o
   `referenceId`/`dgoId` sin `guiaId` (flujo actual), o ambos tras la
   migración automática (decisión 3).
2. Regla de validación en el service: al crear una factura se exige **al
   menos una ancla** presente, en OR puro y sin exclusividad mutua, sobre
   el conjunto `referenceId OR shipmentId OR guiaId` (se descubrió en la
   investigación previa a implementar que hoy ya existe una tercera ancla,
   `shipmentId`, con la misma regla "al menos uno" —
   `invoices.service.ts:242-246`; el plan original solo contemplaba
   `referenceId`/`guiaId` y se corrige aquí para no romper el caso
   `shipmentId`). No se agrega exclusividad entre anclas — no hay ningún
   escenario de negocio que la requiera y una factura migrada puede
   terminar con varios campos poblados a la vez.
   - Implementación: centralizar la regla en un único helper/constante
     (p. ej. `ANCHOR_FIELDS = ['referenceId', 'shipmentId', 'guiaId']` +
     una función `assertHasAnchor(dto)`) usado solo por `create()` — NO
     se toca `createFromPackingList()` (decisión 4 más abajo). Evita que
     la validación quede duplicada/desincronizada entre los dos métodos
     de creación de `Invoice`.
   - No se agrega un `CHECK` constraint de base de datos para esta regla:
     `shipmentId` no es una columna de `Invoice` sino una fila en la tabla
     puente `InvoiceShipmentLink`, invisible para un `CHECK` de Postgres
     sin un trigger — complejidad desproporcionada para este caso. La
     regla vive solo en código de aplicación, con cobertura de test
     explícita por combinación (ninguna ancla → 400; cada una sola →
     201; combinadas → 201).
3. La migración automática ocurre en el flujo ya existente de creación de
   Referencia desde guías seleccionadas (`linkGuiasToReference` en
   `guias.service.ts:393-498`, el que hoy usa `linkGuiasToReference` en el
   front). Al crear la Referencia/DGO, el backend debe: por cada grupo de
   guías (agrupadas por `pedimentoCode`, la misma clave que el código ya
   usa hoy para decidir cuántos DGOs crear — ver nota de decisión 6 más
   abajo), buscar las facturas de esas guías con `referenceId IS NULL` y
   setearles **tanto `referenceId` como `dgoId`** del DGO de ese grupo, de
   forma atómica en la misma transacción. Es necesario setear ambos campos
   porque el flujo de despacho (`operations.service.ts:987`) valida
   pertenencia de una factura a una referencia vía `invoice.dgo.referenceId`,
   no vía `invoice.referenceId` directamente — una factura migrada que solo
   tuviera `referenceId` seteado seguiría sin poder usarse en operaciones.
   Esta migración es un mecanismo nuevo y distinto al barrido existente en
   `DgoService.ensureDefault` (`dgo.service.ts:54-58`), que solo cubre
   facturas con `referenceId` ya seteado pero `dgoId` nulo — no colisionan
   entre sí.
4. No se crea endpoint `getById` de Guia de propósito general. Para listar
   facturas de una guía se agrega `guiaId` como filtro adicional de `GET
   /invoices` (el endpoint paginado ya existente en
   `invoices.controller.ts:46-104`), no un endpoint anidado nuevo tipo
   `GET /guias/:id/invoices` — se prefiere este patrón porque ya es el que
   usa el front para filtrar facturas y ya trae paginación.
5. `guiaId` solo se soporta en `InvoicesService.create()` (la vía que
   usará el nuevo formulario del módulo de Guías) — no se replica en
   `createFromPackingList()` (`invoices.service.ts:1110`), que mantiene su
   regla de anclaje actual (`referenceId`/`shipmentId`) sin cambios.
6. La migración automática de facturas huérfanas hereda la agrupación por
   `pedimentoCode` que `linkGuiasToReference` ya usa hoy para decidir
   cuántos DGOs crear (no se corrige la discrepancia de nomenclatura
   preexistente en el código, que dice agrupar "por régimen" cuando en
   realidad agrupa por `pedimentoCode` — se confirma que es un comentario
   impreciso en código ya existente, no un bug que afecte este sub-plan, y
   queda fuera de alcance corregirlo aquí).
7. `referenceId` de `Invoice` **ya es nullable** tanto en `schema.prisma`
   (`String? @db.Uuid`) como en la base de datos (la migración inicial
   nunca tuvo `NOT NULL`, FK con `ON DELETE SET NULL`) — el paso de este
   plan que hablaba de "relajar `referenceId` de obligatorio a opcional"
   vía migración de Prisma **no aplica**, ya está resuelto. El único
   trabajo de este punto es agregar `guiaId` como nueva ancla válida en la
   validación de aplicación (decisión 2).
8. La FK `Invoice.guiaId → Guia` usa `onDelete: SetNull` (mismo patrón que
   `dgo`/`reference` en `Invoice` hoy). En la práctica esto es un
   respaldo defensivo: `Guia` nunca se borra a nivel de fila (ver
   decisión 9, borrado lógico).
9. Se agrega borrado lógico de `Guia` (queda dentro de alcance de este
   sub-plan, confirmado explícitamente por el usuario), replicando al
   detalle el patrón ya existente de `Reference.remove()`/`restore()`
   (`references.service.ts:5771-5831`) en vez de inventar uno nuevo:
   - Prisma: agregar a `Guia` los mismos campos de auditoría que
     `Reference` — `deletedAt DateTime? @db.Timestamptz(6)`,
     `deletedBy String? @db.Uuid`, `deletedByName String? @db.VarChar(255)`,
     `deleteReason String? @db.VarChar(100)`, `deletionObservations String?`
     — vía `npx prisma migrate dev --name add-soft-delete-to-guia`.
   - DTO: `DeleteGuiaDto` idéntico a `DeleteReferenceDto`
     (`src/references/dtos/delete-reference.dto.ts`): `reason` obligatorio
     (`@IsString() @IsNotEmpty() @MaxLength(100)`), `observations`/
     `deletedByName`/`deletedBy` opcionales.
   - Regla de negocio (la que pidió el usuario): no se puede borrar una
     Guia si ya existe una `Operation` vinculada a su Referencia. Mismo
     query que usa `Reference.remove()` (`references.service.ts:5805-5812`):
     `operationCount = await tx.operation.count({ where: { referenceId:
     guia.referenceId } })`; si `operationCount > 0` → 400. Si
     `guia.referenceId` es `null` (guía sin Referencia aún), el conteo es
     0 y el borrado siempre procede.
   - Servicio: `GuiasService.remove(id, dto)` y `GuiasService.restore(id)`
     calcados de `ReferencesService.remove()`/`restore()`
     (`references.service.ts:5771-5831`): `runAudited`, 404 si no existe o
     ya está borrada (`remove`) / si no existe o no está borrada
     (`restore`), `update` seteando/limpiando `deletedAt` + los 4 campos
     de auditoría.
   - Controller: `DELETE /guias/:id` (body `DeleteGuiaDto`) y
     `PATCH /guias/:id/restore`, mismo patrón que
     `references.controller.ts:753-768`. **Sin guard de autenticación** —
     sigue la convención vigente (temporal) de todo el clúster
     Guias/References (`guias.controller.ts:25-32`,
     `references.controller.ts:34-38`), no específica de este endpoint.
   - Listados: `GuiasService.findAll` (usado por `GET /guias`) debe
     excluir `deletedAt` no nulo por defecto, igual que el resto de
     módulos con soft delete (patrón `where: { deletedAt: null }`, p.ej.
     `references.service.ts:88`).

## Fuera de alcance

- Tareas 2, 3, 4, 5 de [[2026-07-31 - Revisión de Movimientos - Guía]]
  (movimiento de entrada automático, integración a expediente, pregunta de
  guía al subir desde DGO, documento genérico en factura DGO).
- Cambios a reportes/métricas existentes que lean `Invoice.referenceId`
  como siempre presente — solo se audita su impacto (paso 4), no se
  refactorizan en este sub-plan salvo que bloqueen la migración.

## Pasos

- [x] Localizar el modelo `Invoice` en `prisma/schema.prisma` y su
      service/controller (`src/invoices/**`) y confirmar nombre exacto de
      campos y relaciones actuales — hecho en la investigación previa:
      `referenceId` (`schema.prisma:1851`, ya nullable), `dgoId`
      (`:1909`, ya nullable), service en
      `src/invoices/services/invoices.service.ts`, controller en
      `src/invoices/controllers/invoices.controller.ts`, DTO en
      `src/invoices/dtos/create-invoice.dto.ts`.
- [x] Agregar campo `guiaId` (nullable) + relación a `Guia` en
      `schema.prisma` (Invoice → `guiaId String? @db.Uuid` + relación;
      Guia → agregar el lado inverso `invoices Invoice[]`, obligatorio
      para que Prisma acepte la relación bidireccional), y correr
      `npx prisma migrate dev --name add-guia-id-to-invoice` (prohibido
      escribir el SQL de migración a mano, regla global). **No** se toca
      `referenceId` (ya es nullable en schema y BD, decisión 7).
- [x] Actualizar `CreateInvoiceDto` para aceptar `guiaId` opcional
      (`@IsOptional() @IsUUID()`, mismo patrón que `referenceId`/`dgoId`
      en `create-invoice.dto.ts`).
- [x] Refactor de la validación de "ancla" en `InvoicesService.create()`
      (hoy inline en `invoices.service.ts:242-246`, solo
      `referenceId`/`shipmentId`): extraer a un helper centralizado que
      valide "al menos una de `referenceId`/`shipmentId`/`guiaId`" (ver
      decisión 2) y usarlo únicamente en `create()`. No modificar
      `createFromPackingList()` (decisión 5).
- [x] Agregar `guiaId` como filtro opcional de `GET /invoices` (paginado,
      `invoices.controller.ts:46-104` / `invoices.service.ts:615-639`),
      siguiendo el mismo patrón que los filtros `referenceId`/
      `shipmentId`/`supplierId` ya existentes (decisión 4). No se crea
      `GET /guias/:id/invoices` ni `getById` de Guia.
- [x] En `GuiasService.linkGuiasToReference` (`guias.service.ts:393-498`),
      dentro del loop existente por grupo de `pedimentoCode`
      (`:447-476`, donde ya se resuelve el `dgo.id` de cada grupo antes de
      la transacción final), agregar al array de la transacción
      (`:478-495`) un `invoice.updateMany` por grupo: `where: { guiaId: {
      in: <guiaIds del grupo> }, referenceId: null, deletedAt: null },
      data: { referenceId: dto.referenceId, dgoId: <dgo.id del grupo> }`
      (decisión 3). Confirmar que no colisiona con el barrido de
      `DgoService.ensureDefault` (`dgo.service.ts:54-58`, alcance
      distinto: `referenceId` ya seteado + `dgoId` nulo).
- [x] Auditar (grep, no refactor) qué otras queries/reportes asumen
      `Invoice.referenceId` siempre presente. Ya auditado en la
      investigación previa: todos los usos en `references.service.ts`
      (líneas 1288, 1365, 2982, 4884, 5068, 3708-3738, 6144) están
      acotados a una Reference específica (`where: referenceId: <id
      conocido>`), riesgo bajo — una factura con `referenceId` nulo
      simplemente no aparece hasta que la migración automática la
      engancha. No se encontraron reportes/agregaciones en `src/reports/**`
      que dependan de `referenceId` siempre no-nulo. Sin hallazgos
      adicionales que requieran ajuste — dejar esta nota como cierre del
      paso, sin necesidad de tocar código extra aquí.
- [x] Escribir/actualizar specs: el pre-commit de este repo exige que todo
      método público de los `.service.ts`/`.controller.ts` tocados
      aparezca referenciado en su `.spec.ts` — cubrir explícitamente:
      matriz de combinaciones del helper de ancla (ninguna → 400; cada
      una sola de `referenceId`/`shipmentId`/`guiaId` → 201; combinadas →
      201), el filtro `guiaId` en `GET /invoices`, y la migración
      automática al crear Referencia (extender
      `guias.service.spec.ts` — mockear `mockPrisma.invoice.updateMany` y
      verificar `referenceId`/`dgoId` correctos por grupo). **Nota
      obligatoria**: `mockPrisma` en `guias.service.spec.ts` (líneas 8-31)
      no incluye la clave `invoice` hoy — agregar
      `invoice: { updateMany: jest.fn() }` al mock o los tests existentes
      de `linkGuiasToReference` rompen con `TypeError` en cuanto se agregue
      la llamada real.
- [x] Agregar borrado lógico de `Guia` (decisión 9): campos de auditoría en
      `schema.prisma` vía `npx prisma migrate dev --name
      add-soft-delete-to-guia`, `DeleteGuiaDto`, `GuiasService.remove()`/
      `restore()` con la regla de "no borrar si hay Operation asociada a
      la Referencia de la guía", endpoints `DELETE /guias/:id` y
      `PATCH /guias/:id/restore`, y excluir `deletedAt` no nulo en
      `GuiasService.findAll`. Cubrir en `guias.service.spec.ts`/
      `guias.controller.spec.ts`: borrar guía sin Referencia → 200;
      borrar guía con Referencia sin Operation → 200; borrar guía con
      Referencia que ya tiene Operation → 400; restore → limpia los 4
      campos de auditoría; listado `GET /guias` no incluye guías
      borradas.

## Riesgos y side effects a vigilar

- El pre-commit de `carmi-odin-api-v2` corre `test:cov` completo; tocar
  `invoices.service.ts` puede exponer deuda de cobertura preexistente
  ajena a esta iniciativa (ver precedente documentado en el paraguas
  Querétaro-aéreo, donde se saltó una vez con `--no-verify` bajo
  autorización explícita).
- La migración automática debe ser transaccional junto con la creación de
  la(s) Referencia(s)/DGO(s): si falla a medio camino, no debe dejar
  facturas parcialmente migradas.
- Confirmado en la investigación previa: no existe constraint `NOT NULL`
  de `referenceId` a nivel de base de datos (migración inicial,
  `migration.sql:969`, columna ya nullable con FK `ON DELETE SET NULL`,
  `:5675`) — no hay riesgo de bloqueo aquí, ver decisión 7.
- El flujo de despacho (`operations.service.ts:987`) valida pertenencia de
  una factura vía `invoice.dgo.referenceId`, no `invoice.referenceId`
  directo — la migración automática debe setear ambos campos (decisión 3)
  o las facturas migradas seguirán sin poder usarse en operaciones aunque
  tengan `referenceId`.

## Criterios de verificación

- `npx prisma migrate dev --name add-guia-id-to-invoice` corre limpio y
  genera el SQL esperado (revisar el archivo generado, no escribirlo) —
  solo agrega la columna/FK de `guiaId`, no toca `referenceId`.
- `npm run test:cov` (o el comando equivalente del repo) pasa, incluyendo
  los specs nuevos.
- Prueba manual/integración: crear una guía sin Referencia → crear factura
  con `guiaId` y sin `referenceId` → confirmar 201 y campos correctos en
  BD. Intentar crear factura sin `referenceId`/`shipmentId`/`guiaId`
  (ninguna ancla) → confirmar rechazo (400). Crear Referencia desde esa
  guía → confirmar que la factura queda con `referenceId`/`dgoId`
  poblados automáticamente, y que puede usarse en una Operación (validando
  que `invoice.dgo.referenceId` también quedó correcto).
- Prueba manual borrado lógico de Guia (decisión 9): borrar una Guia sin
  Referencia → 200. Crear Referencia + Operación desde una Guia y luego
  intentar borrarla → 400 con mensaje de operación(es) asociada(s).
  Restaurar una Guia borrada → 200 y campos de auditoría en `null`.
  `GET /guias` no debe listar guías borradas.
