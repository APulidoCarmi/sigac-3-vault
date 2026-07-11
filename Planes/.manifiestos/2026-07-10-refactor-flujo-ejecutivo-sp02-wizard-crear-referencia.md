# Manifiesto — SP-02: Wizard de Crear Referencia (3 pasos)

Sub-plan: [[2026-07-10-refactor-flujo-ejecutivo-sp02-wizard-crear-referencia]]
Rama: `refactor/customs-operation-sp02` en ambos repos (encadenada desde
`refactor/customs-operation-sp01`, diff acumulado de SP-03+SP-01 intacto).
Estado: **parcialmente completado** — bloqueo real en la migración Prisma
(infraestructura, no de código), resto de tareas completo.

## Archivos tocados

### Front (`carmi-digital`)
- `app/(customerPortal)/references/createReference/page.tsx` — orquestador del
  wizard: reordenado a 3 pasos (básicos → documentos → auditoría), retirado el
  paso "Documentos Comerciales", y eliminadas ~640 líneas de estado/handlers
  que quedaban muertos como consecuencia directa de quitar ese paso
  (auto-búsqueda de proveedor, mapeo de `invoices`/`packingLists`, modal de
  factura duplicada).
- `app/(customerPortal)/references/createReference/create-reference-context.tsx`
  — lista de pasos del wizard actualizada a 3.
- `app/(customerPortal)/references/createReference/components/stepBasicDataReference/index.tsx`
  — campo "orden de compra"; ocultamiento de selectores de cliente/tráfico/
  instrucciones cuando el creador es el cliente-portal (ver decisión abierta
  más abajo).
- `lib/schemas/new-reference-basicData.ts` — validación del nuevo campo OC.
- `components/Ref/ReferenceMain.tsx` — se corrigió un import muerto hacia
  `reference-stepper` (componente no usado en ninguna ruta activa).
- Eliminado: `app/(customerPortal)/references/reference-stepper/index.tsx`
  (wizard alterno con placeholders, código muerto confirmado por grep: solo
  lo importaba `ReferenceMain.tsx`, que a su vez no está montado en ninguna
  ruta activa).

### Back (`carmi-odin-api-v2`)
- `prisma/schema.prisma` — campo `purchaseOrder String? @map("client_purchase_order")`
  en `Reference`, con comentario documentando el bloqueo de migración (ver
  abajo). Mapeado a una columna que YA existe físicamente en la DB de este
  entorno.
- `src/references/dtos/create-reference.dto.ts` — campo OC expuesto en el DTO
  de creación con validadores class-validator siguiendo el patrón existente.
- `src/references/services/references.service.ts` — persiste/retorna el
  campo OC en creación y en `GET /references/:id`.

## Por qué de cada cambio
1. **Orden 3 pasos**: decisión M9 ya tomada en el sub-plan (datos básicos
   primero porque la auditoría de extracción los necesita) — aplicada tal
   cual.
2. **Retirar "Documentos Comerciales" del wizard sin borrar el componente**:
   el propio sub-plan dice que esa sección se REUBICA al detalle en SP-05,
   no se elimina como feature. Borrarla habría anticipado trabajo de otro
   sub-plan fuera de este alcance.
3. **Campo OC**: pedido explícito del sub-plan (M9: el cliente busca por su
   OC).
4. **Dos sabores cliente/ejecutivo**: se reutilizó `user.IsInternalUser`, ya
   presente en el contexto de sesión/auth pero sin usar en ninguna vista —
   es la distinción de rol reutilizable más cercana encontrada en el repo.
5. **Limpieza `reference-stepper`**: código muerto confirmado por grep antes
   de borrar, según regla del proyecto.

## Desviaciones y decisiones abiertas (requieren confirmación humana)
- **Distinción cliente/ejecutivo (tarea 4)**: se infirió a partir de
  `user.IsInternalUser`, que no estaba cableado a ninguna vista previamente.
  Es una inferencia razonable pero NO una decisión de producto confirmada —
  validar con negocio antes de dar la tarea por cerrada.
- **`stepChecklistGlosa` y `stepInvoicesReference`**: confirmado que no están
  cableados al wizard actual; no se decidió integrarlos ni eliminarlos por
  falta de certeza sobre su necesidad en el flujo de 3 pasos. Quedan
  intactos, sin usar. Pendiente de decisión.
- **`stepCommercialDocsReference`**: queda en el repo sin cablear al wizard
  (no es código muerto: material para SP-05). No tocar su implementación
  interna, solo su desconexión del wizard ya se hizo aquí.

## Bloqueo: migración Prisma (tarea 3, campo OC)
`npx prisma migrate dev --name add-purchase-order-to-reference` falla con
P3015: existen dos carpetas de migración vacías y NO versionadas en git
(`prisma/migrations/20260710180000_add_reference_client_purchase_order` y
`prisma/migrations/20260710220000_sp04_expediente_document_metadata_checklist_history`)
que SÍ figuran como aplicadas en `_prisma_migrations` de la DB de este
entorno — la columna `client_purchase_order` ya existe físicamente en
`references` (confirmado vía `information_schema.columns`), pero falta el
`migration.sql` correspondiente en el historial versionado.

Conforme a la regla dura del proyecto (nunca escribir SQL de migración a
mano), NO se generó el archivo faltante ni se corrió `migrate resolve`. En
su lugar, el campo Prisma se mapeó (`@map`) a la columna ya existente para
que el ORM funcione en runtime sin duplicar la columna. **Se requiere
decisión humana** sobre cómo reconciliar el historial de migraciones de
este entorno (parece un remanente de otra sesión/sub-plan en paralelo,
posiblemente SP-04, que dejó migraciones a medio generar). Una vez resuelto
el historial, sí correr la CLI de Prisma para dejar la migración de OC
correctamente versionada.

## Verificación
- **Back**: `npx tsc --noEmit`, `eslint`, `jest src/references` (21/21 tests)
  — sin errores nuevos.
- **Front**: `npx tsc --noEmit` y `eslint` sobre los archivos tocados — 0
  errores (solo warnings preexistentes de `any`/variables no usadas).
- **Playwright**: NO ejecutado — no se confirmó un dev server corriendo y el
  backend tiene el bloqueo de migración arriba descrito. Pendiente de correr
  una vez resuelto el historial de migraciones y con el entorno levantado.

## Estado del sub-plan
Actualizar el sub-plan: tareas 1, 2, 5 completas; tarea 3 completa en
código pero con migración bloqueada (infraestructura); tarea 4 completa con
decisión de rol pendiente de validar.
